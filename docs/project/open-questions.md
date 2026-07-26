# Open Questions

The master list of what has to be decided. **This page drives the design conversation.** When
something gets settled it moves out of here and into the [Decision Log](decisions.md) *with the why*.

Ordered by how much downstream work each one blocks.

Legend: <span class="pill risk">BLOCKING</span> decide before any code
<span class="pill wip">SOON</span> decide during the PoC
<span class="pill idea">LATER</span> safe to defer
<span class="pill done">DECIDED</span> settled: see the [Decision Log](decisions.md)

---

## Q1. How many players, and what is multiplayer *shaped* like? <span class="pill done">DECIDED</span> {#q1}

**2-player co-op**, each player their own necromancer with their own party in a shared campaign,
**split-authority** networking so both machines simulate real work. See the
[Decision Log](decisions.md) and [Netcode](../tech/netcode.md).

*Still open under this:* whether the design should later stretch to 3-4 players. Nothing should be
built that makes it impossible, but 2 is the target.

## Q2. Turn-based or real-time overworld? <span class="pill risk">BLOCKING</span> {#q2}

**Reopened 2026-07-26.** An activity-derived real-time clock was recorded as decided on 2026-07-25;
it is open again, with **simultaneous turns** (Total War: Warhammer 3 style, with a march range) now
an equal candidate.

**Smaller than it looks:** battles are real-time either way, so this is an **overworld-layer**
decision only. Nothing in combat, the unit system, or battle netcode changes.

**Option A, simultaneous turns.** Deletes the multiplayer time problem outright: no shared clock, no
speed negotiation, no tick-aligned timescale risk, and trivial overworld netcode. Battle length stops
mattering. Costs the Bannerlord texture Feature 2 asked for, weakens day/night, adds turn-waiting
friction, and reduces reinforcement to a range check.

**Option B, real-time with derived speed.** Matches Feature 2, keeps the ride-to-your-friend's-battle
moment and day/night as real mechanics, nobody waits on anybody. Costs a whole derived-speed
subsystem, a hard battle-length budget, and continuous overworld replication.

**Sharpest test available:** whichever makes [Act 1](../systems/progression.md) more tense, one
skeleton hiding in a cellar, is probably the right answer.

