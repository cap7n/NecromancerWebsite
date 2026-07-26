# Unit System Design

The complete list of what the player, the horde and the enemy must be able to **do**, and the
architecture that falls out of it.

!!! info "How to read this page"
    <span class="pill must">MUST</span> — **stated by the designer.** A requirement, not a proposal.
    Argue with the *implementation*, not the need.
    <span class="pill prop">PROPOSED</span> — a suggestion awaiting sign-off. Not decided.
    <span class="pill done">M0</span> — already proven by the [agent spike](performance.md).

    Keeping these apart is the house rule made visible: a proposal must never read as a decision.

## Why this page exists

[M0](performance.md) answered *"can we afford 800 agents?"* and answered it well — but it is a
**performance spike, not a foundation**. Its `AgentSim` is one flat array of identical agents with a
single behaviour.

Combat, corpses, unit types, ownership, mounts, formations and morale are not "more of that". Each
adds state and a pass over the data. Bolting them on one at a time produces a system whose shape is
an accident of the order features arrived. So: **capabilities first, architecture second, code
third.**

---

## 1. The Necromancer (third-person character)

### 1.1 Movement

| # | Capability | Source |
|---|---|---|
| P1 | Third-person ground movement — walk/run, a body of known height on the field | <span class="pill done">M0</span> stand-in |
| P2 | Mounted movement — ride a horse (or something worse) | <span class="pill parked">MAYBE</span> |
| P3 | Mount / dismount, and what that costs in time or vulnerability | <span class="pill parked">MAYBE</span> |
| P4 | Collide with terrain and geometry; can't walk through walls | <span class="pill prop">PROPOSED</span> |

!!! note "Mounted movement — parked, not cut (2026-07-26)"
    Downgraded from a requirement to a *maybe* by the designer. Recorded here rather than deleted,
    because it splits cleanly into a cheap half and an expensive half, and only one of them is
    actually in doubt:

    **Overworld mounts are nearly free.** The party already moves as an icon on the campaign map; a
    mount is a speed modifier, an icon, and possibly an upkeep cost. No new movement model, no camera
    work, no animation set. If the fantasy of riding matters, this delivers most of it for almost
    nothing.

    **Battle mounts are expensive, and they fight the core design.** A second movement model (turning
    radius, momentum), a second camera treatment, a second animation set — *and* a balance problem:
    the [control aura](../game/combat.md) is anchored to your body, so a mount lets you **outrun your
    own horde**. Either riding ahead slips the leash (interesting, but it means the mount is mostly a
    trap), or the leash stretches (which quietly weakens the whole positioning game). Plus: if you
    can cast while mounted, mounted is strictly better and nobody ever dismounts.

    **Recommendation if it comes back: overworld yes, battle no.** That's the cheap 90%.

### 1.2 Casting

| # | Capability | Source |
|---|---|---|
| P5 | **Damage spells** | <span class="pill must">MUST</span> |
| P6 | **Resurrect** — the Feed verb, raise the dead in an area | <span class="pill must">MUST</span> |
| P7 | **Heal** | <span class="pill must">MUST</span> |
| P8 | **Debuffs** on the living | <span class="pill must">MUST</span> |
| P9 | Impulse/scatter as a spell effect (`vel += impulse`, decayed) | <span class="pill done">M0</span> |
| P10 | Targeting model per spell — self, point, area, unit, cone | <span class="pill prop">PROPOSED</span> |
| P11 | Cast times, cooldowns, and a resource cost | <span class="pill prop">PROPOSED</span> |
| P12 | Can you cast while mounted? While moving? | <span class="pill prop">PROPOSED</span> |

This is a **meaningfully bigger kit** than the wiki previously assumed ("a few spells, kept few").
Four categories — damage, resurrect, heal, debuff — is a spell *system*, with schools and
progression, not a handful of buttons. That's fine, but it should be sized deliberately.

**"Heal" needs defining.** Undead don't heal in the usual sense. Candidates: *mend* (repair damaged
undead), *bolster* (temporary HP), *restore Will*, or *heal living followers/lieutenants*. See **N5**.

