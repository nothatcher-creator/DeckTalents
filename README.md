# Deck Talents

Permanent progression, full-screen talent webs, and a Talent Point economy for Balatro.

**Current version:** 1.6.3 — Full-Canvas Web  
**Requires:** Balatro 1.0.1o, Lovely, and Steamodded `1.0.0~BETA-1814a` or newer

Deck Talents adds a permanent progression system to every vanilla Balatro deck. Each deck has its own 15-talent tree, alongside 30 Global Talents, a shared Talent Point bank, a Talent Shop, a Talentless Deck, mastery tracking, Live Power readouts, and optional Ante-based scoring scaling.

## Highlights

- 15 talent trees for the 15 vanilla decks
- 225 deck-specific talents
- 30 Global Talents
- 255 permanent talents total
- One Legendary capstone per deck
- Full-screen mouse-driven talent webs
- Connector-sprite pathways showing progression
- Mouse-wheel zoom and click-drag panning
- Persistent Talent Points
- Boss Blind and Blind-skip Talent Point rewards
- Talent Shop with vanilla and custom rewards
- Per-talent enable/disable controls
- Optional Ante-scaled Chips, +Mult, and XMult
- Live Power readouts
- Deck mastery tracking
- Talentless Deck for talent-free runs that still earn Talent Points
- Custom art, icons, frames, backgrounds, token art, and reward animation

## Talent Points

Talent Points are shared across all decks.

- Defeat a Boss Blind: **+1 Talent Point**
- Skip a Small or Big Blind: **+1 Talent Point**
- Spend points permanently on Deck or Global Talents
- Spend points in the Talent Shop for special rewards

Progress is stored through Steamodded and is designed to survive normal mod updates.

## Adaptive Power

Deck Talents can scale scoring bonuses with the current Ante. The previous fixed scoring values are treated as their approximate **Ante 8 reference strength**.

- Ante 1: roughly 12.5% of reference scoring power
- Ante 4: roughly 50%
- Ante 8: 100%
- Endless: continues scaling upward

XMult scales only the bonus above X1, so early Antes do not turn positive XMult talents into penalties.

Ante Scaling can be toggled from the mod configuration. Individual unlocked talents can also be enabled or disabled without losing ownership, prerequisites, or mastery progress.

## Talent Web Controls

The current talent browser is a full-screen web.

- **Mouse wheel:** zoom
- **Left-click + drag:** pan
- **Click a talent:** open its detail card
- **Unlock:** purchase an available talent
- **Enable / Disable:** toggle an owned talent without unlearning it
- **Reset View:** restore the default camera position
- **Legacy:** open the older page-based talent interface as a fallback

## Talent Shop

The Talent Shop appears during normal shops and uses the same permanent Talent Point currency.

Custom rewards include:

- Royal Forge
- Numerical Paradox
- Talent Battery
- Vault Expansion
- Bossbreaker Sigil

The shop also includes several vanilla-style premium rewards. See [`TALENT_SHOP.md`](TALENT_SHOP.md) for the full catalog.

## Talentless Deck

The Talentless Deck randomly copies one vanilla deck's base effect when a run begins while suppressing Deck and Global talent power for that run.

It still earns Talent Points from Boss Blinds and Blind skips, and the Talent Shop remains usable.

## Installation

1. Install Lovely.
2. Install Steamodded `1.0.0~BETA-1814a` or newer.
3. Download or clone this repository.
4. Make sure the folder containing `DeckTalents.json` is named `DeckTalents`.
5. Put that folder in your Balatro Mods directory.
6. Launch Balatro and confirm **Deck Talents** appears in the Mods menu.

Typical Windows path:

```text
%AppData%/Balatro/Mods/DeckTalents/
```

The final layout should look like:

```text
Balatro/
└── Mods/
    └── DeckTalents/
        ├── DeckTalents.json
        ├── main.lua
        ├── talents.lua
        ├── global_talents.lua
        ├── talent_map.lua
        ├── talent_shop.lua
        └── assets/
```

## Updating

Replace the old Deck Talents files with the newer version while keeping the folder name `DeckTalents`. Existing Talent Points, purchased talents, mastery state, cosmetics, and activation settings are migrated by the mod configuration system.

## Compatibility / Testing

The mod targets:

- Balatro `1.0.1o-FULL`
- Steamodded `1.0.0~BETA-1814a`
- LÖVE `11.5.0`
- Lovely `0.9.0`

Deck Talents is under active development. Static and mocked runtime checks are used during development, but live-game UI behavior can still differ across resolutions, platforms, and mod combinations. Detailed bug reports with a crash log, resolution, window mode, and screenshots are especially useful.

## Documentation

- [`CHANGELOG.md`](CHANGELOG.md) — version history
- [`TALENT_SHOP.md`](TALENT_SHOP.md) — Talent Shop catalog and rules
- [`GLOBAL_TALENTS.md`](GLOBAL_TALENTS.md) — Global Talent notes
- [`LEGENDARY_TALENTS.md`](LEGENDARY_TALENTS.md) — Legendary talent details
- [`TALENTLESS_DECK.md`](TALENTLESS_DECK.md) — Talentless Deck behavior
- [`SPRITES.md`](SPRITES.md) — sprite/atlas notes
- [`TESTING_CHECKLIST.md`](TESTING_CHECKLIST.md) — QA checklist

## Credits

**Deck Talents** by Brad & OpenAI.

Special thanks to the testers and community members who have repeatedly put the mod through real runs and helped identify UI, balance, progression, and compatibility issues.

Balatro is created by LocalThunk. Deck Talents is an unofficial fan-made mod and is not affiliated with LocalThunk, Playstack, or the Steamodded project.
