# Design Pillars

The taste and constraints Necromancer is designed through. These are strong defaults, not laws, but
break them on purpose, not by accident.

!!! warning "OPEN: proposed, not ratified"
    These pillars are a first draft written from the five stated features. They should be argued
    with and edited before anything is built on top of them. A pillar nobody has pushed back on
    hasn't earned its place.

## 1. Prototype the unproven thing, not the familiar thing

The overworld, the towns, and the economy are all **known quantities**: Bannerlord proved they work
and we can copy the shape. **Horde control is the only genuinely unproven mechanic in the pitch**,
and multiplayer is the only genuinely unproven *engineering* problem. Those two are what the
prototype is for. Building a beautiful campaign map before we know whether steering a horde is fun
is building the parts we already know the answer to.

See [Prototype Scope](project/poc-scope.md).

## 2. The horde is a resource, not an army

Every design decision should push toward *spending* undead rather than *preserving* them. If a
system makes the player careful with individual skeletons, it's working against the fantasy. The
correct emotional register is **"throw more bodies at it"**, and the game should reward that and
then punish the player somewhere *else*: in control, in upkeep, in the strategic economy, never in
"you lost 30 skeletons and that was sad."

Corollary: **losses must be replaceable in-fight**. See [Horde Combat](game/combat.md).

## 3. The limit is Will, not headcount

Bannerlord limits you by party size and wages. Necromancer limits you by **how much you can
control**. Units past your control ceiling don't vanish: they go **feral**, and feral undead are
a genuine threat to everyone including you. This gives horde control its texture: the interesting
decision isn't "do I have enough troops" but "am I still holding the leash."

## 4. The living world is the supply chain

The necromancer's inputs come from the living: bodies, reagents, fear, labour. **Total victory over
the living is economic suicide**, and the game should make that legible early. A player who
exterminates a region should feel the shortage. This is what makes the economy "deeper" rather than
just "longer": it's a predator/prey relationship, not a resource-gathering one.

See [Economy](systems/economy.md).

## 5. Multiplayer is a design constraint, not a feature bolted on

Every system gets designed **multiplayer-first**. If a mechanic only works when time can be
fast-forwarded or paused at will, it does not survive contact with two players and must be
redesigned now, not later. This kills a lot of comfortable Bannerlord conventions and we should
accept that up front rather than fighting it for a year.

See [Multiplayer & Time](systems/multiplayer.md).

## 6. The scale target is a number, and everything serves it

!!! success "Decided: ~800 active agents, confirmed by measurement"
    200 undead per player (≈400 friendly), an enemy force of 200-400, and a corpse field of 3-5k
    records. **Measured 2026-07-26: 800 agents at ~120 fps third-person**, sim at 3.60 ms against an
    8 ms budget. 1600 also passes threaded.

    **Amended:** this pillar originally said "mid-range machine". It doesn't: supporting weak and
    mid-range hardware is now a [deliberate non-goal](project/decisions.md). Read it as *a strong
    machine at 60 fps*.

"As many as possible" is not a target. The number decides the architecture, and the architecture
that reaches it is non-negotiable: **positional agents, no physics, GPU animation, flow fields,
MultiMesh**. Drop any one of those and the number is out of reach.

The corollary is that **per-unit physical fidelity is spent**, deliberately: no ragdolls, no
dismemberment. What we buy with it is *count*, plus the thing the fantasy actually needs: thousands
of persistent corpses that can be individually raised. Spectacle comes from **spell impulses
scattering a mass**, not from simulating each body. See
[Horde Scale & Performance](tech/performance.md).

## 7. Readability over spectacle in the crowd

A thousand identical skeletons is noise. The player has to read, at a glance: *where is my mass,
where is theirs, what's about to break, and what's gone feral.* Silhouette, colour-coding of
allegiance/state, and legible massed movement beat per-unit fidelity. If a visual effect makes the
battlefield harder to read, it loses.

## Working-relationship notes

*Carried over from TowerDrop: the same working style applies unless we decide otherwise.*

- **Steer by feel, then numbers.** Get the feel right, then let logging inform the tuning, not the
  other way around.
- **Be honest about status** in this wiki. Half-wired is not "built".
- **Don't allow lazy shortcuts** when proper authoring is the right call.
