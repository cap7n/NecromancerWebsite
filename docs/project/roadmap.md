# Roadmap & Priorities

Ordering, and the reasoning behind the ordering. The detailed task list is the
[Backlog](backlog.md); the milestone definitions are in [Prototype Scope](poc-scope.md).

## The priority rule

> **Build in order of how much you don't know.**

The overworld, the settlements, and the economy are all *design work on proven foundations*. Horde
control and networked crowds are *unknowns*. Unknowns first, always: every week spent on a campaign
map is a week the core question stays unanswered.

## Phase 0: Decide (days, not weeks)

The four <span class="pill risk">BLOCKING</span> questions:

1. Multiplayer shape ([Q1](open-questions.md#q1))
2. Time model ([Q2](open-questions.md#q2))
3. The unit-count number ([Pillar 6](../pillars.md))
4. The horde verb set ([Q3](open-questions.md#q3))

These cost nothing but conversation, and every one of them is expensive to change later. Then the
engine spike (M0) turns question 3's number into an engine choice.

## Phase 1: Prove the fun (M1 → M2)

The corpse field, then horde control. Ugly, single-player, placeholder everything.

**This phase has a real fail state and we should be honest about it.** If M2 isn't fun, the correct
response is to change the combat design or the project, not to press on and hope the campaign layer
saves it.

## Phase 2: Prove the tech (M3)

Two clients in one battle with hundreds of units and a shared corpse field. Once this works the
project is credible; until it does, everything else is speculative.

## Phase 3: Prove the shape (M4 → M5)

A stub overworld and one economic loop. The question here isn't "is the campaign good": it's
whether the [fixed-clock time model](../systems/multiplayer.md#time) is bearable with two people,
and whether the parasitic economy reads without a tutorial.

## Phase 4: Only then, build the game

Everything in the [Backlog](backlog.md)'s parked list becomes real: settlements with depth, faction
AI, sieges, the full economy, art direction, more players.

## What "PoC finished" means

A definition worth agreeing on up front so the prototype has an end:

> **Two people can play together for two hours: travel a small map, raid a village, fight a battle
> where the dead get back up, and feel the economy pushing back, and want to keep playing.**

If that's true, the concept is proven. Nothing in the PoC has to be pretty, saved, or balanced.

## Related

- [Prototype Scope](poc-scope.md) · [Open Questions](open-questions.md) · [Backlog](backlog.md)
