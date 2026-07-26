# Horde Combat

**This is the game's one genuinely novel system, and the first thing that should be prototyped.**
Everything else in the pitch has a proven reference implementation; this doesn't.

!!! warning "OPEN, nothing here is decided or built"
    This page is a design proposal to argue with, not a spec.

## The design problem

Bannerlord's battle game is **formation management**: eight formations, an F1-F6 command tree,
shieldwall/loose/square, morale and discipline, and a cavalry charge you time. It's a game about
*precision with a scarce, valuable resource*.

Feature 3 asks for combat that *feels* like Bannerlord: real-time, physical, on the ground, at
scale, but where the unit of thought is a **horde**, not a formation. The horde must be:

- **numerous**: you should never count your units
- **stupid**: precise orders would be a lie
- **expendable**: losing them is the intended play pattern
- **self-replenishing on the field**: the corpses are the point

## The core loop: the field feeds itself

> **Kill → corpse → raise → kill.** The battlefield is a resource node that both sides are
> standing on.

This is the mechanic that makes horde combat its own game rather than worse RTS. Consequences we
want and should protect:

- **Big fights are better than small fights.** More enemies = more raw material.
- **A losing fight can turn.** Attrition is not a one-way ratchet, so the tension curve is
  U-shaped instead of a slide.
- **Casualties don't hurt**: your own fallen are also raw material. This removes the "save your
  troops" instinct that would kill the fantasy.
- **Position matters for a new reason.** The pile of dead is terrain now. You want to fight *on*
  the killing field, not away from it.
- **The enemy gets a counter-verb for free:** burn the dead, consecrate ground, fight in a place
  with no bodies. That's a whole design space handed to us.

## The verb set

