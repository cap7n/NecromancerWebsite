# Design Pillars

The taste and constraints Necromancer is designed through. These are strong defaults, not laws, but
break them on purpose, not by accident.

!!! success "Pillars 2, 3 and 4 are RATIFIED. Do not overwrite them."
    Locked by the project owner on 2026-07-26. These three are **fixed**: a proposal that violates one
    of them is wrong by default, and the correct response is to change the proposal, not the pillar.

    If one ever genuinely needs revisiting, that is its own explicit decision with its own entry in
    the [Decision Log](project/decisions.md). It never happens as a side effect of designing
    something else.

    **The other four remain open** and should still be argued with. A pillar nobody has pushed back
    on hasn't earned its place.

!!! note "Why those three, and not the others"
    Worth noticing: 2, 3 and 4 are the pillars about **what the game is**. The rest are working
    practice (1), an engineering constraint (5), a measured budget (6), and a readability rule (7).
    Process and tech can bend as we learn. Identity should not.

## 1. Prototype the unproven thing, not the familiar thing

The overworld, the towns, and the economy are all **known quantities**: Bannerlord proved they work
and we can copy the shape. **Horde control is the only genuinely unproven mechanic in the pitch**,
and multiplayer is the only genuinely unproven *engineering* problem. Those two are what the
prototype is for. Building a beautiful campaign map before we know whether steering a horde is fun
is building the parts we already know the answer to.

See [Prototype Scope](project/poc-scope.md).

## 2. The horde is a resource, not an army <span class="pill done">RATIFIED</span>

Every design decision should push toward *spending* undead rather than *preserving* them. If a
system makes the player careful with individual skeletons, it's working against the fantasy. The
correct emotional register is **"throw more bodies at it"**, and the game should reward that and
then punish the player somewhere *else*: in control, in upkeep, in the strategic economy, never in
"you lost 30 skeletons and that was sad."

Corollary: **losses must be replaceable in-fight**. See [Horde Combat](game/combat.md).

!!! note "The deliberate exception, which proves rather than breaks the rule"
    [Lieutenants](game/combat.md#anchors-solve) are precious, persistent and mournable. That is not a
    violation: this pillar governs the **horde**, and the chaff is free *precisely so* the anchors
    can be the one thing you can lose. A game where nothing can be taken from you has no tension.

## 3. The limit is Will, not headcount <span class="pill done">RATIFIED</span>

Bannerlord limits you by party size and wages. Necromancer limits you by **how much you can
control**. Units past your control ceiling don't vanish: they go **feral**, and feral undead are
a genuine threat to everyone including you. This gives horde control its texture: the interesting
decision isn't "do I have enough troops" but "am I still holding the leash."

Not to be confused with the ~800-agent figure in Pillar 6. That is a **technical budget**; this is a
**design limit**. If the two are ever mistaken for each other, the design one wins.

!!! note "Proposed mechanism: Devotion"
    Will may be sourced from **living worshippers** rather than being an abstract stat. See
    [Devotion](systems/economy.md#devotion). If adopted, it turns this pillar and Pillar 4 into rules
    the game enforces mechanically rather than statements we have to keep honouring by hand.

## 4. The living world is the supply chain <span class="pill done">RATIFIED</span>

The necromancer's inputs come from the living: bodies, reagents, fear, labour. **Total victory over
the living is economic suicide**, and the game should make that legible early. A player who
exterminates a region should feel the shortage. This is what makes the economy "deeper" rather than
just "longer": it's a predator/prey relationship, not a resource-gathering one.

See [Economy](systems/economy.md).

!!! warning "Act 4 has to be checked against this pillar"
    [The campaign arc](systems/progression.md) ends with the player as a warlord fighting
    coalitions. Taken naively, an endgame of conquest is **extermination**, which is exactly what
    this pillar says is economic suicide.

    So the endgame needs a shape that doesn't break it: domination that keeps the living producing,
    a fixed end condition that arrives before you can strip the map, or a late-game
    [necropolis](game/settlements.md) economy that genuinely replaces the parasitic one.

    **The [Devotion](systems/economy.md#devotion) proposal solves this outright** by making
    extermination a loss condition rather than a design tension. If it is adopted, this warning can
    be closed.

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