-> [Multiplayer & Time](../systems/multiplayer.md#time)

## Q3. What is the actual horde-control verb set? <span class="pill risk">BLOCKING</span> {#q3}

**Working shape as of 2026-07-26: aura plus detachments.** The main body lives in the player's aura
and takes no orders at all (you steer it by walking). Portions can be **split off as detachments**
and given tactical orders, but a detachment needs an **anchor** to hold its leash: a lieutenant, a
planted standard, or a decaying tether that slips toward feral. Anchors owned = detachments
runnable. **Line-drawing** positions either tier: drag a line, they distribute across it X deep,
and hold it loosely.

**Still open under this:** how the player switches between commanding the main body and a
detachment without a selection UI (the real risk), the detachment cap (3-4 suggested by both the
anchor fiction and the flow-field budget), and which of the four original verbs survive as
detachment orders.

-> [Horde Combat](../game/combat.md#aura-control)

## Q4. Engine <span class="pill done">DECIDED</span> {#q4}

**Godot 4.** Team comfort and TowerDrop carry-over, and the
[no-physics agent decision](decisions.md) removes the weakness that would have ruled it out.
Conditional on declining three Godot conveniences, no node per unit, no `NavigationAgent3D`, no
high-level multiplayer. See [Engine & Tooling](../tech/engine.md).

*Live risk:* GDScript may not hold ~800 agents. Mitigation is a contained GDExtension port of the
hot loop: design the agent sim as a swappable module from day one.

## Q5. Is the player a lone necromancer, or a clan/faction lord? <span class="pill wip">SOON</span> {#q5}

Bannerlord's whole strategic layer assumes clans, vassals, fiefs, and marriage. A necromancer is
thematically a *loner or a cult leader*. If there's no clan there's no diplomacy layer, and the
overworld game becomes much smaller: possibly correctly so.

**Recommendation: cult, not clan.** Living lieutenants and death knights instead of family;
recruitment by corruption instead of marriage.

→ [The Necromancer](../game/necromancer.md)

## Q6. Do you *conquer* settlements or *parasitise* them? <span class="pill wip">SOON</span> {#q6}

Do you take towns and hold them like Bannerlord, or do you keep them alive and milk them, converting
only a few into necropolises?

**Recommendation: both, staged.** Parasitise early, convert late, and make conversion **permanent
and costly** so it's a real strategic decision rather than a checkbox.

→ [Settlements](../game/settlements.md)

## Q7. What is the economy's actual resource list? <span class="pill wip">SOON</span> {#q7}

"Deeper economy" needs to become a specific, small set of resources with real conversions between
them, or it becomes twelve bars nobody reads.

**Recommendation: keep it to five -** Corpses, Bone/Material, Reagents, Anima, Dread, **layered on
top of an ordinary living economy** the player interacts with only indirectly.

→ [Economy](../systems/economy.md)

## Q8. Does the player fight personally, Bannerlord-style? <span class="pill done">DECIDED</span> {#q8}

**No melee. The necromancer is a caster**: spells and commands only, but has a body on the field
and is killable, with the control aura anchored to it. The single largest scope cut available. See
[The Necromancer](../game/necromancer.md).

## Q9. Who are the enemies, and is there a living faction AI? <span class="pill wip">SOON</span> {#q9}

Bannerlord's world is alive because kingdoms fight each other without you. A world where everyone
only reacts to the necromancer is much cheaper and much flatter.

→ [Campaign Overworld](../game/overworld.md)

## Q10. PvP? <span class="pill idea">LATER</span> {#q10}

Can players fight each other? If the campaign is co-op, probably not for the PoC, but it changes
netcode requirements (cheat resistance, authority) if it's ever planned.

## Q11. Art direction & tone <span class="pill done">DECIDED</span> {#q11}

**Style:** stylized, **low-detail models**: TABS and Valheim cited for *model detail level*, not
for their full tonal package. Colour and atmosphere do the work. Not Bannerlord realism.

**Tone:** **lighthearted with an undertone of suffering: dark humour.** Not grim-dark serious.
This is what makes the [economy](../systems/economy.md) playable rather than oppressive.

**Consequence:** comedy is carried by writing, audio, and animation personality, not model detail.
That's where the character budget goes. See [Art Direction](../tech/art-direction.md).

*Still open:* whether to carry TowerDrop's no-outlines rule. And *XMODE* was cited as a reference
and still hasn't been characterised.

## Q14. How many units actually *read* as a horde? <span class="pill wip">SOON</span> {#q14}

A perceptual question, not a technical one, and it may make the technical one moot. 30 reads as a
squad; 100 starts to read as a crowd. If **300 already reads as an overwhelming horde from a
third-person camera**, then 800 is budget we could spend on corpses, effects, or a bigger battle
count instead.

The [M0 spike](../tech/performance.md) has a third-person camera (`C`) and count presets on `1`..`6`
(30 / 100 / 300 / 800 / 1600 / 3200) with **spawn density held constant across presets**, so the
comparison isolates count rather than sparseness. Judge it from eye level next to the 1.8 m
stand-in body: an overhead strategy view answers a different question.

**Answer this before optimising for more agents.** More may not be better.

## Q12. Project name <span class="pill idea">LATER</span> {#q12}

"Necromancer" is a working title and is unsearchable, ungoogleable, and probably untrademarkable.

## Q13. Scope reality check <span class="pill risk">ONGOING</span> {#q13}

Bannerlord took ~8 years and a large studio, and this pitch is Bannerlord **plus multiplayer** plus
a novel combat system. That is not a criticism of the idea: it's a reason to be ruthless about what
the PoC actually contains and what "done" means for it.

→ [Prototype Scope](poc-scope.md)
