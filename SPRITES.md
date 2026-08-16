## v1.1.8 Talent Tree Connector Atlas

Registered atlas:
- `dkt_talent_connectors` → `assets/1x,2x/talent_connectors.png`

Layout:
- 6 columns × 10 rows
- 1x cell size: `96x96`
- 2x cell size: `192x192`
- Rows 0–4: locked/dark magenta connector set
- Rows 5–9: unlocked/gold-cyan connector set

Live UI usage:
- `DeckTalents.connector_sprite_node(key, unlocked, width, height)`
- Deck and Global Talent pages render a three-piece branch rail under the header.
- Prerequisite-complete inbound paths use the unlocked artwork.
- Outbound paths remain locked until the current talent is purchased.

## v0.9.0 Talent Point Award Animation

Registered atlas:
- `dkt_talent_point_award` → `talent_point_award.png`

Layout:
- 8 frames in one horizontal row
- 1x frame size: `222x222`
- 2x frame size: `444x444`

The animation is played manually from frame `{x = 0, y = 0}` through `{x = 7, y = 0}` so it runs exactly once instead of looping.

## v0.8.0 Additional Art Assets

Additional registered atlases:

- `dkt_talent_tree_background` → `assets/1x,2x/talent_tree_background.png`
- `dkt_talent_unlearned` → `assets/1x,2x/talent_unlearned.png`
- `dkt_talent_learned` → `assets/1x,2x/talent_learned.png`

UI usage:
- `DeckTalents.tree_background_node(width, height)`
- `DeckTalents.talent_art_node(owned, width, height)`

# Sprite Sheets

Deck Talents v0.7.0 includes original sprite sheets, registers them as SMODS atlases, and renders them throughout the live UI.

## Included atlases

- `dkt_deck_emblems` → `assets/1x/deck_emblems_sheet.png` and `assets/2x/deck_emblems_sheet.png`
- `dkt_ui_symbols` → `assets/1x/ui_symbols_sheet.png` and `assets/2x/ui_symbols_sheet.png`
- `dkt_talentless_emblem` → `assets/1x/talentless_emblem.png` and `assets/2x/talentless_emblem.png`

## Atlas sizes

- Deck emblems: `640x640` at 1x and `1280x1280` at 2x; 4x4 grid with `160x160` base cells.
- UI symbols: `640x640` at 1x and `1280x1280` at 2x; 4x4 grid with `160x160` base cells.
- Talentless emblem: `256x256` at 1x and `512x512` at 2x.

## Deck emblem cell order

Row 1:
- `(0,0)` Red
- `(1,0)` Blue
- `(2,0)` Yellow
- `(3,0)` Green

Row 2:
- `(0,1)` Black
- `(1,1)` Magic
- `(2,1)` Nebula
- `(3,1)` Ghost

Row 3:
- `(0,2)` Abandoned
- `(1,2)` Checkered
- `(2,2)` Zodiac
- `(3,2)` Painted

Row 4:
- `(0,3)` Anaglyph
- `(1,3)` Plasma
- `(2,3)` Erratic
- `(3,3)` Talentless

## UI symbol cell order

Row 1:
- `(0,0)` Talent point gem
- `(1,0)` Cosmetic
- `(2,0)` Funny
- `(3,0)` Utility

Row 2:
- `(0,1)` Strong
- `(1,1)` Ultimate
- `(2,1)` Legendary
- `(3,1)` Global

Row 3:
- `(0,2)` Owned
- `(1,2)` Locked
- `(2,2)` Ready
- `(3,2)` Toggled On

Row 4:
- `(0,3)` Toggled Off
- `(1,3)` Boss Reward
- `(2,3)` Random / Talentless
- `(3,3)` Main Menu / Talents Button

## Code integration

The mod loads `sprites.lua` during startup and registers the atlases with `SMODS.Atlas`. The sprite maps are exposed through:

- `DeckTalents.sprites.atlases`
- `DeckTalents.sprites.deck_cells`
- `DeckTalents.sprites.ui_cells`
- `DeckTalents.sprite_cell(sheet, key)`

These assets are packaged, registered, and actively rendered by the mod.

## Live UI usage in 0.7.0

The registered sprites are now instantiated with `SMODS.create_sprite` and embedded as `G.UIT.O` nodes. They are used in deck/global headers, talent cards, state badges, stat chips, menu buttons, the Talentless rules panel, and the Talentless deck artwork.
