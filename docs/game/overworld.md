# Campaign Overworld

**Feature 2.** A strategic map you roam as a party icon, like Bannerlord's Calradia: settlements,
roads, terrain, wandering parties, and encounters that drop you into [battles](combat.md).

!!! warning "OPEN, nothing here is decided"
    The map itself is the *least* risky part of the pitch (the shape is proven). The risk is
    entirely in how it interacts with [multiplayer time](../systems/multiplayer.md#time).

## What we're copying from Bannerlord

- Party icons moving over a continuous 3D terrain map with roads, rivers, chokepoints.
- Settlements as nodes: villages feeding towns, towns anchoring regions, castles gating them.
- Line-of-sight / spotting, party speed as a function of size and cargo, terrain affecting speed.
- Encounters resolved by entering a battle scene, with the map position deciding the terrain.
- A world that continues without you: other factions fight, trade, and change hands.

## What has to change

### 1. Scale: probably much smaller

With [no time acceleration](../systems/multiplayer.md#time), a Calradia-sized map means hours of
real-time riding. Options:

- **A small map** (a single region, a handful of towns, a dozen villages), good for the PoC and
  arguably good forever. A necromancer terrorising *one valley* is a stronger story than one
  conquering a continent.
- **Fast travel between owned/known nodes**, with the risk of interception.
- **Route automation**: queue a journey, then do something else (manage the horde, read reports)
  while it plays out.

**Recommendation: small map + route automation.** Skip fast travel initially, interception on the
road is too good a source of drama to design away.

### 2. Night and day matter

The obvious, free thematic win: the necromancer is **stronger at night**. Undead decay slower, the
horde is faster, the living are blind and afraid. This gives the fixed world clock a purpose: you
plan around nightfall instead of skipping it. Strong recommendation to build this in from day one.

### 3. Your party is a horde, not a warband

- Party speed should scale badly with size (a shambling mass is slow): a real strategic cost to a
  big army that Bannerlord solves with the same lever.
- The horde **decays on the road**: units rot without reagents. See [Economy](../systems/economy.md).
  Travel therefore *costs* army, which is a much more interesting pressure than food.
- Hiding a horde is hard. Being seen has consequences: villages evacuate, factions muster.

### 4. Heat / attention

A necromancer isn't a lord, he's a **problem**. A world-level threat meter that rises as you act and
draws increasingly serious responses (militia → lords → a coalition → an inquisition) replaces
Bannerlord's diplomacy layer with something more thematic and much cheaper to build.

!!! warning "OPEN"
    Heat-vs-diplomacy is [Q5](../project/open-questions.md#q5)-adjacent and needs deciding. Heat is
    cheaper, more thematic, and less flexible; diplomacy is richer and a huge amount of work.

### 5. Living faction AI

[Q9](../project/open-questions.md#q9). A world that only reacts to the player is flat and cheap; a
world with kingdoms fighting each other is alive and expensive. **A middle path:** a small number of
scripted-but-reactive powers with simple goals, plus wandering parties: enough to feel alive without
a full Bannerlord kingdom simulation.

## Related

- [Settlements](settlements.md) · [Multiplayer & Time](../systems/multiplayer.md) · [Economy](../systems/economy.md)
