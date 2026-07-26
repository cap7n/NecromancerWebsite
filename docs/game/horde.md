# The Horde (Units)

What actually gets raised, and how it differs from a Bannerlord troop roster.

!!! warning "OPEN: proposal, nothing built"

## The design rule

Per [Pillar 2](../pillars.md): **the horde is a resource, not an army.** So the unit design should
avoid everything that makes a player precious about individuals, no XP on a common skeleton, no
named survivors in the rank and file, no upgrade paths you'd be sad to lose.

The roster splits hard in two:

- **Chaff**: anonymous, countless, made on the field, spent freely.
- **Elites**: few, expensive, made deliberately from [Anima](../systems/economy.md), persistent
  across battles, worth caring about.

That split is what lets losses feel free *and* lets the player have attachments.

## Chaff: the mass

| Unit | Made from | Character |
|---|---|---|
| **Risen** | Any fresh corpse, instantly, on the field | The default. Slow, weak, endless. The [Feed](combat.md) verb's output. |
| **Skeleton** | Bone + reagents, prepared in advance | Faster and tougher than Risen, doesn't rot as fast, but must be *made* not *found*. |
| **Ghoul** | Corpse + reagents | Fast. Runs down archers and fleeing men. The flanking mass. |
| **Bloated / Carrier** | Plague corpse | Walks into the line and bursts. Disposable siege/shock. Excellent expression of "spend bodies." |

Chaff should have **no upgrade tree**. Quality comes from the *body you started with*: a fallen
knight raises as a better Risen than a peasant does. That makes the enemy's composition matter to
your economy, which is a lovely feedback loop and free depth.

## Elites: the few

Bought with [Anima](../systems/economy.md), scarce by construction.

- **Death Knights / Champions**: raised from a *named* enemy you personally killed. They keep
  something of who they were. Persistent, upgradeable, mournable.
- **Bone Colossi**: built from Bone, siege-scale, slow. The "big spend."
- **Wraiths / Shades**: incorporeal, ignore terrain, terrify the living. Anti-morale, not anti-armour.
- **Lieutenants** *(placeholder name, players rename their own)*: the most important elite in the
  game. Each projects its own small control aura, which is what lets a **detachment** operate away
  from you. See [the anchor model](combat.md#aura-control).

    **They are summoned by ritual at great effort**, not promoted from battlefield kills, which
    gates them behind holding territory rather than combat luck. See
    [the campaign arc](../systems/progression.md#lieutenant-origin).

    They are deliberately the exception to everything else on this page: **few, persistent across
    battles, expensive in [Anima](../systems/economy.md), and losable.** In a design where all the
    chaff is disposable, lieutenants are the only thing that can actually be taken from you, which
    is what keeps the game from having no stakes at all.

    Consequences worth designing for: killing one is the enemy's best play, so they need protecting;
    the number you own caps how many detachments you can run; and in co-op the other player is an
    anchor too, so two players divide the field for free.

## Decay and upkeep

Undead **rot**. Reagents hold them together. Consequences:

- An idle army is a cost, so sitting still is punished, good, given the [live world clock](../systems/multiplayer.md#time).
- Chaff raised in a battle is *temporary* by default and mostly won't survive the march home. That
  keeps field-raising powerful without letting a single big battle permanently break the campaign.
- **Barrow-vaults** in a [necropolis](settlements.md) suspend decay: that's what makes territory
  worth having.

!!! warning "OPEN: does field-raised chaff persist?"
    If it fully persists, one huge battle wins the campaign. If it fully evaporates, the field-raise
    loop feels weightless outside the fight. Proposed middle: **field-raised units persist but decay
    fast**, so a big win buys you a window of overwhelming force, not a permanent army. Needs
    prototyping.

## Living followers

An open thread worth pulling: **cultists, grave-robbers, and mercenaries** who are alive, need paying,
can enter towns, and don't rot. They'd handle everything the undead can't: trade, infiltration,
daylight, and give the player a second, differently-shaped force. See
[Q5](../project/open-questions.md#q5).

## Related

- [Horde Combat](combat.md) · [Economy](../systems/economy.md) · [Horde Scale & Performance](../tech/performance.md)
