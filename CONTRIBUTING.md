# Contributing to Deck Talents

Deck Talents benefits heavily from real-run QA because Balatro's UI, scoring contexts, Steamodded hooks, display scaling, and other mods can interact in ways that static checks do not fully reproduce.

## Bug reports

Please use the Bug Report issue template and include as much of the following as possible:

- complete crash log
- Deck Talents version
- Balatro version
- Steamodded and Lovely versions
- platform/OS
- resolution and display mode
- installed mod list
- deck and talent involved
- exact reproduction steps
- screenshot for visual bugs

A reproducible report is much more useful than a description such as "the menu broke."

## Balance testing

Ante Scaling is designed to make scoring talents easier to compare across the run. Useful balance reports include:

- deck and stake
- current Ante
- Ante Scaling state
- active and disabled talents
- important Jokers/Vouchers
- observed Live Power or scoring contribution
- whether the problem is early-game, mid-game, Ante 8, or Endless

Try to distinguish between power coming from the talent itself and power coming from a strong Joker/Voucher interaction.

## Talent Web testing

For web/rendering bugs, include:

- resolution
- fullscreen/windowed/borderless mode
- zoom percentage
- whether the issue changes after Reset View
- whether it affects Deck Webs, Global Web, or both
- selected vs unselected talent screenshots when relevant

## Pull requests

Keep gameplay changes focused. UI fixes should avoid unrelated balance changes, and balance changes should avoid unnecessary UI restructuring.

When changing a talent, update any relevant documentation and test both ownership and enable/disable behavior. Disabled talents must remain owned for prerequisites and mastery while contributing no gameplay power.

When changing permanent progression/config data, preserve migration compatibility wherever practical.
