# Backlog

The changing task list. For *why* things are the way they are, see the [Decision Log](decisions.md).
For what the prototype is actually for, see [Prototype Scope](poc-scope.md).

**Legend:**
<span class="pill done">DONE</span>
<span class="pill wip">WIP</span>
<span class="pill todo">TODO</span>
<span class="pill idea">IDEA</span>
<span class="pill risk">NEEDS DECISION</span>
<span class="pill parked">PARKED</span>

---

## Now — before any code

- <span class="pill done">DONE</span> ~~Q1: multiplayer shape~~ → **2-player co-op, split authority**
- <span class="pill done">DONE</span> ~~Pillar 6: the unit-count number~~ → **~800 active agents**
- <span class="pill done">DONE</span> ~~Q4: engine~~ → **Godot 4**
- <span class="pill done">DONE</span> ~~Units: rigid bodies or positional agents?~~ → **positional agents, no physics**
- <span class="pill done">DONE</span> ~~Q8: does the player melee?~~ → **no, caster only**
- <span class="pill done">DONE</span> ~~Q2: time model~~ → **activity-derived world speed + shared toggle**
- <span class="pill done">DONE</span> ~~Q11: art tone~~ → **dark humour, low-detail models**
- <span class="pill risk">NEEDS DECISION</span> **Q3: the horde verb set** — 4 verbs proposed, needs sign-off before M2. **Last blocker.**
- <span class="pill done">DONE</span> ~~Pick the Godot version~~ → **4.7** (matches TowerDrop / OTR)
- <span class="pill done">DONE</span> ~~Create the Godot project~~ — Godot 4.7, main scene = M0 spike
- <span class="pill done">DONE</span> ~~Study OTR netcode + horde spawning~~ → documented in [OTR Carry-Over](../tech/otr-carryover.md)
- <span class="pill done">DONE</span> ~~Create the wiki's GitHub repo~~ → [cap7n/NecromancerWebsite](https://github.com/cap7n/NecromancerWebsite), Actions deploy green
- <span class="pill todo">TODO</span> Argue with the [Design Pillars](../pillars.md) and ratify or rewrite them.
- <span class="pill todo">TODO</span> Characterise *XMODE* as an art reference (nobody has yet).

## M0 — Agent simulation spike <span class="pill done">COMPLETE</span>

**Result: PASS at 800 agents.** 5.07 ms avg / 5.96 ms p95 single-threaded vs an 8 ms budget.
No GDExtension port needed. Full numbers: [Horde Scale & Performance](../tech/performance.md).

- <span class="pill done">DONE</span> ~~Define pass/fail numbers before building~~ → 800 agents, sim under 8 ms
- <span class="pill done">DONE</span> ~~Agents as indices in packed arrays~~ → `Scripts/Sim/agent_sim.gd`
- <span class="pill done">DONE</span> ~~Spatial hash + neighbour separation~~ → counting sort, allocation-free, `spatial_hash.gd`
- <span class="pill done">DONE</span> ~~Flow field over a grid~~ → SPFA integration + baked directions, `flow_field.gd`
- <span class="pill done">DONE</span> ~~`MultiMeshInstance3D` transform write-back~~ → 6 floats/agent/frame, 0.23 ms
- <span class="pill done">DONE</span> ~~Perf overlay, ms per subsystem~~ → live overlay + `-- bench` headless mode
- <span class="pill done">DONE</span> ~~Swappable module boundary~~ → `Scripts/Sim/` has no scene-tree access
- <span class="pill done">DONE</span> ~~Try `WorkerThreadPool` on the separation pass~~ → ~1.6×, available, default off
- <span class="pill done">DONE</span> ~~Decide GDExtension now or later~~ → **not needed**; boundary kept as insurance
- <span class="pill done">DONE</span> Spell-impulse scatter (`vel += impulse`, decayed) — proves the no-physics spectacle

### Follow-ups M0 surfaced

