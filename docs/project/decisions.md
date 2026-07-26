# Decision Log

Decisions that have actually been made, **with the why**. When something moves out of
[Open Questions](open-questions.md), it lands here.

Format: date, what was decided, why, and what it rules out.

---

## 2026-07-25: Project started; five features fixed

**Decided:** the project exists and is defined by five features: multiplayer, a Bannerlord-style
overworld, horde-control combat, necromancer-themed villages and towns, and a deeper economy.

**Why:** stated at kickoff by the project owner. These are requirements, not proposals; the wiki's
job is to serve them.

**Rules out:** nothing yet, no implementation decisions have been made.

## 2026-07-25: Wiki built as MkDocs Material, mirroring TowerDrop

**Decided:** the design wiki is Markdown in `docs/`, MkDocs Material (slate), auto-deployed to
GitHub Pages by Actions on push to `main`.

**Why:** identical to the [TowerDrop wiki](https://cap7n.github.io/towerdrop-docs/) setup, which is
already working and already familiar. Zero new tooling to learn, editable from the browser.

**Rules out:** nothing meaningful. Palette shifted violet/bone; status pills carried over unchanged.

## 2026-07-25: Units are positional agents, not rigid bodies

**Decided:** undead, living troops, and corpses are **positions in space with code-driven
separation**: the TowerDrop enemy model. No physics engine involvement. Agents carry a velocity and
accept **impulses** (`vel += impulse`, decayed) so spells can scatter a crowd, but nothing is
simulated by a physics solver.

**Why:** 400+ rigid bodies is unaffordable at the target count before networking is even considered;
physics is non-deterministic, which poisons the netcode options; and boids-style neighbour repulsion
over a spatial hash both costs less and *looks better* for a flowing mass. It also simplifies the
[corpse field](../tech/performance.md): there is no rigidbody phase to absorb, unlike TowerDrop's
coin pile.

**Rules out:** ragdoll deaths, dismemberment physics, physics-driven siege. Deaths are canned
animation into a corpse record. Accepted cost: units can visibly overlap when a mass compresses into
a chokepoint: separation tuning is the mitigation and this is expected to be the first visual
problem to appear.

## 2026-07-25: 2 players, split-authority networking

**Decided:** **2-player co-op.** Each machine simulates **the units it owns**: your horde on your
PC, your friend's on theirs, enemy units split between them at spawn, and replicates coarse state
to the other. Not deterministic lockstep.

**Why:** the requirement was that both machines do real work rather than one host carrying the
simulation, and ownership-split does that literally. The bandwidth arithmetic makes it easy:
200 units × ~8 bytes (quantised XZ, facing, state, id) × 10 Hz ≈ **16 KB/s each way**. Lockstep
would be cheaper still on bandwidth, but it demands strict determinism, makes desyncs brutal to
debug, and makes joining a battle in progress hard, which would cost us the "ride to your friend's
fight and arrive as reinforcements" moment. Co-op means there is no cheating concern forcing a
strict authority model.

**Rules out:** deterministic lockstep (kept as a fallback if unit counts ever outgrow the
bandwidth). Known friction: two players Feeding the same corpse resolves via corpse ownership, at a
cost of ~1 RTT on a raise that has a cast time anyway.

## 2026-07-25: Scale target: ~800 active agents

**Decided:** **200 undead per player**, so ~400 friendly, plus an enemy force of 200-400, plus a
corpse field of **3-5k records**. Working target ≈ **800 active agents** on the field.

**Why:** stated by the project owner as a target to push against and measure. It is achievable with
positional agents + vertex-animation-texture animation + MultiMesh instancing, and is *not*
achievable with rigid bodies or per-unit skinned animation, which is what makes the two decisions
above load-bearing rather than stylistic.

## 2026-07-25: The player is a caster; no melee combat

**Decided:** the necromancer fights with **spells and commands only**. No sword-and-shield, no bows,
no Bannerlord-style directional melee. The player has a body on the field and is killable, with the
control aura anchored to it.

**Why:** thematically correct, and it is the single largest scope cut available, no weapon variety,
no hitbox fencing, no ballistics, and the player's own animation set drops to roughly four states.
The body is retained (rather than a free camera) because the aura anchored to it is what makes
positioning a skill: pushing it forward is powerful and exposed.

**Rules out:** melee as a fallback, weapon progression, mounted combat.

## 2026-07-25: Art direction: stylized low-poly, not realistic

**Decided:** **flat / low-poly geometry, with colour and atmosphere doing the work.** References:
*Totally Accurate Battle Simulator* and *Valheim*: for look only, not mechanics. Explicitly not
Bannerlord's realism.

**Why:** beyond cost, it's the right answer for a crowd game. Silhouette-first reads at 800 units,
which is the hard problem; emissive colour-coding of allegiance (yours / your friend's / feral /
enemy) is trivial in flat-shaded art and near-impossible to do cleanly in realistic art; and
Valheim proves fog + lighting + palette make low-poly read as premium, which pairs directly with
the night-gameplay idea. It is also close to TowerDrop's existing pipeline.

**Open within this:** *XMODE* was also cited as a reference: its specific qualities aren't captured
here yet and should be added by someone who knows it.

## 2026-07-25: Engine: Godot 4 *(2026-07-26: version fixed at 4.7)*

**Decided:** **Godot 4.7**: same version as TowerDrop. No engine spike, no comparison round.

**Why:** it's what the team is comfortable and productive in, carried over from TowerDrop, so there
is no ramp-up cost and no licensing risk. Secondary but real: proving a networked horde game in
Godot feeds back into the engine's ecosystem, which is a win beyond this project.

Critically, the positional-agents decision above is what makes this defensible. Godot's
weakness for this genre was always its physics and crowd tooling, and we've decided not to use
either. What's left (MultiMesh instancing, custom shaders, ENet) is territory TowerDrop already
proved.

**The conditions attached.** Godot works here *only* if we refuse three of its built-in conveniences:

1. **No node per unit.** 800 `Node3D`s with `_process` will not hold. Agents live in flat packed
   arrays, updated in one loop, with transforms pushed into a `MultiMeshInstance3D`. Units are
   indices, not nodes.
2. **No `NavigationAgent3D` / built-in RVO avoidance.** Per-agent navigation at this count is not
   affordable and doesn't produce the look we want anyway. Use a **flow field** over a grid, computed
   once per rally point, plus local separation from a spatial hash.
3. **No high-level multiplayer** (`MultiplayerSynchronizer`, scene replication). It is built for a
   handful of entities. Send our own quantised binary blobs over an unreliable ENet channel: the
   ~16 KB/s figure above assumes this.

**The known risk:** GDScript may not hold 800 agents × neighbour queries per frame. Mitigation is a
contained one: build it in GDScript with a spatial hash first, measure, and if it doesn't hold,
port *the hot loop only* to GDExtension (Rust or C++). That's one system, not a rewrite, and it
should be designed from day one as a swappable module so the port stays cheap.

**Rules out:** Unity/DOTS and Unreal/Mass. Revisit only if the GDExtension port also fails to hit
the frame budget at the target count.

## 2026-07-25: Time model: world speed derived from player activity

**Decided:** the overworld clock is **inferred from what both players are doing**, not set manually:
both stationary → **paused**; one moving → **1×**; both moving → **3×**; either player in a battle →
forced **1×**. On top, a **shared HUD speed toggle that only applies when both players engage it**
(reference: Total War: Warhammer's speed-up-turn-resolve).

**Why:** it solves the two-player time problem *without* giving up fast-forward, which the previous
recommendation (fixed clock, no acceleration) had sacrificed. Time only accelerates when neither
player would miss anything, so no negotiation UI is needed for the common case. Battles forcing
global 1× keeps a single clock for everyone and makes the reinforcement window legible: the time
your friend has to reach your fight is the time you're experiencing inside it. It also creates a
useful social pressure toward going to help.

**Consequences:** the map can be larger than the fixed-clock model allowed, since travel compresses
itself. Route automation becomes a nice-to-have rather than load-bearing. Battle length is still a
real budget (~5-10 min), and settlement interaction must still work with the world live.

**Three problems this opens, all logged as OPEN on [Multiplayer & Time](../systems/multiplayer.md#time):**

1. **Waiting ≠ idling.** "Stationary = paused" would make camping, ambushing, and resting impossible.
   Needs an explicit `Wait`/`Camp` action that counts as activity.
2. **Hysteresis.** Speed must not judder when someone stops briefly. Needs a trigger delay and a
   smooth ramp.
3. **Time scale must be authoritative and tick-aligned.** One machine computes the scale and
   broadcasts "scale becomes X at tick N". Deriving it locally on both machines will drift them
   apart. Cheap now, painful to retrofit.

## 2026-07-25: Tone: dark comedy; models are low-detail

**Decided:** **lighthearted with an undertone of suffering: gallows humour.** Not grim-dark
serious. Separately, the TABS/Valheim references were about **model detail level**, not the full
tonal package: models are low-detail, and that's a style choice, not a compromise.

**Why:** dark comedy is what makes the [economy](../systems/economy.md), which is literally a
system for farming human suffering: *playable* rather than oppressive. Played straight, the
parasitic loop is grim in a way that fights against a co-op game you play with a mate. Played for
gallows humour, it's the joke. The Dungeon Keeper / Overlord register.

**Consequence worth acting on:** comedy is carried by **writing, audio, and animation personality**,
not by model detail. Unit barks, event text, and death animations are where the character budget
should go: all cheap, all compatible with low-detail models. Also means UI and system naming should
not be po-faced ("Will" may want a funnier name).

## 2026-07-26: OTR is the netcode reference implementation

**Decided:** Necromancer's networking ports from **OTR**
(`C:\Users\Gebruiker\Desktop\OTR\OTR_main`), which already runs the decided architecture in a
shipped-quality 2P co-op codebase: distributed-ownership state sync (`NetworkTickManager`),
Hermite-interpolated snapshot buffers (`StateBuffer`), and, critically, a **200-ghost horde as
pure data records rendered by MultiMesh with promotion/demotion near players** (`horde_manager`),
networked at 8 bytes per ghost with client-side heightmap Y reconstruction.

**Why:** it converts M3 from "design and build novel netcode" into "port and scale proven netcode,"
and it carries years of comment-documented lessons (keepalive for dropped rest packets, 3× interp
delay, MTU chunking, world-ready handshake gating all sync, hysteresis gaps on distance triggers).
The ghost/promotion pattern doubles as the **overworld architecture**: map parties are ghosts,
battles are promotions. Full study: [OTR Carry-Over](../tech/otr-carryover.md).

**Also settled by this:** the connection layer is **Steam lobbies + GodotSteam P2P** (friends-only
lobby, overlay join), as in OTR, not raw ENet with IP addresses.

**Known non-fit:** NTM's dictionary-per-entity sync is for the *few* (necromancers, elites,
parties); the 800-agent mass uses the packed-array path. OTR answers neither flow fields, corpse
persistence, VAT animation, nor variable time scale: those remain ours.

## 2026-07-26: Overworld "moving" is derived from intent, not velocity

**Decided:** the [time model](../systems/multiplayer.md#time)'s "is this player moving?" input is
**"has an active travel order"**: a boolean that flips on discrete events (route set, destination
reached, `Wait`/`Camp` toggled), not a velocity threshold on float positions.

**Why:** it eliminates the hysteresis/judder problem structurally instead of tuning it away.
Discrete events can't flicker, the state is trivially cheap to replicate, and `Wait`/`Camp`
(waiting-as-activity, required for ambushes) falls out naturally as "an order that counts as
activity." A movement grid was considered and isn't needed: overworld movement is click-to-move
with routes anyway, Bannerlord-style. A short speed-ramp stays for feel, but correctness no longer
depends on it.

## 2026-07-26: M0 passed: GDScript holds 800 agents, no GDExtension port

**Measured, not decided.** The M0 spike is built and benchmarked. At the 800-agent target, sampled
**at peak chokepoint density**, the simulation costs **5.07 ms average / 5.96 ms p95
single-threaded** against an 8 ms budget: **3.52 / 4.03 ms** with the separation pass threaded.
Full table: [Horde Scale & Performance](../tech/performance.md).

**What this retires:** the [engine decision](#)'s live risk: "GDScript may not hold 800 agents" -
is answered. **No GDExtension port is needed.** The swappable-module boundary in `Scripts/Sim/`
stays as insurance, and it stays cheap to keep.

**What it confirms and constrains:** 800 is the right target *and a genuine ceiling*. 1600 fails even
threaded (9.18 ms avg). A bigger horde would need real optimisation, not a config change.
**Separation is 81% of sim cost** and scales super-linearly with local density: it's the only thing
worth optimising.

**Two caveats recorded, not resolved:** the benchmark ran on an **i9-13900K**, and the pillar target
is a *mid-range* machine, where single-threaded 800 could land at 9-12 ms. And headless excludes GPU
render, game logic, and netcode from the 16.67 ms frame. **PASS is provisional until it runs on
target hardware.**

**Method lesson worth keeping:** the first benchmark run used a 30-frame warm-up and measured the
masses still walking across open ground: it reported 2.26 ms at 800, roughly 2.6× optimistic. Peak
density is ~3× the cost of open ground. **Any future crowd benchmark must sample at peak density.**

**One real problem found:** flow-field rebuild costs ~28 ms, a two-frame hitch whenever a rally point
moves. Not urgent (it's input-driven, not per-frame) but visible, and it must be fixed before M2 -
the clean fix is running it on a worker thread, since it's a pure function writing to its own arrays.

## 2026-07-26: Hardware target: mid-range is a non-goal

**Decided:** Necromancer does **not** target weak or mid-range machines. A demanding game that
requires decent hardware is an accepted trade, on the *Star Citizen* precedent, not every game has
to run everywhere, and over-optimising for low-end hardware costs ambition.

**Why:** stated by the project owner. It converts the M0 benchmark's outstanding caveat ("measured on
an i9-13900K; a mid-range CPU could be 1.5-2× slower") from a **must-fix** into an **accepted
limitation**, and it removes a constraint from every future performance decision.

**Amends [Pillar 6](../pillars.md)**, which said "mid-range machine". That wording is now wrong and
the pillar should be re-read as *a strong machine at 60 fps*.

**One caveat worth remembering, specific to us:** in [2-player split authority](../tech/netcode.md)
each machine simulates its own units and ships positions to the other, so **the weaker of the two
machines caps the shared experience**: if your friend's PC struggles, their 200 units judder on
*your* screen. Star Citizen's precedent is a single-player-facing one. Not a blocker, but it means
"decent PC" is a requirement for *both* players, not just the host.

## 2026-07-26: 800 agents is confirmed as the target, and it's enough

**Decided:** ~800 is sufficient for the game we're making. Not a placeholder, not a stepping stone
to thousands.

**Why:** confirmed by the owner after seeing it running: 800 agents at ~120 fps in third person,
~235 fps overhead. The [optimisation ladder](../tech/performance.md#ladder) and the
[scattered-slot model](../tech/performance.md#scattered-slots) stay documented as the known route
upward if that ever changes, but nothing should be *designed* around needing them.

**Note:** post-fix measurements show **1600 now passes threaded** (5.86 ms avg), so there is real
headroom above the target rather than a hard wall at it.

## 2026-07-26: Agents need a per-agent lateral bias, or the horde is a queue

**Found by measurement, not design.** With every agent sampling the same flow field toward the same
goal, all 800 walked the identical centreline and the crowd collapsed into a **4-metre-wide column**
90 m long. Separation could only hold it open that far against the field pulling everyone in.

**Fix:** each agent carries a **persistent lateral bias** in [-1, 1], applied as a perpendicular
component of the flow direction. Crowd extent went **4 m → 55 m wide** and it now reads as an
advancing front.

**Why it matters beyond the look:** separation cost tracks **local density**, so spreading the crowd
cut separation ~37% at 800 and took 1600 from failing to passing. **The visual fix and the
performance fix were the same fix.** General principle worth carrying into combat design: *anything
that makes the horde ball up is expensive twice over.*

**Method note:** this was caught by adding a `-- diag` occupancy-map dump, because from a
ground-level screenshot "traffic jam" and "broken transform buffer" look identical. Measure the
shape; don't squint at it.

## 2026-07-26: Standing formation is emergent; enemy formations are literal

**Decided, two halves:**

**The horde's standing formation is emergent.** When it stops it packs into a blob via separation
alone, no slots, no authored shape. Agents stop at an **arrival radius that scales with crowd size**
(measured from the flow field's cost grid, so walls count), which means more undead makes a *wider*
formation rather than a denser one. Measured at 800: a 19.5 m blob with 20.8% of bodies clipping by
at most 0.22 m of a 0.55 m width. Clean at 400, poor at 1600.

**The living enemy's formations are literal.** Disciplined troops hold `origin + slot_offset`, which
guarantees non-overlap by construction and needs **no neighbour queries even while marching**. This
is right thematically (order versus chaos) and it makes the enemy roughly an order of magnitude
cheaper per soldier than a horde agent. **The 800 budget is therefore not 400-vs-400**: the enemy
side is nearly free, and the design should not cost a soldier as though it were a skeleton. To be
measured in M2, not assumed.

**Three bugs found by measurement along the way**, all recorded on
[Horde Scale & Performance](../tech/performance.md#settling): the goal cell was a density
singularity; agents with no collision were stranding permanently inside walls; and linear separation
force **cancels** in a uniformly packed crowd, so 93% of a settled blob stayed interpenetrating until
the falloff was changed to 1/d.

**Also added:** wall collision, resolved per axis so agents slide rather than stick (+0.5 ms at 800),
and an escape direction baked into blocked cells so strays can walk out.

## 2026-07-26: M0 is a spike, not a foundation; specify the unit system before M1

**Decided:** stop extending the M0 spike. Write a **capability specification** for what the horde and
the living must be able to do, derive the architecture from it, and only then build M1. See
[Unit System Design](../tech/unit-system.md).

**Why:** M0's `AgentSim` is one flat array of identical agents with a single behaviour. Combat,
corpses, unit types, ownership, morale and formations are not "more of that": each adds state and a
pass over the data. Adding them one at a time would produce a system whose shape is an accident of
the order features arrived, held together by tuning constants. Called by the project owner:
*"we can't use this system and just start bolting things on top until it works."*

**What survives from M0:** `spatial_hash.gd` and `flow_field.gd` unchanged; the `Scripts/Sim/`
module boundary; the perf overlay and headless benchmark/diagnostic harness. `agent_sim.gd` keeps its
movement maths but is **rewritten** around the new pipeline. `agent_spike.gd` is a test harness and
gets discarded.

**The architecture it derives:** one unit array tagged by a **behaviour byte**: `FLOCK`
(flow field + separation, the horde), `SLOT` (formation origin + offset, living troops, non-overlap
free by construction), `ENGAGED` (locked in melee, skips both steering and separation). All three
stay in the shared spatial hash as findable participants; only `FLOCK` pays to compute its position.
**The state machine is the optimisation**: a breaking formation is `SLOT → FLOCK`, and a unit
reaching an enemy is `→ ENGAGED`.

**The risk it surfaces:** **target acquisition is a genuinely new cost of the same order as
separation**, which is already 63% of the frame. It must be **time-sliced from the first line of
code** (re-target 1/8th of units per frame). Partly offset because `ENGAGED` units drop out of both
targeting and separation exactly when the crowd is densest. **Net effect unknown until measured** -
that is M1's first job, before any tuning.

## 2026-07-26: Working rule: documentation first, code only to answer questions

**Decided:** from this point the default output is **wiki documentation**. Code gets written only
when a decision genuinely depends on a number we cannot reason our way to: a measurement, not a
feature. Everything else is designed on paper first.

**Why:** stated by the project owner. It follows directly from the
[M0-is-a-spike decision](#) above: the project is at the stage where the expensive mistake is
building the wrong architecture confidently, not building the right one slowly. Prototype code is
cheap to write and expensive to own.

**How to apply:** propose the design in the wiki, get it argued with, and only then implement. When a
measurement *is* needed, build the smallest throwaway that answers the specific question, record the
result and the method in the wiki, and don't keep the harness as a foundation. M0 is the model:
it answered "can GDScript hold 800 agents", found three bugs worth knowing about, and is now
reference material rather than a codebase.

---

!!! note "What's still open"
    See [Open Questions](open-questions.md). The remaining <span class="pill risk">BLOCKING</span>
    items are the time model ([Q2](open-questions.md#q2)) and the horde verb set
    ([Q3](open-questions.md#q3)).
