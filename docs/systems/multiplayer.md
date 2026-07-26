# Multiplayer & Time

**Feature 1, and the constraint that shapes everything else.** Per [Pillar 5](../pillars.md), systems
get designed multiplayer-first; anything that only works with a pausable, fast-forwardable world
doesn't survive and has to be redesigned now.

!!! warning "OPEN, nothing here is decided"
    Two blocking questions live on this page: [Q1](../project/open-questions.md#q1) (shape of
    multiplayer) and [Q2](../project/open-questions.md#q2) (time model). They gate the engine choice
    and the entire netcode architecture.

## Why this is hard

Bannerlord's campaign is built on two single-player affordances:

1. **The player controls the clock**: pause to think, fast-forward through travel.
2. **The world stops for a battle**: a 20-minute fight costs zero campaign time.

With two or more players, both break immediately. If Player A pauses to browse a market, does
Player B stop mid-journey? If A fights a 20-minute battle, does B stand still in a field for
20 minutes?

This is not a netcode problem, it's a **design** problem, and it has to be answered before code.

## Q1: The shape of multiplayer {#shape}

<div class="opt rec" markdown>

### Option A: Small co-op (2-4 players) · **recommended**

One shared campaign. Each player is their own necromancer with their own party, roaming the same
overworld. Host-authoritative (or a lightweight dedicated server). Drop-in/drop-out; when a player
is offline their party sleeps in place or retires to a stronghold.

**For:** achievable as a PoC · matches the natural "me and my mates" appeal of a Bannerlord co-op ·
scales up later if it works.
**Against:** shared progress means the campaign only advances when people are online together.

</div>

<div class="opt" markdown>

### Option B: Larger co-op (up to 8, cult sub-factions)

Same as A but with enough players to want internal structure: one Arch-Lich and lieutenants, or
several allied cults. Needs a real dedicated server and a much more serious approach to time.

**For:** more social, more replayable.
**Against:** every design problem gets harder and the PoC gets further away.

</div>

<div class="opt" markdown>

### Option C: Persistent shard (MMO-lite)

The world runs 24/7 on a server; players log in and out of an ongoing campaign. Mount & Blade's
*Persistent World* mod is the reference.

**For:** genuinely novel, huge long-tail potential.
**Against:** a wholly different (and far larger) engineering project: server costs, persistence,
offline vulnerability, griefing, and an economy that has to survive absentee players. **Not a
prototype.**

</div>

<div class="opt" markdown>

### Option D: Competitive (rival necromancers, campaign has a winner)

4-8 players race/fight to dominate a region; the campaign is a match with an end condition.

**For:** solves the "when does it end" problem and makes PvP a feature rather than a hazard.
**Against:** PvP battles between two hordes is a *third* combat design problem on top of the two
we already have, and balance becomes a permanent tax.

</div>

## The time model: still open {#time}

!!! danger "REOPENED 2026-07-26. This is a blocking question again."
    An activity-derived real-time clock was recorded as decided on 2026-07-25. It is **back open**:
    the project owner is not sure real-time is right, and a **simultaneous-turn model** in the style
    of *Total War: Warhammer 3* is now on the table as an equal candidate.

    The earlier decision stays in the [Decision Log](../project/decisions.md) marked as reopened
    rather than deleted, because the reasoning behind it is still worth having.

### This decision is smaller than it looks

Worth saying first, because it makes the choice less frightening: **battles are real-time either
way.** Total War is turn-based on the campaign map and real-time in the fight, which is exactly the
split this game already has. So this is purely an **overworld-layer** decision. Nothing on
[Horde Combat](../game/combat.md), [Unit System](../tech/unit-system.md) or the battle netcode
changes whichever way it goes.

<div class="opt" markdown>

### Option A: simultaneous turns (Total War: Warhammer 3)

Both players plan and move in the same turn. Each party has a **march range**, a movement budget it
can spend. The turn resolves when both players commit, or a timer expires.

**For, and the first point is a big one:**

- **It deletes the multiplayer time problem outright.** No shared clock, no speed negotiation, no
  "what is my friend doing while I fight", no [tick-aligned timescale](../tech/netcode.md) risk. An
  entire subsystem we designed stops needing to exist.
- **Overworld netcode becomes trivial.** Send orders, resolve, sync at the turn boundary. No
  continuous replication of a live world.
- **Battle length stops mattering.** A 20-minute fight costs zero campaign time, so the 5-10 minute
  budget we imposed on combat can relax.
- **Save/load is trivial**, since state is clean at turn boundaries.
- Thinking time is free. You can plan a poisoning campaign without the world running on.

**Against:**

- **Feature 2 says "Bannerlord-like overworld", and Bannerlord is real-time.** This is the option
  that most changes what was asked for.
- **Turn-waiting friction is real and well documented** in Total War co-op campaigns. You wait on
  your partner constantly.
- **Day/night as a mechanic weakens**, since turns abstract time. The "undead are stronger at night"
  idea gets much less interesting.
- Reinforcement becomes a **range check at battle start** rather than a live ride to your friend's
  fight. Not lost, but far less dramatic.

</div>

<div class="opt" markdown>

### Option B: real-time, activity-derived speed (Bannerlord)

The previously-recorded model. World speed is inferred from what both players are doing: both
stationary pauses, one moving runs 1x, both moving runs 3x, and either player in a battle forces 1x.
Plus a shared HUD toggle that only applies when both players engage it.

**For:**

- **Matches Feature 2 literally**, and preserves the Bannerlord texture the pitch asked for.
- **The marquee co-op moment survives**: you see your friend's fight on the map, you ride for it, you
  arrive mid-battle as reinforcements.
- **Day/night is real gameplay**, which gives the world clock a purpose.
- No turn-waiting. Nobody is ever blocked on their partner clicking End Turn.
- Interception on the road stays possible, which is good drama for a game about being hunted.

**Against:**

- Requires the whole derived-speed system, a `Wait`/`Camp` action so waiting is not idling, and a
  **tick-aligned authoritative timescale** or the two machines drift apart.
- **Battle length becomes a hard budget**, because a long fight is live campaign time for your
  partner.
- Settlement interaction must work with the world live, so no deep menu-diving.
- Continuous overworld replication, which is more netcode than Option A by a wide margin.

</div>

### What each does to the campaign arc

Worth weighing, since [the arc](progression.md) is now the spine of the game.

**Acts 1 and 2 are sneaking, poisoning and scavenging.** Real-time suits the *sneaking* (avoiding
patrols, timing a move) while turns suit the *poisoning* (act, end turn, come back later for the
bodies). Neither is obviously right, and this is probably the sharpest test available: whichever
model makes "one skeleton hiding in a cellar" more tense is the one to take.

### Nothing has been chosen

Both options are live. The [netcode](../tech/netcode.md) and
[Campaign Overworld](../game/overworld.md) pages currently assume Option B and will need revising if
Option A is chosen. That revision is contained, because it is the overworld layer only.

## Knock-on effects to work through

- **Map scale.** Less constrained than it was: 3× travel compresses the dead time. Still shouldn't
  be Calradia-sized. **OPEN.**
- **Offline players.** Does their territory decay? Get raided? Freeze? **OPEN.**
- **Session length.** With a live clock, what does a 2-hour session accomplish? Needs an answer
  before the economy is tuned.
- **Reinforcement window.** How long does a friend have to reach your battle? This is a real tuning
  knob and probably the best feature in the model.

## Related

- [Netcode Architecture](../tech/netcode.md): the engineering side
- [Campaign Overworld](../game/overworld.md): what the shared map actually is
- [Prototype Scope](../project/poc-scope.md)
