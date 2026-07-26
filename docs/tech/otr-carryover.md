# OTR Carry-Over (Netcode & Horde Reference)

**OTR** (`C:\Users\Gebruiker\Desktop\OTR\OTR_main\OTR`) is the house's proven 2-player co-op
codebase, and it matters enormously here: **it already contains a working embryo of exactly the
architecture Necromancer decided on**: split-authority 2P networking, and a horde rendered as pure
data + MultiMesh with promotion to real entities near players. Studied and documented 2026-07-26.

!!! success "What this means"
    The [netcode decision](../project/decisions.md) (split authority, custom packets, unreliable
    state + reliable events) is **not a plan, it's a port**. The riskiest engineering in M3 has a
    tested reference implementation, including its hard-won tuning constants and comment-documented
    failure lessons.

## The five systems worth carrying

### 1. `NetworkTickManager.gd`: distributed-ownership state sync (autoload)

The core pattern: **every peer broadcasts state for the entities it owns.** Entities implement a
small contract: `get_sync_state()`, `apply_sync_state(data)`, and optionally
`get_sync_owner_peer_id()` (defaults to host). OTR uses this so the drone is simulated on the
operator's machine with zero input lag. **This is Necromancer's split-authority model, already
running.**

Mechanisms worth porting wholesale:

- **Dirty-flag delta sync.** Only entities whose pos/rot changed beyond thresholds get sent
  (0.01 m / ~0.25°). Idle entities cost zero bandwidth. Non-spatial-but-visible fields (steering,
  lock-on paint, engine audio) have their own dirty thresholds: the lesson being *anything a peer
  can see or hear needs a dirty check, not just transforms*.
- **Keepalive resend** (1 s): rest-state packets are unreliable, so if the *last* packet before an
  entity stops moving drops, clients would hold a stale position forever. A periodic clean-entity
  resend self-heals it. Subtle, discovered the hard way, free to inherit.
- **MTU chunking.** Steam's MTU is ~1392 B; packets are split at a 1200 B estimate with per-chunk
  `tick` keys so chunks process independently.
