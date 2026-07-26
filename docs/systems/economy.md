# Economy

**Feature 5: "deeper economy."**

!!! success "Scoped down 2026-07-26"
    **We are not simulating a world economy.** No goods flowing between towns, no supply and demand,
    no price curves, no trade routes to arbitrage. Bannerlord genuinely simulates that and it is an
    enormous system. See the [Decision Log](../project/decisions.md).

## Deep in decisions, shallow in simulation

That distinction is the whole design, and it is worth stating plainly because "deeper economy" was a
stated requirement and this page is cutting things.

A simulated economy is *complicated*: many numbers moving, most of them invisible, and depth is
claimed rather than felt. What actually makes an economy interesting is having **choices that cost
you something**. One good tension beats twelve resource bars.

So we keep the tension and throw away the simulation.

## The tension: you farm consequences

A normal strategy economy is extractive. Land makes grain, grain feeds soldiers.

The necromancer's is **parasitic**. His one essential input, bodies, is produced by the living
world's suffering: war, plague, famine, and simple population.

> **The living are livestock. You need them alive to keep producing.**

Exterminate a region and its output goes to zero forever. Terrorise it *just enough* and it produces
indefinitely. That is a predator and prey relationship rather than a gathering one, and it is the
source of nearly every interesting decision in the campaign. Per [Pillar 4](../pillars.md), it has to
be legible to the player early.

## Three resources

Down from five.

| Resource | Where it comes from | What it's for |
|---|---|---|
| **Corpses** | Battles, graveyards, raids, plague. **Perishable.** | The raw body count. Becomes units directly. |
| **Reagents** | Bought through fronts and black markets, or grown in a converted [necropolis](../game/settlements.md) | **Upkeep.** Undead rot without them. The running cost of an army. |
| **Anima** | Killing the living personally or with intent. Rare, never bulk-farmable. | Elites, named lieutenants, and the player's own power. |

**Dread is not a resource you carry**, it is a property of a *place*: how cowed a given settlement
is. It sits on the settlement, not in your inventory. That removes a stockpile without removing the
mechanic.

**Bone and Material are gone**, folded into Corpses. The corpse-to-bone conversion was one step too
many for what it added.

**Gold barely exists.** A lich cannot walk into a market, so money is only useful through fronts.

### The conversions are where the depth lives

- **Corpses + Reagents make units.** Reagents are the bottleneck, and they come from the *living*
  economy. Your army's running cost is therefore paid by people who hate you.
- **Reagents drain continuously.** An army is a materials cost, not a wage bill. Bigger horde,
  faster burn. That is a natural size limiter that isn't an arbitrary party cap.
- **Anima is never bulk.** It cannot be ground for, so elite power stays scarce by construction.
- **Dread against Prosperity** is the sharpest tension in the design: dread extracts tribute *now*
  but suppresses the settlement's output, which reduces future corpses and reagents. Every
  short-term gain visibly eats the long-term supply.

## Devotion: the living pay for your control {#devotion}

!!! warning "PROPOSED 2026-07-26, not decided"
    Proposed by the designer as a possible mechanic for [Pillar 3](../pillars.md). Written up here
    because it is unusually load-bearing: it turns two ratified pillars from statements of intent
    into rules the game enforces on its own.

**The idea:** you collect living people who venerate you, and their devotion is what keeps the horde
obedient. Undead have hit points *and* draw continuously on that pool. Run dry and they go
[feral](../game/combat.md#the-feral-question). Recovering a feral unit costs Devotion, so if you are
empty, **you cannot get them back, and they stay a danger to you.**

### Why it is worth the extra resource

**It makes [Pillar 4](../pillars.md) literal.** "The living world is the supply chain" stops being a
theme and becomes arithmetic: living people generate the thing that keeps your army from killing you.

**It resolves the Act 4 contradiction.** [Flagged](../game/settlements.md) when the pillars were
ratified: the arc ends in warlord conquest, but conquest means extermination, which Pillar 4 calls
economic suicide. With Devotion, that stops being a design tension and becomes a **loss condition**.
Kill everyone and you have no worshippers, no Devotion, no control, and your own horde turns on you.
The pillar enforces itself, and the endgame is no longer "conquer the map" but "how much can I take
while keeping enough alive to hold the leash."

**It changes the raid verb.** You stop raiding purely to kill and start raiding to **take people
alive**. Capturing is a different action from slaughtering, and having both makes the strategic layer
noticeably richer for very little work.

**It makes a death spiral possible**, which is dramatic and earned: over-raise, Devotion drains, some
units go feral, the feral ones kill your worshippers, Devotion falls further, more go feral. That is
a genuinely frightening failure cascade, and the player will have seen every step of it coming.

### The tensions to design against

!!! danger "The death spiral needs a floor"
    A cascade you cannot break out of is a cascade that makes people quit. There must be an exit:
    **releasing or destroying your own undead to cut the drain**, sacrificing worshippers for a burst
    of Devotion, or a hard minimum trickle. *Recommend the first: unmaking your own army to save
    yourself is a wonderfully thematic panic button.*

!!! warning "Keep it out of the admin layer"
    If worshippers are individuals to be housed, fed and managed, this becomes a colony sim.
    *Recommend: a single number per settlement*, in keeping with the
    [three-dials model](../game/settlements.md).

!!! note "Tone: worshippers, not chattel"
    Played straight, slave management is grim in a way that fights the
    [dark-comic tone](../tech/art-direction.md). *Recommend framing them as a **congregation**:
    deluded cultists, willing worshippers, a flock.* Funnier, more unsettling, and it makes
    "your followers pray to you" a joke that runs the whole campaign.

### Does it replace Reagents?

Probably yes, and that keeps the resource list at three.

<div class="opt rec" markdown>

### Merge: Devotion is the only upkeep
The living keep your undead both **standing and obedient**. One drain, one source, and Pillar 4
becomes the single spine of the whole economy. Simpler, and much sharper.

</div>

<div class="opt" markdown>

### Keep both
Reagents for the body (rot), Devotion for the mind (control). Thematically distinct, but two upkeep
drains is one more bar than the game needs and we just cut the list from five to three.

</div>

**Naming:** *Devotion* is a working name and reads well against **Anima**. The symmetry is worth
keeping: **you take Anima from the dead, you receive Devotion from the living.**

### Decay keeps you moving

Corpses rot, Dread fades, undead fall apart without reagents. Almost nothing stockpiles safely. The
player has to keep *acting* rather than hoarding, and time itself becomes pressure, which matters a
lot given the [live world clock](multiplayer.md#time).

## Buying things without a market

A lich cannot shop. That constraint is content:

- **Fronts.** A corrupted contact in a living settlement. One per settlement, discoverable and
  burnable by the enemy.
- **Black markets and grave-robbers.** The only places selling reagents and corpses openly, at
  terrible rates.

The trade game is therefore **maintaining illicit access**, not buying low and selling high. Much
more thematic, and it generates overworld gameplay without a price simulation behind it.

## Multiplayer

Shared world, **separate stockpiles**, and trade between players. Worth designing toward
deliberately: give the two players *asymmetric access* so trading each other is worthwhile rather
than optional.

!!! warning "OPEN: the shared livestock problem"
    If both players parasitise the same region they compete for a limited living population. Either
    great emergent tension or a friendship-ending frustration. Needs playtesting.

## Related

- [Settlements](../game/settlements.md) · [The Horde](../game/horde.md) · [Campaign Overworld](../game/overworld.md)
