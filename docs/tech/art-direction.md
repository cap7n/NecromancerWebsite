# Art Direction

!!! success "Decided 2026-07-25"
    **Style:** low-detail models, colour and atmosphere doing the work. *Totally Accurate Battle
    Simulator* and *Valheim* are cited **for model detail level** — not for their mechanics (TABS is
    a physics game; we are explicitly [not](../project/decisions.md)) and not for their full tonal
    package. Explicitly not Bannerlord's realism.

    **Tone:** **lighthearted with an undertone of suffering — dark humour.** Not grim-dark serious.

## Why this is the right call beyond cost

It would be easy to read "low-poly" as a budget compromise. It isn't — it's the correct answer for a
crowd game:

- **Silhouette reads at 800 units.** That's the hard problem ([Pillar 7](../pillars.md)), and flat
  shading with strong shapes solves it where realistic detail actively works against it.
- **Emissive allegiance coding is trivial.** Your undead, your friend's undead, feral, and enemy
  living all need to be distinguishable in a pile of near-identical bodies. In flat-shaded art
  that's an emissive colour; in realistic art it's a nightmare.
- **Valheim proves the ceiling.** Fog, lighting, and a committed palette make low-poly read as
  premium rather than cheap. That pairs directly with night being real gameplay
  ([Overworld](../game/overworld.md)) — atmosphere is doing double duty as mechanic and as look.
- **No texture memory, no PBR authoring**, one mesh reused hundreds of times. Which is what the
  [performance budget](performance.md) needs.

## Hard constraints the art must satisfy

1. **Allegiance and state readable in a crowd.** Emissive colour per faction/state. This is the
   **first art task**, and it's functional, not decorative — prototype it in M2 with placeholder
   shapes.
2. **Corpses read as raw material, not debris.** The player should look at a battlefield and see
   *stock*. Piles need to be legible and appealing, since they're the core economy made visible.
3. **Readable at night.** Serious constraint on palette and contrast. Test it early rather than
   discovering it late.
4. **Cheap enough to have hundreds of, in several damage states.**
5. **Compatible with vertex-animation-texture baking** — see [Performance](performance.md). Rigs stay
   simple; no per-unit skeletal animation at runtime.

## Deaths without physics

Since [units aren't rigid bodies](../project/decisions.md), there are no ragdolls. Death is a
**canned tip-over** into a corpse record. Flat-shaded art is forgiving here — a handful of hand-
authored fall animations, chosen at random with a random rotation, will read fine at this scale and
distance. Budget 3–4 death clips, not a physics system.

Impulses still apply to living agents, so **spells still scatter a crowd** — that motion is the
spectacle, not the deaths.

## Tone: where the character budget actually goes

Dark humour is what makes the [economy](../systems/economy.md) — a system for literally farming
human suffering — *playable* rather than oppressive. Played straight it's grim in a way that fights
against a co-op game you play with a mate; played for gallows humour, it's the joke. The Dungeon
Keeper / Overlord register.

**The consequence to act on: comedy is carried by writing, audio, and animation personality, not by
model detail.** All three are cheap, and all three are fully compatible with low-detail models. So:

- **Unit barks and event text** are the primary comedy vehicle. Budget for a lot of them.
- **Animation personality** over model fidelity — a skeleton that shrugs before shambling off reads
  funnier than a higher-poly skeleton that doesn't.
- **Death animations** get to be funny. This helps enormously, since [no physics](../project/decisions.md)
  means hand-authored deaths anyway — make the 3–4 clips characterful rather than neutral.
- **Don't let UI and system naming go po-faced.** "Will" is a placeholder and probably wants a
  funnier name.

The tension to watch: the strategic layer still needs to carry real weight, or the economy's central
tension stops mattering. Humour in the *texture*, seriousness in the *stakes*.

## Still open

- **XMODE** was cited as a reference and isn't captured here — its specific qualities should be added
  by someone who knows the game.
- **Outlines.** TowerDrop's direction is explicitly *no cartoon outlines*. Carrying that forward is
  probably right (outlines get noisy fast in a crowd) but should be a deliberate choice.

## Related

- [Pillars](../pillars.md) · [Horde Scale & Performance](performance.md) · [Decision Log](../project/decisions.md)