- **Tick→time mapping with drift correction.** Clients anchor the first received tick to local
  time, then space subsequent ticks at exactly the send interval: smooth timestamps regardless of
  network jitter, with a gentle 10% baseline nudge when drift exceeds half an interval. This is
  the exact machinery the [time model](../systems/multiplayer.md#time)'s "scale becomes X at tick N"
  requirement needs.
- **Documented tuning:** `interp_delay = 3×` send interval. The comment records that 2× was tried
  and caused visible snap-back under jitter. 60 Hz sync for 2P on LAN/Steam is affordable.
- **Trust model:** pure 2P co-op, no cheat defence, `any_peer` RPCs. Same call as ours.

**The one thing that does NOT scale:** state is a `Dictionary` keyed by **node-path strings**, ~80 B
per entity. Perfect for OTR's dozens of entities; wrong for 800 agents. **Division of labour:**
NTM-style sync for the *few, important* entities (necromancers, elites, overworld parties);
HordeManager-style packed arrays for the mass (below).

Also noted in the source: the tick-baseline assumes **one remote sender** and says so in an
invariant comment. Fine for 2P; becomes a per-sender map if we ever stretch to 3-4 players.

### 2. `StateBuffer.gd`: snapshot interpolation

Per-entity timestamped snapshot buffer with **Hermite cubic interpolation** (velocity-aware
tangents), quaternion slerp, capped extrapolation (0.2 s) when the buffer runs dry, and a
**snap-distance teleport** instead of interpolating across big gaps.

Best documented lesson in it: **Hermite with zero-velocity tangents degenerates to smoothstep,
which reads as pulsing at low tick rates**: so it falls back to linear lerp when velocities are
zero. That's exactly the class of thing we'd have rediscovered over a lost week.

Reuse: as-is for necromancers, elites, and overworld parties. The horde chaff doesn't need
Hermite: sloppy mass movement hides simple lerp, per the [netcode page](netcode.md).

### 3. `horde_manager.gd`: **the crown jewel**

OTR spawns up to 200 "ghost" enemies on a ring 5 km out and marches them toward the players, and
a ghost is **pure data**: `{pos, target}` in an array. No node, no physics. Rendered by writing
transforms into a **MultiMesh**, with Y snapped to the Terrain3D heightmap so they walk the ground
without colliders. This *is* the [positional-agents decision](../project/decisions.md), already
built and networked:

- **Promotion / demotion with spatial hysteresis.** A ghost within **300 m** of a player promotes
  into a real `enemy.tscn`; a real enemy beyond **400 m** of every player demotes back to a ghost.
  The 100 m gap is explicit hysteresis so entities don't flip-flop at the boundary: the comment
  says so. Promotions/demotions are budgeted (`transitions_per_tick = 5`) to cap instantiation
  spikes.
- **Network: 8 bytes per ghost.** XZ only as a `PackedFloat32Array` at 5 Hz, **Y is recomputed
  client-side from the local heightmap** rather than sent. Chunked at 140 ghosts/packet under MTU.
  Compare our [budget](netcode.md): ~8 B/unit at 10 Hz. OTR already ships that wire format.
- **Cached player positions** refreshed every 0.5 s instead of per-frame group lookups.
- Promoted enemies spawn via reliable name-keyed RPCs on every peer, then sync through
  NetworkTickManager like any entity; a `tree_exited` hook keeps the server registry clean.

**What Necromancer changes:** OTR's ghosts are server-owned; ours split ownership per player -
trivial, since NTM's ownership contract already exists. And our "promotion" isn't to a physics
node but to a *battle-simulated agent*: same shape, cheaper target.

**Bonus:** this is also the **overworld architecture**. Wandering parties, caravans, and patrols on
the campaign map are ghosts: data records marching between settlements, rendered as instances,
promoted to a battle only when engaged. OTR proved it across a 5 km Terrain3D map.

### 4. `enemy_pool.gd` + `enemy_spawner.gd`: pooling and zone spawning

- Pool **pre-warms exact counts** (the spawner sums all spawn points per scene first), instances
  parked out-of-tree with processing disabled; `acquire`/`release`/`pool_reset` contract; falls
  back to fresh instantiation with a debug warning when exhausted.
- Spawn points are group-tagged nodes with an optional `zone_id`; `SpawnZoneTrigger`s fire zones on
  approach. Immediate points spawn on level load.

Reuse: the pool pattern applies directly to our promoted-agent slots and any per-unit visual
attachments. Zone-triggered spawning maps to battle-map deployment.

### 5. `WorldManager.gd` + world-ready handshake: sessions and late join

- **Late join, already solved at the scene level:** on `peer_connected`, the server sends the
  current scene path to the new peer. Joining an in-progress session works today in OTR.
- **The world-ready handshake:** peers are marked ready in `Globals`, and *every* sync path checks
  `is_peer_world_ready(pid)` before sending. This ordering discipline (never sync to a peer still
  loading) is exactly what battle-instance late-join needs, and it's easy to get wrong from scratch.
- Carry-over state (car HP, seats, cargo) collected into dictionaries across scene transitions -
  the same shape our overworld↔battle transitions will use.
- **Transport: GodotSteam**: friends-only lobby, 2 players max, overlay "join friend" flow. This
  answers a question we hadn't asked yet: **Steam lobbies + P2P is the connection layer**, not raw
  ENet with IP addresses. (The MTU constant in NTM is Steam's.)

## What OTR does *not* answer

- **No flow fields**: OTR ghosts march straight at a target. Our M0 flow-field work is still new.
- **No corpse field**: nothing persists after death in OTR. Still ours to build (M1).
- **No VAT animation**: OTR ghosts are static-mesh instances. The bake pipeline is still ours.
- **No variable time scale**: OTR runs 1×. The tick→time machinery helps, but "scale becomes X at
  tick N" is still ours to implement.
- **Scale**: OTR proves 200 data-agents + a handful of synced nodes. 800 + battle sim is our spike
  to run (M0).

## Grid vs floats: answering the hysteresis question

The [time model](../systems/multiplayer.md#time) needs "is the player moving?" to not flicker. The
suggestion was a grid. The OTR-informed answer is simpler: **derive "moving" from intent, not from
velocity.** Overworld movement is click-to-move with a route (Bannerlord-style), so "moving" =
**has an active travel order**, a boolean that flips on discrete events (route set, destination
reached, Wait/Camp toggled). No float thresholds, no judder, nothing to tune, and it's what makes
`Wait`/`Camp` (which must count as activity) fall out naturally. A short ramp between speed states
stays for *feel*, but correctness no longer depends on it. A movement grid is not needed for this -
though spatial hysteresis (OTR's 300/400 promotion gap) remains the pattern for any
distance-triggered state like reinforcement zones.

## Related

- [Netcode Architecture](netcode.md) · [Horde Scale & Performance](performance.md) ·
  [Multiplayer & Time](../systems/multiplayer.md) · [Decision Log](../project/decisions.md)
