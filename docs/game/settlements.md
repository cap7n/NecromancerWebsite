# Settlements

**Feature 4: villages and towns like Bannerlord, necromancer-themed.**

!!! success "Scoped down 2026-07-26"
    Bannerlord towns carry a garrison, a market, workshops, a keep, an arena and a tavern. **We are
    not building that.** A settlement here is a small number of dials and a short list of things you
    can do to it. See the [Decision Log](../project/decisions.md).

## What a settlement actually is

Three numbers and a type. That is the whole model.

| Dial | Meaning | Moves when |
|---|---|---|
| **Prosperity** | How much the place produces, and how fast its graveyard refills | You extort it, plague it, or leave it alone to recover |
| **Population** | How many bodies exist here, eventually | Births, plague, famine, your raids |
| **Dread** | How cowed it is | You do something visible and awful nearby. Decays on its own |

No market inventory. No workshop production chains. No garrison roster to manage. If a number
doesn't change a decision the player makes, it isn't in the model.

**Settlement types:** village (small, feeds a town), town (the regional node), and necropolis (one
you have converted). Castles are out unless sieges come back.

## What you can do to one

Every verb has to be doable in under a minute with the world clock live.

### Parasitise, the default

- **Extort tribute.** Costs Dread, gives resources, lowers Prosperity a little.
- **Raid the graveyard.** A burst of corpses from a finite pool that refills with Prosperity.
  Enormous Dread cost, and it is the loudest thing you can do.
- **Spread plague.** Raises mortality over time, so corpses accrue passively. Permanently damages
  Prosperity. The purest expression of the economy's central tension.
- **Run a front.** One corrupted contact in the settlement gives you access to buy
  [reagents](../systems/economy.md). Discoverable and burnable by the enemy.

A parasitised settlement keeps producing forever. Push too hard and it collapses, and **a dead
village produces nothing ever again.**

### Convert to a necropolis, the late game

One-way and expensive. The living population is consumed. The place stops producing food, recruits
and Prosperity, and starts producing for you instead.

Buildings are deliberately few. Proposed, not decided:

- **Charnel works.** Corpses into durable material at scale.
- **Reagent vats.** The only self-sufficient reagent source, so late-game independence from living
  markets is *earned*.
- **Barrow-vault.** Dormant units you can wake instantly. A strategic reserve, and the thing that
  makes territory worth holding.
- **Obelisk.** Extends your control aura or Will pool regionally.

Four, not a build tree. Converting is a **strategic wound to your own economy**: you have turned a
renewable corpse farm into a factory. That decision is the interesting part, so it should be visible
and painful.

**Recommendation: both, staged.** Parasitise early, convert late, conversion permanent.

## The visit experience

A **map-level panel**, not a walkable scene. A necromancer cannot be seen in public, so a walkable
town buys very little for a large content cost.

!!! warning "OPEN"
    Walkable settlements are much of what people love about Bannerlord. Deferred, not rejected. See
    the [Backlog](../project/backlog.md).

## How the living respond

Settlements react rather than sit still: evacuate the cemetery, hire mercenaries, wall up, and
**burn their own dead**. That last one is the key counter-verb, because it attacks the
[combat loop](combat.md) directly rather than just fielding more soldiers.

## Related

- [Economy](../systems/economy.md) · [Campaign Overworld](overworld.md) · [Horde Combat](combat.md)
