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

## Q2. Turn-based or real-time overworld? <span class="pill done">DECIDED</span> {#q2}

**Closed again 2026-07-27: real-time, Bannerlord-ish, not turn-based.** The activity-derived speed
model stands. New wrinkle recorded with it: **overworld information is invisible**; the state of the
world reaches you via **messengers**, so news is stale, partial, and interceptable.

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

## Q5. Lone necromancer or clan? <span class="pill done">DECIDED</span> {#q5}

**A clan / faction / cult of necromancers**, not a loner. Fits the anchor model: a player can even be
another player's lieutenant (K3, answered yes).

## Q6. Conquer or parasitise settlements? <span class="pill done">DECIDED</span> {#q6}

**Both. More choices.** The staged model (parasitise early, convert late) remains the working shape.

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

## Q9. Living faction AI? <span class="pill done">DECIDED</span> {#q9}

**Several rival factions, simulated but not deep.**

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
