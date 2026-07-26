# Engine & Tooling

!!! success "Decided 2026-07-25 — Godot 4.7"
    Same version as TowerDrop and OTR. No spike, no comparison round. See the
    [Decision Log](../project/decisions.md) for the full reasoning.

## Why Godot

- **Team comfort.** It's what TowerDrop is built in. No ramp-up, no new tooling, full source access,
  no licensing risk.
- **The physics decision removed the objection.** Godot's weakness for this genre was always crowd
  tooling and physics scale — and we've decided to use [neither](../project/decisions.md). What's
  left is MultiMesh instancing, custom shaders, and ENet, all of which TowerDrop already proved.
- **Ecosystem payback.** Proving a networked horde game in Godot feeds back into the engine's
  ecosystem. A real, if secondary, reason.

## The three conditions

Godot works here **only** if we decline three of its conveniences. Each of these is the difference
between hitting the target and not:

### 1. No node per unit

800 `Node3D`s running `_process` will not hold. Agents are **indices into flat packed arrays**,
updated in a single loop, with transforms written into a `MultiMeshInstance3D`. A unit is an integer,
not a scene.

### 2. No `NavigationAgent3D` or built-in RVO

Per-agent navigation and avoidance at this count is unaffordable, and it produces the wrong look
anyway. **Flow field** over a grid per rally point, plus separation from a spatial hash. See
[Performance](performance.md).

### 3. No high-level multiplayer

`MultiplayerSynchronizer` and scene replication are built for a handful of entities. Pack our own
quantised `PackedByteArray` blobs over an unreliable ENet channel. See [Netcode](netcode.md).

## The known risk, and the plan

**GDScript may not hold ~800 agents × neighbour queries per frame.** This is the most likely thing to
go wrong, and it should be measured early rather than discovered at M3.

The plan:

1. Build the agent simulation in GDScript with a spatial hash.
2. **Measure at the target count from the very first build** — not at the end.
3. If it doesn't hold, port **only the hot loop** to GDExtension (Rust via godot-rust, or C++).

Critically: **design the agent simulation as a swappable module from day one.** A clean boundary
(arrays in, transforms out) makes that port one contained job instead of a rewrite. Also consider
`WorkerThreadPool` for parallelising the neighbour pass before reaching for GDExtension.

## Tooling to carry over from TowerDrop

- **Balance logger** writing CSVs every run, plus an in-game overlay. Feel first, numbers second.
- **Perf overlay from day one** — agent count, frame budget split, bandwidth. Given the risk above,
  this isn't a nicety, it's the instrument the project steers by.
- **Hybrid workflow rule:** systems in code, tunable visual values in `.tres`/scenes the editor can
  adjust without a code round-trip.
- **Editor-conflict rule:** don't write `.tscn`/`.tres` from tooling while the Godot editor has them
  open — it will clobber on save.

## Related

- [Horde Scale & Performance](performance.md) · [Netcode Architecture](netcode.md) · [Decision Log](../project/decisions.md)
