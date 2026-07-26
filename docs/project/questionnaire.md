# Design Questionnaire

**Purpose: get the whole game out of the designer's head and onto paper**, so the framework can be
planned against a complete picture instead of discovered feature by feature.

!!! info "Revised 2026-07-26"
    Rewritten after the capability braindump, which is deconstructed in full on
    [Unit System Design](../tech/unit-system.md). Questions already answered there have been removed;
    questions that braindump *created* are marked <span class="pill must">NEW</span>.

Work through it in sittings: it's deliberately long. Answers move into the relevant design page,
and where they close a real choice, into the [Decision Log](decisions.md) *with the why*.

**Where a recommendation is given, "yes, as recommended" is a complete answer.** Say what you'd
change; a one-word confirmation is fine, but silence isn't agreement.

**Legend:** <span class="pill risk">BLOCKER</span> gates the framework ·
<span class="pill wip">SOON</span> needed for the PoC ·
<span class="pill idea">LATER</span> safe to defer ·
<span class="pill must">NEW</span> raised by the braindump

---

## A. The player character <span class="pill risk">BLOCKER</span>

**A1. Mounts: parked as a "maybe".** <span class="pill parked">PARKED</span> If they come back,
the split is what matters: **overworld mounts are nearly free** (a speed modifier on a party icon),
**battle mounts are expensive** and fight the aura design by letting you outrun your own horde.
*Recommend, if revisited: overworld yes, battle no.* Nothing else needs deciding while it's parked.

**A2. What does "Heal" mean for undead?** <span class="pill must">NEW</span> Mend (repair damaged
undead) · Bolster (temporary HP) · Restore Will · Heal living followers/lieutenants. *These are very
different spells. Possibly more than one of them.*

**A3. How big is the spell kit meant to be?** Damage / resurrect / heal / debuff is four *schools*,
not four buttons. Roughly how many spells total: 6? 12? 30? *This decides whether it's a kit or a
progression system.*

**A4. Is commanding done with the mouse locked or free?** <span class="pill must">NEW</span> The
line-drag needs a cursor. *Recommend: `C` toggles to free-cursor "command mode", the game becomes
third-person action with an RTS layer you drop into. Worth adopting deliberately, it's a real
identity.*

**A5. What's in the menus?** (P14) Inventory, spellbook, horde roster, map, journal?

---

## B. Horde commands & formations <span class="pill risk">BLOCKER</span>

**B1. Is the verb set right?** Rally (drift here) · Tide (attack-move) · Hold (anchor) · Feed
(raise), **plus Line (drag-to-place)**. *Recommend: these five, and add nothing else until the
prototype proves something's missing.* Anything to cut or merge?

**B2. Does the line formation persist while moving?** <span class="pill must">NEW</span> *Recommend:
no: they hold it standing, and collapse to a loose mass the moment they walk. That's precisely
what separates the horde from the enemy, and it's the cheaper option.*

**B3. How is rank depth set?** <span class="pill must">NEW</span> Player-controlled (drag depth,
scroll wheel), or derived automatically from line length ÷ unit count?

**B4. Can the horde be split into multiple groups?** <span class="pill must">NEW</span> Two lines,
or a holding group and a striking group? *This is the single biggest scope fork in the control
scheme: one blob is far simpler; multiple groups edges toward RTS.*

**B5. How does Feed actually work?** Cast time, cooldown, radius, cost. Does it raise everything in
range, or up to a cap? *Recommend: capped by Will, not by a number, so the limit reads as "how much
can I control".*

**B6. What does the state machine need to expose to the player?** (H14) Should you be able to *see*
a unit's state (engaged, feral, obeying)? *Recommend: yes, via emissive colour. It's the readability
pillar.*

---

## C. Horde combat <span class="pill risk">BLOCKER</span>

**C1. Duel system: strict 1v1 pairs, or many-on-one?** <span class="pill must">NEW</span> Total War
pairs duellists. A horde arguably wants five skeletons mobbing one soldier. *Recommend: many-on-one
with a hard cap of 3-4 attackers, so a lone soldier isn't buried under twenty bodies on one tile.*

