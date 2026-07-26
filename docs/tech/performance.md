# Horde Scale & Performance

!!! success "Decided 2026-07-25"
    **~800 active agents**: 200 undead per player (≈400 friendly), plus an enemy force of 200-400,
    plus a corpse field of **3-5k records**. Units are **positional agents, not rigid bodies**.
    Engine is **Godot 4**. See the [Decision Log](../project/decisions.md).

    The number is a target to push against and measure, not a guarantee. Everything below is how we
    intend to hit it.

## M0 results: measured 2026-07-26 <span class="pill done">SPIKE COMPLETE</span>

**GDScript holds 800 agents. No GDExtension port needed.** The project's biggest technical risk is
retired.

Measured on an **i9-13900K / RTX 4090**. Headless, so these are CPU-only numbers; the 8 ms budget
was always about CPU. Two masses of agents walking into each other
through a walled gap, sampled **at peak chokepoint density**, 200 frames per case.

| agents | threading | hash | separation | integrate | **sim avg** | **sim p95** | multimesh | verdict |
|---:|---|---:|---:|---:|---:|---:|---:|---|
| 400 | single | 0.83 | 1.21 | 0.21 | **2.25** | **3.23** | 0.15 | ✅ PASS |
| 400 | ×8 | 0.60 | 0.75 | 0.15 | **1.50** | **1.74** | 0.11 | ✅ PASS |
| **800** | **single** | 0.70 | 2.60 | 0.31 | **3.60** | **4.02** | 0.23 | ✅ **PASS** |
| **800** | **×8** | 0.69 | 1.65 | 0.30 | **2.63** | **2.94** | 0.22 | ✅ **PASS** |
| 1600 | single | 0.87 | 7.33 | 0.62 | **8.82** | **10.09** | 0.46 | ❌ OVER |
| 1600 | ×8 | 0.85 | 4.43 | 0.58 | **5.86** | **6.67** | 0.44 | ✅ PASS |
| 3200 | ×8 | 1.17 | 12.20 | 1.15 | **14.51** | **16.56** | 0.88 | ❌ OVER |

*(all figures in milliseconds)*

**In the actual running build**, on the same machine: **800 agents at ~120 fps in third person and
~235 fps overhead.** The 60 fps target has a lot of room under it.

### What the numbers say

