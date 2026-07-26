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

## 2026-07-26 - Economy and settlements scoped down: deep in decisions, shallow in simulation

**Decided:** **no simulated world economy.** No goods moving between towns, no supply and demand, no
price curves, no trade routes. And settlements are **not** Bannerlord towns: no market inventory, no
workshop production chains, no garrison roster, no arena, no tavern, no keep.

**Why:** stated by the project owner. It also resolves a tension the wiki had been carrying: "deeper
economy" is a stated requirement, but a *simulated* economy is complicated rather than deep. Most of
its numbers are invisible, and the depth is claimed rather than felt. What makes an economy
interesting is **choices that cost something**, and the parasitic loop already supplies that. One
good tension beats twelve resource bars.

**What a settlement is now:** three dials (Prosperity, Population, Dread) and a type
(village / town / necropolis). If a number doesn't change a decision the player makes, it isn't in
the model. Necropolis buildings capped at four, not a build tree.

**Resources cut from five to three:** Corpses (perishable), Reagents (upkeep), Anima (rare, elites
and player power). **Bone/Material folded into Corpses** as one conversion step too many. **Dread is
no longer a carried resource**, it is a property of a place, which keeps the mechanic and removes a
stockpile. Gold barely exists, since a lich cannot use a market.

**What is deliberately kept:** the parasitic tension (the living are livestock, and killing a region
kills your income), reagent upkeep as the army's running cost, Dread against Prosperity as the
short-term-versus-long-term dial, and universal decay so the player acts rather than hoards.

**Rules out:** trade as a buy-low-sell-high minigame. Trade is now about **maintaining illicit
access** through fronts and black markets, which is more thematic and needs no price model.

## 2026-07-26 - Lieutenants are anchors, and that settles the control scheme

**Decided in principle** (implementation still open): a detachment operating away from the player
needs something to hold its leash, and **a lieutenant projecting its own small aura is the primary
answer**. Standards and decaying tethers stay as secondary options. Anchors owned equals detachments
runnable.

**Why it earns its place:** it resolves three problems that were being treated as separate.

1. **The control UI.** The open risk was distinguishing "commanding the main body" from "commanding
   a detachment" without a selection box. Anchors make it structural: **you never command units, you
   command anchors.** Your body anchors the main body; each detachment has a lieutenant. The whole
   control scheme becomes at most four addressable things, each with a position and a shape. Legible
   from a third-person camera, and not an RTS selection model.
2. **Enemy counter-play.** Kill the lieutenant and its detachment goes feral. The living AI needs one
   instruction, *find the glowing one*, to be genuinely dangerous. A protect-the-VIP layer inside a
   game about disposable bodies, with the counter-play falling out for free.
3. **Something to lose.** The design insists the horde is expendable and losses should not hurt,
   which is right, but taken alone produces a game where **nothing can be taken from the player**,
   and no possible loss means no tension. Lieutenants are the deliberate exception: few, persistent,
   expensive in Anima, mournable. **The chaff is free precisely so the anchors can be precious.**

**Also falls out:** in co-op the second player's aura anchors units the same way, so two players
divide the field with no additional systems.

**Open under this:** what happens when a lieutenant dies (proposed: a grace period of degradation
rather than instant feral, so there is a scramble rather than a cutscene), where lieutenants come
from, and the name itself, which is a military word in a game that is not played straight.
*Overseer*, *Foreman* or *Bailiff* fit the dark-comic register better.

## 2026-07-26 - Regiments, addressed by hotkey, card and banner. RTS elements are fine.