**C2. Arrows: individual projectiles, or resolved volleys?** <span class="pill must">NEW</span>
*Hundreds of arrows in flight is its own simulation with its own budget. Volleys resolved as timed
area effects are far cheaper and read almost identically at this scale.*

**C3. Do undead get ranged units at all, or is archery only for the living?** <span class="pill must">NEW</span>
*Skeleton archers are a classic, but "the horde" reads as a melee tide: ranged undead may dilute it.*

**C4. What do feral units do?** Attack everything including you and your co-op partner · wander and
rot · go neutral · attack the living only. *No recommendation: needs a prototype answer. Thematically
perfect, potentially miserable in co-op.*

**C5. What happens when the player dies mid-battle?** *Recommend: not an instant loss, the horde
starts going feral. A dramatic collapse rather than a fail screen.*

**C6. The control aura**: does the player trade radius against strength manually, or is it fixed and
upgraded through progression?

**C7. Will**: spent per-unit as upkeep, or a cap on total raised? Does it regenerate in battle?

---

## D. The living enemy <span class="pill wip">SOON</span>

**D1. Who are they?** One kingdom, several rival factions, or a patchwork of local powers?

**D2. Do they fight each other without you?** *Recommend: a few scripted-but-reactive powers plus
wandering parties: alive without a full kingdom simulation.*

**D3. When does a formation break?** Casualty threshold, morale, or contact? *This directly sets how
the enemy feels to fight.*

**D4. Do they rout, and can they rally?**

**D5. What are the counter-verbs to necromancy?** *Recommend: burning corpses, definitely, it
attacks the core loop and gives the enemy AI something smart to do.* Consecrated ground? Priests?

**D6. Heat/threat, or diplomacy?** *Recommend: heat. A necromancer is a problem, not a peer -
militia → lords → coalition → inquisition.*

**D7. Can you ever ally with the living?** Mercenaries, desperate lords, cultist towns?

---

## E. Corpses & the raise/harvest choice <span class="pill wip">SOON</span>

**E1. Is harvest-vs-raise a per-corpse choice, an area choice, or a stance?** <span class="pill must">NEW</span>
*Per-corpse is the most interesting and the most clicking. Recommend: area, with the split decided by
which spell you cast: Feed raises, Harvest reaps.*

**E2. What exactly does harvesting give?** <span class="pill must">NEW</span> Reagents, Anima, or
both depending on what died?

**E3. Does corpse quality matter?** A fallen knight raising better than a peasant makes the enemy's
composition part of your economy. *Recommend: yes, it's free depth.* How many tiers?

**E4. Do field-raised units persist after the battle?** *Recommend: persist but decay fast, a big
win buys a window of overwhelming force, not a permanent army.*

**E5. How long do corpses last, and can the player extend it?**

---

## F. Battles <span class="pill wip">SOON</span>

**F1. Target battle length?** *Recommend: 5-10 minutes, it's live campaign time for your partner.*

**F2. Is there a deployment phase?** With a live world clock, a long pre-battle setup is a problem -
but the line-drag verb makes deployment natural. *Recommend: a short one, on a timer.*

**F3. How does a battle end?** Enemy routs · enemy wiped · objective taken · timer.

**F4. Can you retreat, and what does it cost?**

**F5. How varied is battle terrain?** *Recommend: start flat with walls. Terrain height is a real
chunk of work and it complicates the flow field.*

**F6. Sieges**: confirm they're out of the PoC?

---

## G. Overworld <span class="pill wip">SOON</span>

**G1. Map size and settlement count?** *Recommend: one region, a handful of towns, a dozen villages.
A necromancer terrorising one valley is a stronger story than one conquering a continent.*

**G2. What else is on the map?** Barrows, ruins, old battlefields, graveyards, hermits? *These are the
necromancer's version of loot and probably matter a lot.*

**G3. Route automation? Fast travel? Interception?** *Recommend: automation yes, fast travel no -
interception on the road is too good a source of drama to design away.*

**G4. Day/night, what changes mechanically?** *Recommend: undead faster/stronger at night, living
blind and afraid. Gives the fixed world clock a purpose.*

**G5. How does the player hide?** A horde is not subtle. Does being seen have consequences?

---

## H. Settlements <span class="pill wip">SOON</span>

