# Prototype Scope (PoC)

!!! danger "Read this before building anything"
    The pitch is *Bannerlord, plus multiplayer, plus a novel combat system*. Bannerlord alone was
    ~8 years and a large studio. That's not a reason to abandon the idea — it's a reason to be
    ruthless about what the prototype is. **The PoC is not a small version of the game.** It is a
    test of the two things nobody knows the answer to.

## The two unknowns

| # | Unknown | Question the PoC must answer |
|---|---|---|
| **A** | **Is horde control fun?** | Does steering a mass with 4 verbs and raising the dead mid-fight feel better than commanding formations, or just vaguer? |
| **B** | **Can hundreds of units + persistent corpses run networked?** | Two clients, one battle, stable frame rate, sane bandwidth. |

Everything else in the pitch — overworld, towns, economy — has a **proven reference implementation**.
We know those work. Building them first is building the parts whose answer we already have, and it's
the single most likely way this project quietly dies. Per [Pillar 1](../pillars.md): prototype the
unproven thing.

## Proposed milestones

### M0 — The agent simulation spike <span class="pill done">COMPLETE 2026-07-26</span>

**PASSED.** 800 agents at **5.07 ms avg / 5.96 ms p95** against an 8 ms budget, measured at peak
chokepoint density. **No GDExtension port needed.** 1600 fails even threaded, so 800 is a real
ceiling. Numbers and caveats: [Horde Scale & Performance](../tech/performance.md).

*Original scope, for the record:*
*(No longer an engine comparison — [Godot is decided](decisions.md).)* Time-boxed to days. In Godot:
**800 dumb agents** as indices in packed arrays, drawn by MultiMesh, moving on a flow field with
spatial-hash separation. No combat, no art, no networking. Just: does it hold 60 fps in GDScript?

**Output: the frame budget, and a yes/no on whether the [GDExtension port](../tech/engine.md) is
needed now rather than later.** This is the project's biggest technical unknown and it is cheap to
answer — do it first, before M1.

### M1 — The corpse field <span class="pill todo">TODO</span>
Single-player, one flat test arena. Units fight, die (canned tip-over, no ragdoll), and settle into a
persistent corpse field in a spatial hash. The **Feed** verb raises everything in a radius. No
economy, no UI, placeholder art.
**Answers: is the core loop mechanically satisfying?**

### M2 — Horde control <span class="pill todo">TODO</span>
Add the player as a third-person body with a control aura, plus Rally / Tide / Hold, plus the Will
pool and going feral. Fight a scripted living force.
**Answers: unknown A. This is the milestone the project lives or dies on — judge it on feel and be
willing to hear "no."**

### M3 — Two players, one battle <span class="pill todo">TODO</span>
M2 with a second client. Both players have hordes, both can Feed, both can go feral. Late join into
a running battle.
**Answers: unknown B.**

### M4 — A stub overworld <span class="pill todo">TODO</span>
Only after M3. A tiny map, 2–3 settlements, a live world clock, travel, and an encounter that spins
up an M3 battle. **Deliberately ugly and deliberately small.**
**Answers: does the [fixed-clock, concurrent-bubble time model](../systems/multiplayer.md#time)
actually feel okay with two people?**

### M5 — One economic loop <span class="pill todo">TODO</span>
Exactly one: **raid a village → corpses → reagents → units → decay**. Not five resources. One loop,
end to end, to see whether the parasitic relationship reads.

## Explicitly NOT in the PoC

- Walkable town scenes
- Diplomacy, clans, marriage, vassals
- Full five-resource economy, trade, smuggling, fronts
- Sieges
- Living faction AI beyond scripted parties
- Art direction, UI polish, audio pass
- More than 2 players
- Any kind of save system beyond "it works for one session"

Every one of these is real, wanted, and *later*.

## The honest risk

The most likely failure mode is not technical. It's spending nine months building a good campaign
map and a beautiful economy, and then discovering at month ten that steering a horde is less
satisfying than commanding formations — because it plausibly is, and only a prototype can tell us.
**M2 should exist within weeks, not months.**

## Related

- [Open Questions](open-questions.md) · [Roadmap](roadmap.md) · [Backlog](backlog.md)