**Decided:** the horde is divided into a **small number of named regiments during deployment**, each
with an [anchor](../game/combat.md#aura-control). Regiments are addressed three ways, layered:
**hotkeys 1-4** (fast, always reliable), **cards** (state readout, especially proximity to feral),
and **clicking the banner** carried by each anchor in the world.

**Why, and a correction:** earlier drafts treated "RTS" as something to avoid on principle. That was
defending a vocabulary rather than a constraint, and the project owner pushed back correctly: *"we
can use RTS elements if they fit what we want to do."*

The constraint that actually matters is **how many things the player can address and how fast they
must switch**, not whether the widget resembles an RTS. Four regiments on hotkeys is a readable
tactical layer. Forty is APM micro-management, which is a worse version of a game other studios
already make well. **The number is the design; the widget is not.**

**What assigning regiments pre-battle buys:** composition decisions happen in a calm moment rather
than under pressure, and it gives the **deployment phase a purpose**, which had been an open
question. Regiment count stays capped by anchors owned, so the limit still comes from the fiction.

**The banner does three jobs at once**, which is usually a sign a design is holding together: it
selects the regiment, it shows the player where their forces are in a field of near-identical bodies
([Pillar 7](../pillars.md)), and it is the enemy's target, since killing an anchor sends its
detachment feral. The living AI needs one instruction, *go for the banners*.

**Still ruled out:** drag-selecting arbitrary subsets, per-unit micro, and drill controls (facings,
spacing, formation shapes from a menu).

## 2026-07-26 - The campaign arc: you start with one skeleton in a basement

**Decided:** the game opens **small and hidden**, not with a horde.

1. **The basement.** One skeleton, raised in a village cellar. You are pathetic and a single
   militiaman would kill you. Verbs are quiet: rob a grave, poison a well, move a body.
2. **The scavenger.** A dozen. You roam to find *other people's* battles and harvest the aftermath,
   and poison villages to raise mortality for later.
3. **The first holding.** You take a village. Undead become a **workforce**, rituals become possible
   because you have a place, and your **first lieutenant is summoned at great cost**.
4. **The warlord.** Regiments, detachments, open battle, coalitions forming against you.

**Why it matters:** most of this wiki had been describing Act 4 as though it were the starting
position. The arc earns that power fantasy instead of granting it, and it **doubles as the tutorial**
with nothing bolted on: you learn the corpse economy when corpses are scarce, aura control when you
can see all five of your units, and detachments only once you own an anchor.

It also quietly answers [Q14](open-questions.md#q14). The player experiences 1, 10, 50 and 800 units
in sequence, so "what reads as a horde" gets calibrated by playing rather than by us guessing.

**Consequence:** the early game needs a **stealth and attrition layer** that the wiki did not
previously have. Poisoning, grave-robbing, and scavenging old battlefields are Act 1 and 2 verbs,
and they arrive before any combat design does.

## 2026-07-26 - Lieutenants are summoned by ritual, not promoted from kills

**Decided:** ritual summoning at great effort. Replaces the earlier proposal of raising them from
named enemies the player had personally killed.

**Why:** a ritual needs **a place**, so lieutenants are gated behind holding territory rather than
combat luck. That ties your *tactical* ceiling to your *strategic* progress, which is the connection
the campaign layer needs. It also makes the first one an event rather than a drop.

**Players name their own.** Cheap, and it does real work: a named Overseer summoned at ruinous cost
is something the player will actually mind losing, which is the role lieutenants exist to fill.

## 2026-07-26 - Undead are a workforce, not only an army

**Decided:** once you hold a settlement, undead **dig, haul, build and power rituals**.

**Why it is more than flavour:** it puts the single resource under real tension. *Every skeleton
digging is a skeleton not fighting.* Army size and economic output come from the same pool, which is
a genuine strategic dial that costs nothing extra to build because the units already exist, and it
gives the horde something to do between battles other than accrue upkeep.

**Open:** granularity. *Recommend assigning whole regiments to labour*, so it reuses the regiment
system rather than adding a parallel one.

## 2026-07-26 - Feral is immediate, chaotic, and recoverable

**Decided:** feral undead **spread out and attack everything with no cohesion**, hostile to the
living, to the player, and to each other. **Walking your aura over them recovers them**; they get
their master back and resume taking orders.

**Why this beats the grace-period proposal it replaces:** the fail state becomes **a mess you go and
clean up** rather than units deleted, so over-raising stays dangerous without being punishing. It
creates a herding job in the middle of a battle, which is a genuinely novel thing for a player to be
doing. And since feral units also attack the living, a horde that slips its leash is *uncontrolled*
rather than *useless*.

It also disposes of a separate open question: when a lieutenant dies, its detachment simply goes
feral and scatters. The scramble to recover it is the drama, so no timer is needed.

## 2026-07-26 - "No RTS" restated correctly: no overview camera, no per-unit micro

**Decided:** the two things actually ruled out are the **battlefield overview camera** typical of the
genre, and **per-unit micro-management**. RTS elements beyond those are available if they fit.

**Why:** the earlier framing rejected "RTS" as a category, which was defending a vocabulary rather
than a constraint. Correction from the project owner: *"By definition RTS is just real time strategy.
The only thing that we are ruling out is the battlefield overview that typical RTSes have, and the
per unit micro."*

**The consequence worth designing around:** combat is commanded **from inside the fight**, in third
person, so **the player cannot see their whole army.** That is not a limitation to work around, it is
a feature to lean on. It makes [cards](../game/combat.md#addressing) important as a readout of what
you cannot see, it makes banners the way you locate your own forces, and it means a detachment
fighting out of sight is genuinely out of sight.

## 2026-07-26 - Time model reopened: turn-based is back on the table

**Reopened, not decided.** The activity-derived real-time clock recorded on 2026-07-25 is no longer
settled. **Simultaneous turns** with a march range, as in *Total War: Warhammer 3*, is an equal
candidate.

**Why record a reopening at all:** the house rule says a wiki records decisions rather than replacing
them, and quietly editing a decision away is how a design doc starts lying about its own history. The
original entry stays above, marked.

**The useful framing:** battles are real-time in both options, so this is an **overworld-layer**
decision only. Nothing in [combat](../game/combat.md), the [unit system](../tech/unit-system.md) or
battle netcode is affected. Turn-based would delete an entire subsystem we had designed (derived
speed, `Wait`/`Camp`, tick-aligned authoritative timescale) and make overworld netcode trivial, at
the cost of the Bannerlord texture Feature 2 asked for, weaker day/night, and turn-waiting friction.

**Affected if turns win:** [Netcode](../tech/netcode.md) and
[Campaign Overworld](../game/overworld.md) both currently assume real-time and would need revising.
Contained, since it is the overworld layer only.

## 2026-07-26 - Pillars 2, 3 and 4 are ratified and locked

**Decided:** three [design pillars](../pillars.md) are now **fixed**, at the owner's instruction:

2. **The horde is a resource, not an army.**
3. **The limit is Will, not headcount.**
4. **The living world is the supply chain.**

**What "locked" means in practice:** a proposal that violates one of these is wrong by default, and
the fix is to change the proposal rather than the pillar. Revisiting one is its own explicit decision
with its own entry here. It never happens as a side effect of designing something else.

**Worth noticing which three these are.** They are the pillars about *what the game is*. The
remaining four are working practice (1), an engineering constraint (5), a measured budget (6) and a
readability rule (7). Process and tech can bend as we learn; identity should not. Those four stay
open to argument.

**Two consistency checks this immediately surfaced**, both now flagged on the pillars page:

- **Lieutenants versus Pillar 2.** Not a violation. Pillar 2 governs the *horde*, and the chaff is
  free precisely so the anchors can be the one thing the player can lose.
- **Act 4 versus Pillar 4.** [The arc](../systems/progression.md) ends with the player as a warlord
  fighting coalitions, and an endgame of conquest is extermination, which Pillar 4 calls economic
  suicide. The endgame needs a shape that survives the pillar: domination that keeps the living
  producing, an end condition that arrives before the map is stripped, or a necropolis economy that
  genuinely replaces the parasitic one. **Open, and exactly the kind of thing normally discovered too
  late.**

---

!!! note "What's still open"
    See [Open Questions](open-questions.md). The remaining <span class="pill risk">BLOCKING</span>
    items are the time model ([Q2](open-questions.md#q2)) and the horde verb set
    ([Q3](open-questions.md#q3)).
