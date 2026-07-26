# Settlements

**Feature 4: villages and towns like Bannerlord, necromancer-themed.**

!!! warning "OPEN, nothing here is decided"
    [Q6](../project/open-questions.md#q6) is the blocking question: do you *conquer* settlements or
    *parasitise* them?

## Bannerlord's model, briefly

Villages produce goods and recruits and feed a bound town. Towns have prosperity, a garrison, a
market, workshops, a keep, an arena, a tavern. Castles gate regions. You take them by siege, hold
them as fiefs, and they generate income and troops.

The necromancer version has to answer: **what does a settlement do for someone who doesn't want
subjects, doesn't eat, and can't shop?**

## The two modes

<div class="opt rec" markdown>

### Mode 1: Parasitise (the early game) · **recommended default**

The settlement stays alive and human. You extract from it without owning it:

- **Tribute**: coerced with [Dread](../systems/economy.md), no siege needed.
- **Graveyard harvesting**: the cemetery is a finite, slowly-refilling node. Raiding it is fast
  income and enormous Dread cost.
- **Fronts**: a corrupted merchant, an undertaker on the payroll, a cult cell in the slums. Your
  only legal-ish access to reagents and markets.
- **Plague / curse**: deliberately raise mortality. Produces corpses *passively* and permanently
  damages prosperity. The purest expression of the economy's central tension.

A parasitised settlement keeps producing. Push too hard and it collapses, and a dead village
produces nothing ever again.

</div>

<div class="opt" markdown>

### Mode 2: Convert to a Necropolis (the late game)

You take it and remake it. **One-way and expensive.** The living population is consumed, and the
settlement stops producing food/recruits/prosperity and starts producing Bone, Reagents, and
undead capacity instead.

- **Charnel works**: corpses → Bone at scale.
- **Reagent gardens / alchemical vats**: the only self-sufficient reagent source, so late-game
  independence from living markets is *earned*.
- **Barrow-vaults**: stored, dormant units you can wake instantly. Strategic reserve.
- **A soul-well / obelisk**: extends the player's control aura or Will pool regionally.
- **Garrison**: undead don't need pay, but they need reagents. A big garrison is a permanent drain.

Converting is a **strategic wound to your own economy**: you've turned a renewable corpse farm into
a factory. Doing it too early starves you. That's the decision that makes the system interesting,
and it needs to be visible and painful.

</div>

**Recommendation: both, staged.** Parasitise early, convert late, conversion permanent. It gives
the campaign a natural arc: hidden predator → open warlord, without needing a scripted story.

## The visit experience

With [no pausing](../systems/multiplayer.md#time), Bannerlord-style deep menu-diving in settlements
is out. Whatever a settlement visit is, it has to be doable **live, in under a minute**, or be
something you set going and walk away from.

Working assumption: settlements are **map-level interactions with a compact panel**, not walkable
3D scenes, at least for the PoC. Walkable towns are a huge content cost and buy little for a player
who can't be seen in public anyway.

!!! warning "OPEN"
    Walkable settlement scenes are a real question for the full game (they're much of what people
    love about Bannerlord). Deferred, not rejected: see [Backlog](../project/backlog.md).

## Living-faction response

Settlements should react to you: evacuate cemeteries, hire mercenaries, wall up, burn their own
dead. **Burning the dead is the key counter-verb**: it attacks the [combat loop](combat.md)
directly and gives the living AI something smart to do.

## Related

- [Economy](../systems/economy.md) · [Campaign Overworld](overworld.md) · [Horde Combat](combat.md)