- **800 passes with room to spare, and 1600 now passes threaded.** The earlier "1600 fails even
  threaded" result was measured *before* the crowd-spreading fix below: packing density, not agent
  count, was the real cost driver. 3200 remains out of reach without the
  [optimisation ladder](#ladder).
- **Separation is ~72% of the cost and tracks local DENSITY, not agent count.** This is the single
  most useful thing M0 taught us: *clumping is what costs money.* A spread-out horde of 1600 is
  cheaper than a tightly packed 800. Every performance lever should therefore be aimed at density,
  not headcount. Hash and integrate are nearly free and scale linearly.
- **Benchmark at peak density or don't bother.** An early run with a 30-frame warm-up caught the
  masses still strolling across open ground and reported 2.26 ms at 800: roughly 2.6× optimistic.
  Any future crowd benchmark must sample when the crowd is actually jammed together.
- **Threading the separation pass gives ~1.5×**, and it is what takes 1600 from OVER to PASS. Not
  needed at 800: keep it available, default off, per the CoW caveat in `agent_sim.gd`.
- **Matching hash cell size to separation radius** (1.5 → 1.2) bought ~12% at 800 and ~22% at 1600.
  A wider cell scans dead area; candidate count scales with cell area.
- **MultiMesh write is a non-issue** at 0.23 ms, writing only the six floats per agent that change.

!!! warning "One caveat that still matters"
    **Headless means no GPU render, no game logic, no netcode.** At 60 fps the whole frame is
    16.67 ms. A 4.02 ms sim leaves ~12 ms for everything else: comfortable, but combat, the corpse
    field and replication all still have to fit in it.

    The mid-range-CPU question is now a **deliberate non-goal**: see the
    [hardware-target decision](../project/decisions.md). Threading exists as the lever if it's ever
    wanted.

### The optimisation ladder: how to go past 800 {#ladder}

Nothing below is built. It's recorded so "800" is understood as *today's* number with a known route
upward, rather than a permanent cap. **Separation is ~72% of sim cost and tracks density**, so every
rung attacks one or the other.

Ordered cheapest-first:

1. **Temporal LOD (time-slicing).** Update separation for 1/N of the agents each frame, round-robin.
   Separation is a smoothing force: running it at 15-20 Hz instead of 60 Hz is invisible even up
   close. Roughly a **3-4× win for a few dozen lines**, and it's the *safest* rung because it
   degrades uniformly everywhere (see the multiplayer caveat below).
2. **Distance-based simulation LOD.** Far agents skip neighbour queries entirely and just follow the
   flow field. They'll interpenetrate, but at distance that's invisible. Combines multiplicatively
   with (1).
3. **Symmetric pairs.** Compute each pair once and apply ±force to both, instead of computing every
   pair twice. A clean **~2×** on the separation pass. Costs the current threading model: parallel
   chunks would need per-thread accumulation buffers.
4. **Ghost tier.** Extreme-distance agents demote to position-only records with no separation at all,
   promoting back when a player nears. **OTR already implements exactly this**
   (`horde_manager`, 300 m promote / 400 m demote with a hysteresis gap): see
   [OTR Carry-Over](otr-carryover.md).
5. **GDExtension.** Still the escape hatch, still unnecessary. The `Scripts/Sim/` boundary keeps it a
   contained job.

Rungs 1, 2 and 4 are all forms of the same idea: **LOD the *simulation*, not just the visuals.** The
third-person camera makes this especially attractive: the player sees a cone, and most of the field
is behind them or far away.

!!! danger "LOD must key off nearest-player distance, NOT the local camera"
    This is the trap, and it's specific to our [split-authority netcode](netcode.md). Each machine
    simulates the units it owns and ships positions to the other. If *my* machine cheapens *my*
    units because they're far from *my* camera, while they're directly in front of *your* camera -
    you watch a mass of units interpenetrating and juddering.

    **Distance LOD must be computed against the nearest of *all* players**, exactly as OTR's
    `horde_manager` does with its cached `_player_positions` list. Temporal LOD (rung 1) doesn't
    have this problem at all, which is another reason to do it first.

### What Total War does, and the one trick we should steal {#total-war}

Worth answering directly, because the honest answer reframes our numbers.

!!! note "Confidence"
    The two mechanisms below are well established from the series' GDC talks and are plainly visible
    in the games. Exact LOD tiers and internals are not something we should assume: treat the
    principles as solid and the specifics as unverified.

**1. Formations are the optimisation.** Total War's unit of simulation is the **regiment**
(~100-160 men), not the man. A regiment has a position, facing and formation shape; each soldier's
position is largely *derived*: `regiment_origin + slot_offset`. So a 6,000-man battle simulates
~40 regiments, and marching soldiers run **no neighbour queries at all**, because they aren't
avoiding each other, they're holding a slot.

A formation is, in effect, a **compression scheme**: one position plus a slot table instead of N
independent positions.

> **This is precisely the thing our design deliberately throws away.** "Horde control, not
> formations" means sloppy, emergent, individually-positioned mass, so we pay full per-agent price
> where Total War pays almost nothing. Our 800 versus their thousands is not us being worse at
> engineering; it's the bill for the product. Worth accepting knowingly.

**2. Melee is resolved as locked pairs, not continuous collision.** When two regiments clash,
soldiers are matched into 1v1 duels playing synchronised paired animations. Once paired they're
**pinned in place**: no pathfinding, no separation, just an animation state machine. It's why Total
War melee reads as a churning mat of duelling pairs.

#### The trick we should steal, and it's a big one

**Agents locked in melee don't need separation.** They need to stand still and hit each other.

Look at where our cost actually is: peak load is 4.11 ms at 800, ~3× open-ground cost, and it occurs
exactly when both masses jam into the chokepoint. But **in the real game, those agents would be
fighting**: and a fighting agent is stationary and paired.

So if "engaged in melee" becomes a state that **skips the separation query entirely**, the worst case
stops being the worst case. Densely packed *combat* would cost *less* than open-field movement, not
three times more. **That inverts the entire cost curve**, and it's the highest-value optimisation on
this page: bigger than anything on the [ladder](#ladder), because it removes work rather than
amortising it.

It also costs nothing aesthetically: locked, grinding melee is exactly what a horde crush should look
like, and it *fixes* the predicted interpenetration problem in the same move: engaged agents hold
position instead of being shoved through each other by separation forces.

**Caveat:** this can't be measured until M1/M2, when agents can actually fight. Until then the 800
figure stands as the honest no-combat number. But it means the *combat* build may well be cheaper
than this spike, which is the opposite of the usual direction.

### Scattered slots: the option that gets us to thousands {#scattered-slots}

Total War: Warhammer's **undead units are formations with scattered slots.** Zombies and skeletons
shamble in a loose, ragged mob, and underneath they're still a regiment with a slot table. The
compression is intact; only the *slot offsets* are jittered and the spacing loosened.

That's the important observation, because it means:

> **Formation compression and "sloppy horde" appearance are orthogonal.** We assumed we had to pay
> full per-agent price to get the look. We don't.

#### The model

- A **clump** has an origin, a facing, and a slot table with **jittered offsets**.
- An agent's rest position is `clump_origin + slot_offset`: derived, ~free, no neighbour query.
- An agent's actual position is its rest position **plus a deviation**.
- **Deviation is what disturbances create**: spell impulses, enemy contact, terrain. It decays back
  toward the slot over time: a spring, essentially.

So the sim is **cheap when calm and expensive only where something is happening**, which is exactly
where the cost *should* go. It also composes with everything above: melee-locked agents simply stop
returning to slot.

Crucially, clumps would be **emergent, not authored**: agents heading for the same rally point form
one. The player never selects, names, or shapes a clump.

#### What it costs

**Emergent local behaviour.** A slot-derived agent doesn't individually flow around an obstacle,
doesn't pile organically at a chokepoint, doesn't find its own way through a gap. Some of that comes
back via the deviation term, but not all of it. The mass becomes *choreographed-then-disturbed*
rather than *genuinely emergent*.

Whether that's a real loss or an invisible one is a **question for the eye, not the spreadsheet** -
and it's the same question as [Q14](../project/open-questions.md#q14).

!!! tip "This is not a violation of 'no formations'"
    [Horde Combat](../game/combat.md) rejects **formations as a control abstraction**: no unit
    selection, no shapes, no facings, no command tree. It says nothing about **formations as a
    spatial compression**, which is an implementation detail the player never sees or touches.
    Borrowing RTS machinery under the hood is fine as long as it never surfaces in the control scheme
    or the silhouette.

#### The netcode payoff is possibly bigger than the CPU one

Replicate **clump origin + seed** instead of per-agent positions and bandwidth collapses from
~8 bytes × N to ~16 bytes × *clump count*. Deviating agents still need individual updates, but
deviation is rare and local, and **OTR's dirty-flag delta sync already handles exactly this shape**:
quiet entities cost zero bandwidth, disturbed ones cost a packet. See
[OTR Carry-Over](otr-carryover.md).

That may be what makes 2,000+ agents viable *in multiplayer specifically*, where the per-agent wire
cost bites hardest.

#### When to decide

**Not now.** It's a real architectural commitment with a real trade, and two cheaper things come
first: answer [Q14](../project/open-questions.md#q14) (if 300 reads as a horde, none of this
matters), and measure melee-lock in M2. Keep it as the known route to thousands if we ever want them.

### The bug M0 found: a flow field alone makes a queue, not a horde {#queue-bug}

**The most valuable thing this spike produced.** A ground-level screenshot showed what looked like a
solid *wall* of agents. Rather than guess, the spike grew a `-- diag` mode that prints a coarse
occupancy map, because "traffic jam" and "broken transform buffer" look identical from eye level.

The verdict was neither. With 800 agents the crowd had collapsed into a **4-metre-wide column**
(`x ∈ [-1.0, 3.0]`) running 90 m north-south. A single-file queue.

**Cause:** every agent samples the *same* flow field toward the *same* goal, so all 800 walk the
identical centreline. Separation pushes back until it balances the field pulling them in, and that
equilibrium is 4 m. Nothing was broken: the design was simply wrong.

**Fix:** each agent gets a **persistent lateral bias** in [-1, 1], applied as a perpendicular
component of the flow direction (`lateral_spread`, currently 0.85). They fan into a front and
converge only where geometry forces it. Extent went **4 m → 55 m wide**, and the shape reads as two
armies advancing rather than two queues.

!!! tip "The fix for the look was also the fix for the frame time"
    Spreading the crowd cut separation cost by ~37% at 800 (4.11 → 2.60 ms) and took **1600 from
    failing to passing**. Because separation tracks local density, *anything that stops the horde
    balling up is a performance optimisation as well as a visual one.* Keep this in mind for the
    combat design: a mechanic that encourages tight clumping is expensive twice over.

This is also a preview of the [scattered-slot model](#scattered-slots): a per-agent persistent offset
is a poor man's slot table.

### Standing still: the horde needs a formation, just not while walking {#settling}

A horde that *stops* still has to arrange itself. Three bugs surfaced while building that, each
invisible until measured:

**1. The goal was a density singularity.** The flow field walks every agent onto the single goal
cell, manufacturing the worst possible packing right where the crowd gathers, and separation cost
tracks density. Fixed with an **arrival radius**: an agent stops once its *path* distance to the goal
(read from the flow field's cost grid, so walls are handled for free) drops below a threshold that
**scales with crowd size**. A bigger horde makes a *wider* blob, not a denser one.

**2. Agents were stranding inside walls.** With no physics there was no collision, so the lateral
bias walked agents into blocked cells: where flow direction is zero and cost is infinite, so they
sat there permanently. It looked like a settling failure (a "blob radius" of 64 m for 100 agents,
which is exactly the wall-to-goal distance). Fixed with **per-axis movement resolution** so agents
slide along geometry, plus an **escape direction** baked into blocked cells so anything that does get
inside can walk out.

**3. Separation cancels in a uniform crowd.** A linear `(r−d)/r` repulsion sums to ~zero for an agent
surrounded on all sides, so a compressed blob's interior can never push itself apart: 93% of agents
were interpenetrating and stayed that way. Fixed by weighting the force **1/d** (clamped), so a very
close pair dominates whatever else is around. Local overlaps now always resolve.

Measured after all three (clipping = bodies actually intersecting, at the 0.55 m body width):

| agents | blob radius | mean speed | clipping | worst clip |
|---:|---:|---:|---:|---:|
| 100 | 6.8 m | 0.17 m/s | **0.0%** | 0.00 m |
| 400 | 14.0 m | 0.22 m/s | **7.0%** | 0.11 m |
| **800** | **19.5 m** | **0.31 m/s** | **20.8%** | **0.22 m** |
| 1600 | 27.3 m | 0.39 m/s | 56.2% | 0.25 m |

**Honest read:** clean to 400, acceptable at the 800 target (worst overlap 0.22 m on a 0.55 m body),
and poor at 1600: where the crowd arrives through a chokepoint over a long window and compresses
faster than it can relax. That last one is tuning, not structure, and **[melee-lock](#total-war)
will change it entirely** since fighting agents hold position instead of being shoved.

**Cost:** wall collision added ~0.5 ms to the integrate pass at 800 (0.31 → 0.79 ms). Worth it: the
alternative was agents walking into geometry. 800 still passes at **4.13 ms avg / 4.52 ms p95**.

### The enemy is the cheap half of the budget {#enemy-formations}

The living enemy fights in **disciplined formations**: which is thematically the point (order versus
chaos) and, conveniently, is the [Total War compression](#total-war) handed to us for free:

- A formation's soldiers are `formation_origin + slot_offset`. **Slots guarantee non-overlap by
  construction**, so troops holding ranks need *no neighbour queries at all*: not even while the
  formation marches, since the origin moves and the offsets don't.
- Separation is only needed **between** formations, and between formation troops and the horde.

So the two sides of a battle have wildly asymmetric cost. Rough shape: a formation soldier is
somewhere near a tenth of the price of a horde agent. **The ~800 budget is not 400-vs-400**: it's
more like *400 horde + a considerably larger enemy force*, or the same enemy force with the savings
spent on more undead.

This should be measured properly during M2 rather than assumed, but it means the enemy side is
close to free, and the design shouldn't budget as though a soldier and a skeleton cost the same.

### Fixed: the flow-field rebuild hitch

Rebuild costs ~40 ms at a 100×100 grid: two dropped frames, triggered on **every rally-point move**.
Now **runs on a `WorkerThreadPool` task**, writing into back buffers that nothing else reads, with the
finished field swapped in on the main thread. The sim keeps sampling the previous gradient for a few
frames, which is invisible; a stutter would not have been. Repeated clicks collapse to at most one
queued rebuild rather than a backlog.

### Reproducing

```bash
Godot_v4.7-stable_win64_console.exe --headless res://Scenes/M0/agent_spike.tscn -- bench
```

Interactive: run the project. `1`/`2`/`3`/`4` set agent count, `T` toggles threading, left-click
moves the green goal, right-click fires a scatter impulse, `WASD`/wheel move the camera. The overlay
breaks down ms per subsystem: build the instrument first, always.

---

## The architecture in one line

> **Units are indices into flat arrays, drawn by MultiMesh, animated on the GPU, steered by a flow
> field, separated by a spatial hash, and never touched by the physics engine.**

Every part of that is load-bearing. Drop any one and the target is out of reach.

## No physics

Agents carry a position and a velocity. Separation is neighbour repulsion computed from a spatial
hash. Spells apply an **impulse** (`vel += impulse`, decayed over a few frames), which is what gives
a shockwave the feel of scattering a crowd without a solver being involved.

**What we give up:** ragdolls, dismemberment, physics-driven siege. Deaths are a canned tip-over into
a corpse record.

**What we gain besides speed:** the simulation is cheap to reason about, and a mass of agents under
separation forces genuinely *looks* better than 800 capsules jostling: you get flow instead of
grinding.

**The first thing that will look wrong:** units visibly overlapping when a mass compresses into a
chokepoint. Without physics there's nothing hard-stopping interpenetration. Separation tuning is the
mitigation; expect to spend real time on it, and expect it to be the most-noticed visual flaw in
the first playable build.

## The corpse field

The unusual requirement, and *easier* than it would have been with physics: there's no rigidbody
phase to absorb (unlike TowerDrop's coin pile, which had to settle real physics bodies first).

1. A unit dies → canned tip-over animation, still a normal agent, for a second or two.
2. It settles → **converted to a corpse record**: position, quality (what it was in life), faction,
   decay timer. Agent slot freed and reused.
3. Corpse records live in a **spatial hash**. The [Feed](../game/combat.md) verb is a radius query
   against that hash, not a scene-tree search.
4. Re-raise → promote a record back into an agent slot.

Corpses are rendered as a second MultiMesh. Per-corpse cost is a struct and one instance transform,
so 3-5k of them is affordable. **This is the single most important system in the game** and should
be built first ([M1](../project/poc-scope.md)).

## Animation is the real cost

800 skinned meshes will kill the frame budget long before the AI does. The answer is **vertex
animation textures**: bake each animation clip into a texture, sample it in the vertex shader by
instance ID and time offset. Per-unit CPU cost goes to zero and everything draws in a handful of
calls.

This is the one piece of tech that has to be right, and it's a natural extension of TowerDrop's
MultiMesh work. Budget real time for the bake pipeline.

## Navigation

**Flow fields, not per-agent pathfinding.** One field computed per rally point over a grid; each
agent samples the field at its position and adds separation. Cheap, and it produces the mass-flow
look we actually want: 800 individuals each solving A\* would be both slower and worse.

Explicitly **not** using Godot's `NavigationAgent3D` or its built-in RVO avoidance.

## Godot-specific rules

Per the [engine decision](../project/decisions.md), Godot only works here if we decline three of its
conveniences:

1. **No node per unit.** Agents are indices into packed arrays, updated in one loop. Not `Node3D`s.
2. **No `NavigationAgent3D`.** Flow field + spatial hash, as above.
3. **No high-level multiplayer.** See [Netcode](netcode.md).

**Known risk:** GDScript may not hold 800 agents × neighbour queries per frame. Plan: build it in
GDScript with a spatial hash, **measure early**, and if it doesn't hold, port *only the hot loop* to
GDExtension (Rust or C++). Design the agent simulation as a swappable module from day one so that
port stays cheap and contained.

## Budgets to set once there's a build

- Frame budget split: agent sim / flow field / rendering / netcode.
- Corpse decay rate: the only thing bounding corpse-field growth in a long fight.
- LOD by distance *and* importance: distant chaff can be crude; the twenty units around the player
  carry the fidelity.

## Related

- [Netcode Architecture](netcode.md) · [Horde Combat](../game/combat.md) · [Art Direction](art-direction.md)