**H1. Confirm the staged model?** Parasitise early (tribute, graveyard raids, fronts, plague),
convert to a necropolis late, conversion permanent and costly.

**H2. What can you actually *do* at a settlement?** List the verbs: each must be doable in under a
minute with the world live.

**H3. Map-level panel, or walkable scene?** *Recommend: panel for the PoC. Walkable towns are a huge
content cost for a player who can't be seen in public.*

**H4. Which necropolis buildings?** Proposed, capped at four: charnel works, reagent vats,
barrow-vault, obelisk. *Not a build tree.*

**H5. Are there named contacts?** A corrupt undertaker, a cult cell, a desperate lord?

---

## I. Economy <span class="pill wip">SOON</span>

**I1. Confirm the three resources?** Corpses (perishable) · Reagents (upkeep) · Anima (rare). Dread
is a property of a settlement rather than something you carry. *Cut from five on 2026-07-26.*

**I2. How does the player get reagents?** Fronts and black markets are proposed: is that the main
trade minigame?

**I3. How punishing is decay/upkeep?** *This is the dial that decides whether the campaign feels tense
or nagging.*

**I4. Does the player use gold at all?**

**I5. Co-op, shared or separate stockpiles?** *Recommend: separate, with trade. Give players
asymmetric access so trading is worthwhile rather than optional.*

**I6. How does the player learn** that killing a region kills their own income?

---

## J. Campaign & progression <span class="pill idea">LATER</span>

**J1. Cult or clan?** *Recommend: cult. Lieutenants by corruption, cells instead of vassals, no
marriage/diplomacy layer.*

**J2. ~~Where do lieutenants come from?~~** <span class="pill done">ANSWERED</span> Summoned by
ritual at great effort, gated behind holding territory. Players name their own. What does a summoning
ritual actually *cost*, and does it need a specific building?

**J3. Campaign length, and is there an end condition?** Or sandbox?

**J4. Meta-progression across campaigns, or self-contained?**

**J5. Story, or emergent?**

---

## K. Multiplayer <span class="pill wip">SOON</span>

**K0. Turn-based or real-time overworld?** <span class="pill risk">BLOCKING</span> Simultaneous turns
with a march range (TWWH3) versus real-time with activity-derived speed (Bannerlord). Battles are
real-time either way, so this is the overworld layer only. See
[Q2](open-questions.md#q2). *No recommendation: both have a real case.*

**K1. Confirm the `Wait`/`Camp` action** so waiting isn't idling? *(Required, without it, ambushing
is impossible.)*

**K2. What happens when a player disconnects mid-campaign?** Party sleeps? Retires? Territory decays?

**K3. Can a player be another player's lieutenant** rather than a rival necromancer?

**K4. Is 3-4 player support ever wanted?** Nothing should block it, but it changes a netcode
assumption.

**K5. PvP, ever?**

---

## L. Art, audio & feel <span class="pill idea">LATER</span>

**L1. Carry TowerDrop's "no outlines" rule?** *Recommend: yes, outlines get noisy in a crowd.*

**L2. What specifically do you like about XMODE?** Still uncharacterised.

**L3. Audio direction?** Dark comedy is carried by **barks and event text** more than by models -
that's where the character budget goes.

**L4. UI style?** Diegetic (a grimoire, a map table) or clean overlay?

**L5. How much gore?** It's a game about corpses and the tone is dark humour: where's the line?

---

## M. Scope & production <span class="pill risk">BLOCKER</span>

**M1. What does "PoC done" mean to you?** Currently proposed: *two people play together for two
hours: travel a small map, raid a village, fight a battle where the dead get back up, feel the
economy push back, and want to keep playing.* Agree?

**M2. Timeline?** Real deadline, or open-ended?

**M3. Solo, or is there a team?** Changes what's realistic enormously.

**M4. Is this commercial?** Steam intent affects scope, name and art budget.

**M5. What are you willing to cut** if the PoC shows horde control isn't fun?

---

## Related

- [Unit System Design](../tech/unit-system.md): the deconstructed capability list
- [Open Questions](open-questions.md): the short list of blockers
- [Decision Log](decisions.md): what's settled, and why