Deliberately tiny. If it needs a command tree, it's failed. These sit *on top of* the
[aura and detachment model](#aura-control): the main body needs none of them, so every verb below is
really a detachment order or a spell.

<div class="opt rec" markdown>

### 1. Rally: where the horde wants to be
A placed marker the horde drifts toward and clumps around. Movement, but *sloppy*: they arrive
as a mass over several seconds, not as a formation on a line.

### 2. Tide: attack-move a direction
Point and they surge. Once committed they're hard to recall, which is the cost. This is the
"charge" and it should feel like releasing something, not steering it.

### 3. Hold: anchor here
Stop drifting, fight what comes. The defensive verb, and the one that lets you split the horde
into a holding mass and a striking mass without ever calling it a "formation."

### 4. Feed: raise the dead in an area
The active necromancy verb, on a cooldown/cost. Everything dead in a radius gets up on your side.
This is the button the whole fight is built around and it should be the most satisfying input in
the game.

</div>

That's four. Add a fifth only when the prototype proves it's missing.

## Control: the aura, and detachments from it {#aura-control}

!!! warning "PROPOSED 2026-07-26, not decided"
    This replaces the earlier four-verb proposal as the working shape of the control scheme. It came
    out of the observation that an aura may be a better fit than formations, but that the horde still
    has to be spread into slots either way.

Two tiers, and only one of them needs commanding.

### The main body lives in your aura

It has no orders and needs none. It clusters around you, moves when you move, and fights what you
walk into. This is most of the horde, most of the time, and **it is what stops the game becoming an
RTS: you steer the bulk of your army by walking.**

### Detachments are portions you split off

A detachment takes a tactical order and operates away from you: hold that ridge, take that gate,
flank around the wood. It does not need you standing next to it.

### The catch, which is the good part: a detachment needs an anchor

The leash *is* the necromancer's aura. Undead outside it degrade and eventually go
[feral](#the-feral-question). So a detachment sent across the field is, by default, a group of units
walking out of your control.

That is the constraint to build on, not to design away. Something has to hold the leash:

<div class="opt" markdown>

### A lieutenant
Projects a small aura of its own. Already in [the roster](horde.md), and this makes lieutenants
**tactically essential rather than merely strong**, which is a much better reason to want one.

### A planted standard, totem or grave-stake
A weaker fixed aura at a spot you choose. Cheap, expendable, and **destructible by the enemy**,
which hands the living a smart thing to do.

### A decaying tether
No anchor at all. The detachment slowly slips toward feral on a timer. Good for a short strike,
useless for holding ground.

</div>

**The number of anchors you own is the number of detachments you can run.** That cap comes from the
fiction rather than from a UI restriction, it is a natural progression axis, and it keeps
[Pillar 3](../pillars.md) intact: the limit is control, never headcount.

### Line-drawing works for both

Drag a line and they distribute across it, X ranks deep: the main body reshapes around you, or a
detachment forms up where you drew. They hold it loosely and abandon it on contact, per
[Placement](#placement-and-what-no-formations-actually-means) below.

### What it costs

**Moving goals cannot use a flow field.** This is the architectural consequence and it is worth
stating plainly: a field takes ~40 ms to rebuild, so a goal that moves every frame (you) can never
have one. The main body therefore steers **directly toward its aura slot**, with separation and wall
sliding, which is fine because its slots are by definition right next to you. Detachments have
*static* goals, so a flow field is exactly right for them.

**One flow field per detachment**: roughly 10,000 cells, ~120 KB, ~40 ms to rebuild off-thread.
Affordable, but it argues for a **cap of 3 or 4 detachments**, which is the same number the anchor
limit suggests. Two independent constraints landing on the same figure is usually a sign the design
is coherent.

**Two control contexts is the real UI risk.** "Am I commanding the main body or a detachment?" has to
be answerable at a glance without a selection UI, or this slides into the RTS we are trying not to
build. That is the thing to prototype, not the maths.

## The Will system

Per [Pillar 3](../pillars.md), the cap isn't headcount, it's **Will** (working name).

- Every raised unit costs upkeep against a **Will pool**.
- Units within your **control aura** are obedient and slightly stronger.
- Units outside it degrade: slow to respond, then ignore orders, then **go feral**: attacking the
  nearest living thing, which includes you and, in multiplayer, your friends.
- Over-raising is therefore self-limiting and *dramatic* rather than a greyed-out button. The
  greedy play is available and it bites.

!!! warning "OPEN: the feral question" {#the-feral-question}
    Feral units attacking allies is thematically perfect and could be miserable in practice
    (griefing, unfun deaths). Alternatives: feral units simply stop and rot, wander off, or become
    neutral to everyone. **Needs a prototype answer, not an argument.**

## Where the player stands

Per [Q8](../project/open-questions.md#q8):
the working assumption is a **third-person caster, physically present and killable**. The control
aura is centred on the player's body, which means:

- Positioning is the core skill: you are the anchor the horde is tethered to.
- Pushing the aura forward into the enemy is aggressive and dangerous.
- Dying mid-battle isn't instant defeat but the horde immediately starts going feral. That's a
  fail-state with drama.

## Placement, and what "no formations" actually means

!!! warning "Revised 2026-07-26: the horde CAN be given a shape"
    An earlier version of this page said "no formation shapes, facings, or spacing controls." That
    is **no longer accurate.** A stated requirement is that the player can **drag a line and have
    the horde divide itself across it, X ranks deep**
    ([H3](../tech/unit-system.md)).

So the distinction is not *shape vs no shape*. It's:

> **The player places a shape. The horde fails to hold it.**

- You drag a line; they *distribute* onto it over several seconds, raggedly, arriving as a mass.
- They hold it **loosely**: jittering around their slot, not standing to attention.
- They **abandon it** the moment they move or make contact, and they don't reform unless told.
- There is no drill, no facing discipline, no shieldwall, no spacing slider.

The living enemy is the contrast: they hold the same kind of slot **precisely**, keep it while
marching, and reform after disruption. Same underlying system, opposite discipline, which is the
whole aesthetic of the game expressed in one parameter.

Total War: Warhammer's undead are the proof this works: zombies and skeletons read as a ragged mob
while still being slot-based regiments underneath. **The sloppy look and the cheap implementation
are orthogonal.** See [scattered slots](../tech/performance.md#scattered-slots).

## What we are *not* doing

- No unit selection boxes. You never click an individual skeleton.
- No drill: no facings, no spacing controls, no shieldwall/wedge/square menu.
- No per-unit micro. If it rewards APM it's the wrong game.
- No morale-driven rout for the undead. **The dead don't flee**: that's their whole advantage, and
  the living enemy's routing is the contrast that sells it.

## Related

- [The Horde (Units)](horde.md): what actually gets raised
- [The Necromancer](necromancer.md): the player's own kit
- [Horde Scale & Performance](../tech/performance.md): how many of them there can be
- [Netcode Architecture](../tech/netcode.md): how hundreds of units survive multiplayer
