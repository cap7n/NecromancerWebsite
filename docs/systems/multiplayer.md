# Multiplayer & Time

**Feature 1, and the constraint that shapes everything else.** Per [Pillar 5](../pillars.md), systems
get designed multiplayer-first; anything that only works with a pausable, fast-forwardable world
doesn't survive and has to be redesigned now.

!!! warning "OPEN — nothing here is decided"
    Two blocking questions live on this page: [Q1](../project/open-questions.md#q1) (shape of
    multiplayer) and [Q2](../project/open-questions.md#q2) (time model). They gate the engine choice
    and the entire netcode architecture.

## Why this is hard

Bannerlord's campaign is built on two single-player affordances:

1. **The player controls the clock** — pause to think, fast-forward through travel.
2. **The world stops for a battle** — a 20-minute fight costs zero campaign time.

With two or more players, both break immediately. If Player A pauses to browse a market, does
Player B stop mid-journey? If A fights a 20-minute battle, does B stand still in a field for
20 minutes?

This is not a netcode problem, it's a **design** problem, and it has to be answered before code.

## Q1 — The shape of multiplayer {#shape}

<div class="opt rec" markdown>

### Option A — Small co-op (2–4 players) · **recommended**

One shared campaign. Each player is their own necromancer with their own party, roaming the same
overworld. Host-authoritative (or a lightweight dedicated server). Drop-in/drop-out; when a player
is offline their party sleeps in place or retires to a stronghold.

**For:** achievable as a PoC · matches the natural "me and my mates" appeal of a Bannerlord co-op ·
scales up later if it works.
**Against:** shared progress means the campaign only advances when people are online together.

</div>

<div class="opt" markdown>

### Option B — Larger co-op (up to 8, cult sub-factions)

Same as A but with enough players to want internal structure: one Arch-Lich and lieutenants, or
several allied cults. Needs a real dedicated server and a much more serious approach to time.

**For:** more social, more replayable.
**Against:** every design problem gets harder and the PoC gets further away.

</div>

<div class="opt" markdown>

### Option C — Persistent shard (MMO-lite)

The world runs 24/7 on a server; players log in and out of an ongoing campaign. Mount & Blade's
*Persistent World* mod is the reference.

**For:** genuinely novel, huge long-tail potential.
**Against:** a wholly different (and far larger) engineering project — server costs, persistence,
offline vulnerability, griefing, and an economy that has to survive absentee players. **Not a
prototype.**

</div>

<div class="opt" markdown>

### Option D — Competitive (rival necromancers, campaign has a winner)

4–8 players race/fight to dominate a region; the campaign is a match with an end condition.

**For:** solves the "when does it end" problem and makes PvP a feature rather than a hazard.
**Against:** PvP battles between two hordes is a *third* combat design problem on top of the two
we already have, and balance becomes a permanent tax.

</div>

## The time model — derived from player activity {#time}

!!! success "Decided 2026-07-25"
    **World speed is derived from what both players are doing, not set by hand.** Plus a
    Total-War-style manual toggle that only applies when both players press it.

The problem was that Bannerlord lets you control the clock and stops the world for battles, and
neither survives two players. The answer isn't to give up time control — it's to make the game
*infer* it. Time only accelerates when neither player would miss anything.

### The table

| State | World speed | Why |
|---|---|---|
| **Both players stationary** | **Paused** | Both are in menus, managing, or thinking. Nothing is being missed. |
| **One player moving** | **1× (normal)** | One person is acting; the other shouldn't have the world yanked past them. |
| **Both players moving** | **3× (fast)** | Both are travelling — this is exactly the dead time Bannerlord's fast-forward exists to skip. |
| **Either player in a battle** | **1× (normal)** | Battles run 1:1 with the world clock. Non-negotiable — see below. |

On top of that: a **manual speed toggle in a shared HUD bar**, visible to both players, which applies
only when **both** have it engaged. Reference: Total War: Warhammer's speed-up-turn-resolve button.
This is the escape hatch for cases the derived rule gets wrong.

### Why battles must run 1:1 with the world clock

If the map ran at 3× while a fight ran at 1×, a 10-minute battle would consume 30 minutes of
campaign time and your friend would have crossed the map. Forcing global 1× the moment anyone
engages keeps one clock for everyone, and it produces a **legible reinforcement window**: the time
your friend has to reach your fight is the same time you're experiencing inside it.

It also creates a good social pressure — your friend fighting slows the world for you, which nudges
you toward going to help rather than ignoring it.

**Reinforcement zone** scales with buffs, upgrades, and stronghold proximity, so extending how far
and how long help can arrive becomes a progression axis of its own.

### Three problems this creates

!!! warning "OPEN — waiting must not be the same as idling"
    "Stationary = paused" breaks **waiting as a strategy**. Camping on a road for a caravan, holding
    outside a town, resting — all are things a player *deliberately* does while not moving, and all
    would freeze time. **Needs an explicit `Wait` / `Camp` action that counts as activity** and lets
    the clock run. Without it, ambushing is impossible.

!!! success "Hysteresis — solved structurally (2026-07-26)"
    "Moving" is derived from **intent, not velocity**: a player is moving iff they have an **active
    travel order** (route set / destination reached / `Wait`-`Camp` toggled are the only
    transitions). Discrete events can't flicker, so there's nothing to tune — a short speed ramp
    stays purely for feel. This also makes `Wait`/`Camp` fall out naturally as "an order that
    counts as activity." See the [Decision Log](../project/decisions.md).

!!! danger "Netcode — time scale must be authoritative and tick-aligned"
    A variable time scale is genuinely dangerous in a networked simulation. If each machine derives
    the speed from its own local view, the two will transition on different frames and drift apart.

    **The rule: one machine computes the intended scale and broadcasts "scale becomes X at tick N".**
    Never derive it locally on both. This is cheap to design in now and painful to retrofit — it
    goes into [M4](../project/poc-scope.md) as a hard requirement. The tick→time mapping machinery
    this needs (baseline anchoring + drift correction) already exists in OTR's `NetworkTickManager` —
    see [OTR Carry-Over](../tech/otr-carryover.md).

### Speeds

**Paused / 1× / 3×.** Three states, no more. With derived speed and two players, extra tiers add
confusion without adding control.

### What this preserves that the old proposal gave up

The earlier recommendation was a fixed clock with no acceleration at all, which forced a small map
and slow travel. The derived model keeps fast-forward, so **the map can be bigger and travel can be
longer** — the boring parts compress themselves automatically. Route automation is still worth
having, but it's no longer load-bearing.

Still true regardless: **battle length is a real budget** (a fight should resolve in ~5–10 minutes,
because that's live campaign time for your friend), and **deep menu-diving has to go** — settlement
interaction happens with the world live, so it must be doable in seconds.

!!! tip "The model, in one line"
    **2-player co-op + activity-derived world speed + battles forcing global 1×.** Time control
    without negotiation, and fast-forward that can't skip anything anyone cares about.

## Knock-on effects to work through

- **Map scale.** Less constrained than it was — 3× travel compresses the dead time. Still shouldn't
  be Calradia-sized. **OPEN.**
- **Offline players.** Does their territory decay? Get raided? Freeze? **OPEN.**
- **Session length.** With a live clock, what does a 2-hour session accomplish? Needs an answer
  before the economy is tuned.
- **Reinforcement window.** How long does a friend have to reach your battle? This is a real tuning
  knob and probably the best feature in the model.

## Related

- [Netcode Architecture](../tech/netcode.md) — the engineering side
- [Campaign Overworld](../game/overworld.md) — what the shared map actually is
- [Prototype Scope](../project/poc-scope.md)