### 1.3 Interaction & UI

| # | Capability | Source |
|---|---|---|
| P13 | **Interaction prompts** on world objects | <span class="pill must">MUST</span> |
| P14 | **Menus** | <span class="pill must">MUST</span> |
| P15 | **`C` unlocks the mouse** — free cursor for clicking the world and UI | <span class="pill must">MUST</span> |

P15 is more consequential than it reads. It establishes **two input modes**:

- **Locked** — cursor captured, mouse drives the camera. Action mode.
- **Free** — cursor released, click and drag on the world and UI. Command mode.

Which immediately raises: **is horde commanding done in locked mode (crosshair-aimed) or free mode
(cursor-aimed)?** The line-drag formation (H3 below) essentially requires a cursor — you cannot drag
a line with a crosshair welded to screen centre. So commanding likely lives in free mode, and the
game becomes *third-person action with an RTS command layer you toggle into*. That's a real design
identity and worth adopting on purpose. See **N6**.

---

## 2. The Horde

### 2.1 Movement & positioning

| # | Capability | Source |
|---|---|---|
| H1 | Move toward a point as a mass — flow field, no per-agent pathing | <span class="pill done">M0</span> |
| H2 | **Hold a scattered formation** when standing | <span class="pill must">MUST</span> <span class="pill done">M0</span> |
| H3 | **Form on a line** — drag a line, they divide themselves across it, **X ranks deep** | <span class="pill must">MUST</span> |
| H4 | Spread into a front rather than a single-file queue | <span class="pill done">M0</span> |
| H5 | Not walk into geometry; slide along walls | <span class="pill done">M0</span> |
| H6 | Different speeds by unit type | <span class="pill prop">PROPOSED</span> |