- <span class="pill risk">NEEDS FIX</span> **Flow-field rebuild is ~28 ms** — a 2-frame hitch on every rally-point move. Run it on a worker thread (it's a pure function writing to its own arrays). **Before M2.**
- <span class="pill todo">TODO</span> **Re-run the benchmark on a mid-range CPU.** Measured on an i9-13900K; the pillar target is mid-range, where single-threaded 800 could be 9–12 ms. PASS is provisional until then.
- <span class="pill todo">TODO</span> **Q14: judge crowd perception from the third-person camera** (`C`, presets `1`..`6`). If 300 already reads as a horde, 800 is budget to spend elsewhere. **Do this before optimising for more agents.**
- <span class="pill idea">IDEA</span> **Melee-lock: engaged agents skip separation entirely** (Total War's paired-duel trick). Would make dense combat *cheaper* than open-field movement, inverting the cost curve. Highest-value optimisation known; measurable only once M2 has fighting.
- <span class="pill idea">IDEA</span> Optimisation ladder for >800: temporal LOD (time-slicing) → distance sim-LOD → symmetric pairs → ghost tier → GDExtension. See [Performance](../tech/performance.md#ladder).
- <span class="pill idea">IDEA</span> **Scattered slots** — clump origin + jittered slot offsets + a decaying deviation term, à la Total War's undead. The known route to *thousands*: near-free when calm, and it collapses netcode bandwidth to per-clump rather than per-agent. Costs some emergent local behaviour. **Decide only after Q14 and after melee-lock is measured.** See [Performance](../tech/performance.md#scattered-slots).
- <span class="pill risk">NEEDS CARE</span> Any distance-based sim LOD must key off **nearest-of-all-players**, not the local camera — split authority means my cheapened units are in your view.
- <span class="pill todo">TODO</span> Chokepoint overlap is now visible and measurable. Tune separation strength for looks, not just cost.

## M1 — The corpse field

**Design first.** [Unit System Design](../tech/unit-system.md) specifies capabilities and the derived
pipeline — review and ratify it before writing code.

- <span class="pill risk">NEEDS DECISION</span> Ratify or rewrite [Unit System Design](../tech/unit-system.md).
- <span class="pill todo">TODO</span> Rewrite `agent_sim.gd` around the behaviour-byte pipeline (`FLOCK` / `SLOT` / `ENGAGED`). Keep the movement maths, keep `spatial_hash.gd` and `flow_field.gd` as-is.
- <span class="pill risk">MEASURE FIRST</span> **Target acquisition, time-sliced from day one.** Same order of cost as separation; measure before tuning anything else.

- <span class="pill todo">TODO</span> Corpse-field data structure: spatial hash + second MultiMesh + decay timer.
- <span class="pill todo">TODO</span> Death pipeline: canned tip-over → corpse record, agent slot freed and reused. **No ragdoll.**
- <span class="pill todo">TODO</span> 3–4 hand-authored death clips, random pick + random rotation.
- <span class="pill todo">TODO</span> **Feed** verb: radius query against the hash → raise. Make it feel *great*; this is the signature input.
- <span class="pill todo">TODO</span> Corpse quality carried from the source unit (a knight raises better than a peasant).
- <span class="pill todo">TODO</span> Vertex-animation-texture bake pipeline. **The one piece of tech that has to be right.**
- <span class="pill todo">TODO</span> Flat test arena + a spawner for target dummies.

## M2 — Horde control

- <span class="pill todo">TODO</span> Verbs: Rally, Tide, Hold (flow field per rally point already exists from M0).
- <span class="pill todo">TODO</span> Player body: third-person caster, killable, control aura attached to it.
- <span class="pill todo">TODO</span> **Spell impulses** — `vel += impulse` in a radius, decayed. The scatter is the spectacle.
- <span class="pill todo">TODO</span> Will pool + aura falloff + **feral** state.
- <span class="pill todo">TODO</span> Scripted living enemy force with morale and routing (the contrast that sells the undead).
- <span class="pill risk">NEEDS DECISION</span> Do feral units attack allies? Prototype both, judge by feel.
- <span class="pill todo">TODO</span> Emissive allegiance/state colour-coding — first art task, purely functional.
- <span class="pill todo">TODO</span> Separation tuning for chokepoints — **expect this to be the first visible flaw.**

## M3 — Two players, one battle

*Reference implementation for most of this exists in OTR — see [OTR Carry-Over](../tech/otr-carryover.md).*

- <span class="pill todo">TODO</span> Port `NetworkTickManager` (distributed ownership, dirty flags, keepalive, MTU chunking) for the *few* entities: necromancers, elites, parties.
- <span class="pill todo">TODO</span> Port `StateBuffer` (Hermite interpolation, snap distance) for those same entities.
- <span class="pill todo">TODO</span> Horde replication à la `horde_manager`: packed XZ arrays, low Hz, client-side Y reconstruction — extended to per-player ownership.
- <span class="pill todo">TODO</span> Steam lobby flow (GodotSteam, friends-only, overlay join) — lift from OTR `steam_manager` + lobby UI.
- <span class="pill todo">TODO</span> World-ready handshake gating all sync (OTR `Globals` pattern).
- <span class="pill todo">TODO</span> Ownership split: each machine sims its own 200 + half the enemy force.
- <span class="pill todo">TODO</span> Quantised state packing (~8 bytes/unit) + remote interpolation over ~2 intervals.
- <span class="pill todo">TODO</span> Reliable channel for deaths / raises / casts; unreliable for positions.
- <span class="pill todo">TODO</span> **Corpse-race test:** both players Feed the same pile. Must not produce ghost units.
- <span class="pill todo">TODO</span> **Late join into a running battle** + enemy-ownership handoff.
- <span class="pill todo">TODO</span> Bandwidth + fps measurement harness (expect ~16 KB/s each way).

## M4 — Stub overworld

- <span class="pill todo">TODO</span> Tiny map, party icons, live world clock.
- <span class="pill todo">TODO</span> **Activity-derived speed**: paused / 1× / 3× from both players' movement state.
- <span class="pill risk">NEEDS DECISION</span> `Wait`/`Camp` action so waiting isn't idling — **required, or ambushing is impossible.**
- <span class="pill todo">TODO</span> Speed ramp between time states — feel only; correctness comes from order-derived state.
- <span class="pill todo">TODO</span> **Authoritative tick-aligned time scale** ("scale becomes X at tick N"). Design in from the start.
- <span class="pill todo">TODO</span> Shared HUD speed toggle, applies only when both players engage.
- <span class="pill todo">TODO</span> Reinforcement zone + the buffs that extend it.
- <span class="pill todo">TODO</span> Travel + route automation.
- <span class="pill todo">TODO</span> Encounter → spins up an M3 battle instance while the map keeps ticking.
- <span class="pill todo">TODO</span> Day/night, with real mechanical effect.
- <span class="pill todo">TODO</span> Session-length sanity check: what does 2 hours accomplish?

## M5 — One economic loop

- <span class="pill todo">TODO</span> Raid a village → corpses.
- <span class="pill todo">TODO</span> Reagent upkeep + undead decay on the road.
- <span class="pill todo">TODO</span> Village prosperity responding to being raided.

## Ideas / parked

- <span class="pill idea">IDEA</span> Plague as a spreading map-level system with its own simulation.
- <span class="pill idea">IDEA</span> Death Knights raised from named enemies you personally kill.
- <span class="pill idea">IDEA</span> Living cultist followers who can enter towns in daylight.
- <span class="pill idea">IDEA</span> Enemy counter-verb: burning/consecrating the dead.
- <span class="pill idea">IDEA</span> Player-to-player trade with asymmetric access as a co-op hook.
- <span class="pill idea">IDEA</span> A player choosing to be another player's lieutenant rather than a rival.
- <span class="pill parked">PARKED</span> Walkable settlement scenes.
- <span class="pill parked">PARKED</span> Sieges.
- <span class="pill parked">PARKED</span> PvP between necromancers.
- <span class="pill parked">PARKED</span> Persistent-shard multiplayer.
- <span class="pill parked">PARKED</span> Meta-progression across campaigns.

## Housekeeping

- <span class="pill todo">TODO</span> Decide what the `Necromancer builds`, `Necromancer COWORK`, and `Necromancer Marketing` folders are for and note it here.
- <span class="pill todo">TODO</span> Project name ([Q12](open-questions.md#q12)) — "Necromancer" is unsearchable.
