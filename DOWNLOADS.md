# Downloads

## Packaged mod

For normal installation, use a packaged Deck Talents ZIP that contains the complete `DeckTalents` folder and its `assets/1x` and `assets/2x` artwork.

### GitHub Releases

When a release is published here, download the `DeckTalents_vX.X.X.zip` asset from the repository's **Releases** section rather than GitHub's automatically generated source-code archive.

### Nexus Mods

Deck Talents is also published on Nexus Mods:

https://www.nexusmods.com/balatro/mods/896

## Why use the packaged ZIP?

The mod contains a large custom art library. The packaged build keeps the Lua code, manifest, documentation, and both 1x/2x sprite atlases together in the exact folder structure Steamodded expects.

## Install layout

```text
%AppData%/Balatro/Mods/DeckTalents/
├── DeckTalents.json
├── main.lua
├── talents.lua
├── global_talents.lua
├── deck_expansion.lua
├── talent_map.lua
├── talent_shop.lua
├── talentless_deck.lua
├── sprites.lua
└── assets/
    ├── 1x/
    └── 2x/
```

Do not nest a new `DeckTalents` folder inside an old `DeckTalents` folder when updating.
