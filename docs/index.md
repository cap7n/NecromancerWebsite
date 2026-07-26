# Necromancer Design Wiki

**Necromancer** *(working title)* is a **multiplayer necromancy campaign game**: a Bannerlord-style
strategic overworld you roam with a party, dropping into large real-time battles — except your army
isn't soldiers in formation, it's a **horde of the risen** you steer rather than command, and every
corpse on the field is ammunition.

!!! success "M0 is built and passing"
    The agent-simulation spike runs: **800 agents at 5.07 ms average / 5.96 ms p95**, against an
    8 ms budget, sampled at peak chokepoint density. **No GDExtension port needed.** The project's
    biggest technical risk is retired — numbers on
    [Horde Scale & Performance](tech/performance.md).

    The **foundation decisions are made** — engine, units, netcode, scale, art,
    time — and they're recorded with their reasoning in the
    **[Decision Log](project/decisions.md)**. Everything else is still a proposal and marked
    **OPEN**; **[Open Questions](project/open-questions.md)** tracks what's left.

    The one remaining blocker is the **[horde verb set](game/combat.md)** — which is also the
    project's biggest unknown, because it's the part with no proven reference implementation.

---

## The pitch in one line

*Mount & Blade's campaign, played from the wrong side of the graveyard — and shared with friends.*

## The five stated features

These are the user-facing requirements the project was started around. Everything in this wiki
exists to serve them:

| # | Feature | Where it's designed | Risk |
|---|---|---|---|
| 1 | **Multiplayer** | [Multiplayer & Time](systems/multiplayer.md), [Netcode](tech/netcode.md) | 🟢 Model and time both settled; execution risk remains |
| 2 | **Bannerlord-like overworld** | [Campaign Overworld](game/overworld.md) | 🟢 Time model solved; the map itself was never the risk |
| 3 | **Combat like Bannerlord, but horde control not formations** | [Horde Combat](game/combat.md), [The Horde](game/horde.md) | 🔴 **Now the highest risk.** Unproven, and *this is the fun* — prototype it first |
| 4 | **Villages & towns, necromancer-themed** | [Settlements](game/settlements.md) | 🟢 Design work, not technical risk |
| 5 | **Deeper economy** | [Economy](systems/economy.md) | 🟢 Design work, but it's where the theme pays off |

## The core idea

Bannerlord's army is a **budget of trained, expensive, loyal men**. You spend gold to recruit them,
you keep them alive because they're valuable, and you win by using them precisely — formations,
flanks, discipline.

The necromancer inverts every one of those. Your army is **numerous, cheap, expendable, and stupid**.
You don't preserve it, you *spend* it. And crucially:

> **The dead of the battlefield are your reinforcements.** Enemies who fall get back up on your side.
> A fight you're losing can become a fight you're winning purely because the floor is now covered in
> raw material.

That single loop — *combat generates its own resource* — is the thing that makes horde control a
different game from formation control rather than a worse one. Battles snowball. Big fights are
better than small ones. Losing units is fine, and running out of *control* rather than out of
*troops* is the real fail state. See **[Horde Combat](game/combat.md)**.

The strategic layer inherits the same inversion. A necromancer doesn't farm grain, he farms
**consequences**: war, plague, famine, and graveyards all produce the one input he needs. The living
world is his supply chain, which means he can't simply destroy it. See **[Economy](systems/economy.md)**.

## Where the project is right now

| Area | Status |
|---|---|
| Game concept & five pillar features | ✅ Stated |
| This wiki | ✅ Live; 7 of 8 blocking questions closed |
| **Engine: Godot 4** | ✅ **Decided** — see [Engine & Tooling](tech/engine.md) |
| **Units: positional agents, no physics** | ✅ **Decided** — TowerDrop's model, not rigid bodies |
| **Multiplayer: 2-player co-op, split authority** | ✅ **Decided** — see [Netcode](tech/netcode.md) |
| **Scale target: ~800 active agents** | ✅ **Decided** — see [Performance](tech/performance.md) |
| **Player: caster, no melee** | ✅ **Decided** — see [The Necromancer](game/necromancer.md) |
| **Art: low-detail models, dark-humour tone** | ✅ **Decided** — see [Art Direction](tech/art-direction.md) |
| **Time: activity-derived world speed** | ✅ **Decided** — see [Multiplayer & Time](systems/multiplayer.md#time) |
| Horde-control verb set | ⚠️ **The last blocker** — see [Horde Combat](game/combat.md) |
| Economy resource list | 🧪 Proposed, unprototyped |
| PoC scope & milestones | 🧪 Proposed — see [Prototype Scope](project/poc-scope.md) |
| **M0 agent-sim spike** | ✅ **Built & benchmarked — PASS at 800 agents** |
| GDExtension port | ✅ **Not needed** — GDScript holds the budget |

## Where to start reading

1. **[Design Pillars](pillars.md)** — the taste and constraints everything is designed through.
2. **[Open Questions](project/open-questions.md)** — what we're actually arguing about right now.
3. **[Horde Combat](game/combat.md)** — the unproven core verb, and the first thing that should exist.
4. **[Prototype Scope](project/poc-scope.md)** — what the PoC is and, more importantly, isn't.

!!! note "The one rule of this wiki"
    **This wiki records decisions, it doesn't replace making them.** If a page describes something
    that hasn't actually been decided or built, it gets marked (a `!!! warning "OPEN"` box) or it
    lives in [Open Questions](project/open-questions.md). At concept stage almost everything is
    open, and that's correct — stale certainty is worse than an honest "not decided."
