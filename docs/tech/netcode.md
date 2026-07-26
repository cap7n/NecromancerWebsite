# Netcode Architecture

!!! success "Decided 2026-07-25"
    **2-player co-op, split-authority.** Each machine simulates the units it owns and replicates
    coarse state to the other. Not deterministic lockstep. Custom binary packets: **not** Godot's
    high-level `MultiplayerSynchronizer`. See the [Decision Log](../project/decisions.md).

!!! tip "This is a port, not a design (2026-07-26)"
    **OTR already runs this architecture**: distributed-ownership sync, snapshot interpolation,
    and a 200-ghost data-driven horde at 8 bytes/ghost: in a working 2P co-op codebase. Study and
    port notes: **[OTR Carry-Over](otr-carryover.md)**. Transport is **Steam lobbies + GodotSteam
    P2P** as in OTR (Steam MTU ~1392 B is the packet-size constraint).

## The model

- **Your machine simulates your 200 undead.** Steering, targeting, separation, deaths.
- **Your friend's machine simulates theirs.**
- **Enemy units are split at spawn**, half owned by each machine.
- Each side sends coarse state for the units it owns; the other side interpolates.

This was chosen specifically because the requirement was that *both machines do real work* rather
than one host carrying the whole simulation. Ownership-split does that literally, and roughly halves
the per-machine agent cost that [performance](performance.md) has to budget for.

## Why this and not lockstep

Deterministic lockstep (both machines simulating everything, only orders on the wire) is cheaper
on bandwidth and elegant. It was rejected because:

- It demands **strict determinism**, and desyncs are brutal to find.
- It makes **joining a battle in progress hard**, which would cost the "see your friend's fight on
  the map, ride for it, arrive as reinforcements" moment: the best co-op hook in the design.
- Bandwidth isn't the binding constraint anyway (see below), so its main advantage buys us nothing.

Kept as a **fallback** if unit counts ever grow past what ownership-split can carry.

## The bandwidth arithmetic

Per owned unit, per update: quantised X/Z (4 bytes, Y sampled from terrain), facing (1), state (1),
id (2) ≈ **8 bytes**.

```
200 units × 8 bytes × 10 Hz = 16 KB/s each way
```

Comfortable on any connection, with headroom for corpse events, spell casts, and overworld traffic.
Interest management and delta compression aren't needed at 2 players: keep them in the back pocket.

## Implementation notes

- **Not `MultiplayerSynchronizer` / scene replication.** Godot's high-level multiplayer is built for
  a handful of entities and will not carry 200 units. Pack our own `PackedByteArray` blobs and send
  them over an **unreliable** ENet channel. The 16 KB/s figure assumes this.
- **Unreliable for state, reliable for events.** Positions are fine to drop: the next packet
  supersedes them. Deaths, raises, and spell casts need the reliable channel.
- **Interpolate remote units** over ~2 update intervals. The horde is *stupid and sloppy by design*,
  which is a genuine networking gift: a skeleton half a metre off between screens is invisible in a
  flowing mass. A shieldwall half a metre off would not be. **Lean on this**: it may be the biggest
  technical advantage the design has.

## The corpse race

Both players can [Feed](../game/combat.md) the same corpse. Resolved by **corpse ownership**: the
machine that owns the record decides who gets it and broadcasts the result. Costs ~1 RTT on a raise
that has a cast time anyway, so it should be invisible in play.

Worth building a deliberate test for this early: two players Feeding the same pile is going to
happen constantly and it must not produce ghost units on either screen.

## The scale lever, if 800 ever isn't enough

Per-agent replication is what makes a big horde expensive on the wire. The
[scattered-slot model](performance.md#scattered-slots) would replace it with **clump origin + seed**
- roughly 16 bytes per *clump* instead of 8 bytes per *agent*: with individual updates only for
agents currently deviating from their slot (spell-scattered, in melee, obstructed).

That maps directly onto OTR's dirty-flag delta sync, which already sends nothing for quiet entities.
It's likely the single change that would make 2,000+ agents viable **in multiplayer specifically**,
where per-agent wire cost bites hardest. Not needed at 800; recorded so the ceiling is understood as
movable.

## Still open

- **Overworld replication.** Lower rate, far fewer entities, much easier, but the
  [time model](../systems/multiplayer.md#time) has to be settled first ([Q2](../project/open-questions.md#q2)).
- **Late join into a running battle.** A hard requirement, since it's the co-op hook. Needs a full
  state snapshot + handoff of ownership for a share of the enemy units.
- **Save/load** of a campaign two people are changing. Host's save as truth is the cheap answer for
  the PoC.

## What the PoC must prove

**Two clients, one battle, ~800 agents, corpses persisting and re-raisable by either player, stable
frame rate, bandwidth in the expected range.** Nothing else. See [Prototype Scope](../project/poc-scope.md).

## Related

- [Horde Scale & Performance](performance.md) · [Multiplayer & Time](../systems/multiplayer.md)