!!! danger "H3 changes the architecture — and it's the most important thing in the braindump"
    "Drag a line, they divide themselves onto it, X deep" is **slot-based placement**. The player
    defines a shape; agents distribute across it. That is exactly the
    [scattered-slot model](performance.md#scattered-slots) — previously filed as a
    *maybe-someday optimisation for the enemy* — promoted to a **core horde control verb**.

    Two consequences, both good:

    1. **The horde and the enemy share one positioning system.** Slots aren't a "living troops"
       feature any more; they're a mode both sides use. The difference is only *how strictly the slot
       is held* — the horde jitters around it and abandons it on contact; soldiers hold it precisely.
    2. **It's cheaper, not more expensive.** A slotted agent's rest position is derived, so it needs
       far less separation work. Giving the player a formation verb makes the horde *cost less* when
       they use it.

    It does need reconciling with "no formations" — see the note under [Horde Combat](../game/combat.md).
    Short version: **the player places a shape; the horde fails to hold it.** That's the difference
    between this and Bannerlord, and the sloppiness is the feature.

### 2.2 Combat

| # | Capability | Source |
|---|---|---|
| H7 | **Attack enemies** | <span class="pill must">MUST</span> |
| H8 | **Ranged attacks — arrows** (to experiment with) | <span class="pill must">MUST</span> *experimental* |
| H9 | **Duel system** à la Total War: Warhammer (to experiment with) | <span class="pill must">MUST</span> *experimental* |
| H10 | Acquire a target — nearest enemy within range | <span class="pill prop">PROPOSED</span> |
| H11 | Take damage, track health | <span class="pill prop">PROPOSED</span> |
| H12 | Die → leave a corpse carrying its quality | <span class="pill prop">PROPOSED</span> |
| H13 | Never rout — the dead don't flee | <span class="pill prop">PROPOSED</span> |

**H8 (arrows) adds a projectile system**, which the plan didn't have. Budget question: individual
projectile entities, or volleys resolved as timed area effects? At this scale, hundreds of arrows in
flight is its own simulation. See **N7**.

**H9 (duels)** is the [melee-lock](performance.md#total-war) idea — a fighting pair holds position
and skips both steering and separation. It's the highest-value optimisation identified so far
*and* a look, and the fact that it's wanted for feel as well as cost is a strong signal.

### 2.3 Control

| # | Capability | Source |
|---|---|---|
| H14 | **A state machine so units know about commands** | <span class="pill must">MUST</span> |
| H15 | Degrade with distance from the control aura → feral | <span class="pill prop">PROPOSED</span> |
| H16 | Cost upkeep against a Will pool | <span class="pill prop">PROPOSED</span> |
| H17 | Belong to a specific player (2-player co-op = two hordes) | <span class="pill prop">PROPOSED</span> |
| H18 | Rot over time without reagents | <span class="pill prop">PROPOSED</span> |
| H19 | Be raised from a corpse, inheriting its quality | <span class="pill prop">PROPOSED</span> |

---

## 3. The Living Enemy

| # | Capability | Source |
|---|---|---|
| E1 | **Hold a strict formation** | <span class="pill must">MUST</span> |
| E2 | **Move in formation** without losing cohesion | <span class="pill must">MUST</span> |
| E3 | **Attack in formation** | <span class="pill must">MUST</span> |
| E4 | **Die and leave an interactable** — raisable, *or* harvestable for magic | <span class="pill must">MUST</span> |
| E5 | Break formation on contact and fight as individuals | <span class="pill prop">PROPOSED</span> |
| E6 | Morale — waver, break, rout | <span class="pill prop">PROPOSED</span> |
| E7 | Ranged volleys (archers) | <span class="pill prop">PROPOSED</span> |
| E8 | Burn or consecrate corpses — the counter-verb | <span class="pill prop">PROPOSED</span> |

!!! tip "E4 is a genuinely good mechanic and it wasn't in the plan"
    "Die and leave something to resurrect **or** leave something behind for us to use in magic"
    makes every corpse a **choice**, not just a resource:

    - **Raise it** → a body now, spent immediately.
    - **Harvest it** → reagents/anima for later, and the corpse is gone.

    That's a per-corpse decision under time pressure (corpses rot, and the enemy can burn them),
    it ties the battle layer directly to the [economy](../systems/economy.md), and it gives the
    player something to do with a battlefield they've already won. Strong recommendation to keep it.

---

## 4. Corpses

A separate system, not a unit state — thousands of records, fast radius queries.

| # | Capability | Source |
|---|---|---|
| C1 | Persist after death; spatially queryable for Feed | <span class="pill prop">PROPOSED</span> |
| C2 | **Be an interactable with two outcomes: raise, or harvest** | <span class="pill must">MUST</span> (from E4) |
| C3 | Store quality, faction-in-life, decay timer | <span class="pill prop">PROPOSED</span> |
| C4 | Render as instances at near-zero per-corpse cost | <span class="pill prop">PROPOSED</span> |
| C5 | Be destroyed by fire/consecration | <span class="pill prop">PROPOSED</span> |
| C6 | Decay away, bounding total count in a long fight | <span class="pill prop">PROPOSED</span> |

---

## Derived architecture

### One unit array, tagged by behaviour

All units — undead, soldiers, elites — live in **one flat set of packed arrays** and one spatial
hash. Not separate systems per side: the horde must find soldiers and soldiers must find the horde,
and two spatial structures would mean cross-querying both. Rendering still splits into a MultiMesh
per mesh type; that's presentation.

What differs per unit is a **behaviour byte**:

- **`FLOCK`** — position emerges from flow field + separation. A horde on the move. Expensive.
- **`SLOT`** — position is `formation_origin + slot_offset`. Used by **both** the enemy (strict) and
  the horde (jittered, via H3's line-drag). Cheap; non-overlap is free by construction.
- **`ENGAGED`** — locked in melee (H9). Holds position, skips steering *and* separation. The densest
  part of the battle becomes the cheapest per agent.
- **`ROUTING`** — broken morale, fleeing. Living only.

A formation breaking is `SLOT → FLOCK`. A unit reaching an enemy is `→ ENGAGED`. A soldier losing its
nerve is `→ ROUTING`. **The state machine is the optimisation** — H14 asked for it as a control
feature, and it doubles as the performance model.

`SLOT` and `ENGAGED` units remain **present in the spatial hash** — findable and collidable, they
just don't pay to compute their own position.

### Per-unit state

Beyond M0's `pos / vel / impulse / faction / bias`:

```
health          float    current hit points
unit_type       byte     risen / skeleton / ghoul / bloated / soldier / archer / ...
behaviour       byte     FLOCK | SLOT | ENGAGED | ROUTING | DYING
owner           byte     player A | player B | living faction | FERAL
target          int32    index of the unit being fought, -1 if none
slot_id         int32    formation index + slot within it (SLOT only)
timer           float    multi-use: attack cadence, decay, dying animation
anim_state      byte     vertex-animation-texture lookup
```

`owner` deliberately separates *which player commands this* from *which side it fights for* — that
makes going feral a change of allegiance rather than a special case, and it's what the
[split-authority netcode](netcode.md) keys off to decide which machine simulates it.

### The update pipeline

```
1. Build spatial hash          all units        O(n)       ~0.7 ms @ 800   [M0 done]
2. Formation/slot update       SLOT units       O(units)   cheap           [new]
3. Target acquisition          TIME-SLICED      O(n·k)     ~= separation   [new, RISK]
4. Combat resolution           ENGAGED pairs    O(pairs)   cheap           [new]
5. Projectiles                 arrows in flight O(shots)   unknown         [new, H8]
6. Separation                  FLOCK only       O(n·k)     ~2.6 ms @ 800   [M0 done]
7. Integrate + wall collision  FLOCK only       O(n)       ~0.8 ms @ 800   [M0 done]
8. Deaths → corpse field       O(deaths)        cheap                      [new]
9. Write render buffers        O(n)             ~0.3 ms @ 800              [M0 done]
```

!!! danger "The one genuinely new cost: target acquisition"
    "Nearest enemy within range" is a spatial query of **the same order as separation**, which is
    already ~63% of the frame. Added naively it could roughly double sim cost.

    **It must be time-sliced from the first line of code** — re-target 1/8th of units per frame
    (≈7 Hz each) is imperceptible and costs an eighth. Cheap to design in, painful to retrofit.

    **Partly self-offsetting:** `ENGAGED` units drop out of both targeting *and* separation, and
    `SLOT` units drop out of separation entirely. Both grow exactly when the battle is densest. Net
    effect is **genuinely unknown until measured** — that's M1's first job, before any tuning.

### What M0 keeps and what it discards

**Keep** — `spatial_hash.gd` and `flow_field.gd` unchanged; the `Scripts/Sim/` no-scene-tree module
boundary; the perf overlay and headless bench/diag harness.
**Rewrite** — `agent_sim.gd`. Movement maths carries over near-verbatim; its *shape* doesn't.
**Discard** — `agent_spike.gd` is a test harness.

---

## New open questions this raised {#new-questions}

| # | Question |
|---|---|
| **N1** | *(Parked with mounts.)* If mounts return: battle, overworld, or both? Can you cast mounted? Can you outrun your own horde? Can the mount be killed under you? |
| **N5** | What does **Heal** mean for undead — mend, bolster, restore Will, or heal living followers? |
| **N6** | Is horde commanding done in **locked (crosshair)** or **free (cursor)** mouse mode? H3's line-drag implies cursor. |
| **N7** | Arrows: **individual projectiles or resolved volleys?** Hundreds in flight is its own simulation. |
| **N8** | Does the **line formation persist** while moving, or collapse to a loose mass the moment they walk? |
| **N9** | How deep is "X deep" — does the player set rank depth, or is it derived from line length and unit count? |
| **N10** | Can the horde be **split into multiple groups** with separate lines, or is it one blob with one shape? |
| **N11** | Is **harvest-vs-raise** a per-corpse choice, an area choice, or a stance? |

## Related

- [Horde Combat](../game/combat.md) · [The Horde](../game/horde.md) ·
  [Horde Scale & Performance](performance.md) · [Netcode](netcode.md) ·
  [Design Questionnaire](../project/questionnaire.md)
