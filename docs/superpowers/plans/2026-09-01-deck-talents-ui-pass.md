# Deck Talents Hybrid UI & Release Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the Talent Universe into the approved Balatro-shell/cosmic-RPG hybrid interface without reintroducing cumulative constellation lag, then ship the combined balance + UI work as Deck Talents v1.8.0.

**Architecture:** Keep the v1.7.13 direct LÖVE HUD for all side controls and inspector content, and keep the existing sparse constellation camera/model for stars. Add pure UI theme/inspector-formatting modules for testable geometry and presentation; Figma is a visual reference only, not a runtime dependency and not a source of heavy exported UI trees.

**Tech Stack:** Lua / LÖVE 11.5, Steamodded, Balatro 1.0.1o-FULL [M], Figma MCP/Plugin API, `texluac -p`, Luacheck 1.2.0, Python static contract tests, Git/GitHub.

**Spec:** `docs/superpowers/specs/2026-09-01-deck-talents-ui-balance-design.md`

**Depends on:** `docs/superpowers/plans/2026-09-01-deck-talents-balance-runtime.md` completed and green.

## Global Constraints

- Keep the v1.7.13 direct HUD architecture; ordinary star selection must not rebuild the constellation overlay.
- Keep zoom and pan in-place; camera replacement is allowed only for the existing zoom/fallback path.
- Do not create new nested UIBox trees for the left rail, top header, right inspector, Help, or Settings.
- Direct HUD hitboxes stay plain Lua tables.
- Decorative animation is bounded and may not allocate persistent objects every frame.
- Inspector cache invalidates only on selection, profile, Ante, relevant run-state change, or Help/Settings mode change.
- Android landscape remains the target; geometry scales from a 1920×1080 reference using the existing direct-HUD scaling approach.
- Existing constellation talent routes/positions are not rearranged.
- Existing profile progression/reset behavior remains unchanged.
- Figma settles visual proportions/tokens; runtime visuals are recreated with lightweight LÖVE drawing and existing sprite assets.
- Final release version is `1.8.0`.

---

### Task 1: Build the Figma visual reference

**Files:**
- Create: `docs/ui/2026-09-01-hybrid-ui-reference.md`

**Interfaces:**
- Consumes: approved design spec.
- Produces: Figma Design file `Deck Talents v1.8 — Hybrid Talent Universe` plus exact runtime geometry/tokens.

- [ ] **Step 1: Load required Figma skills before any write**

Use `figma-use` and `figma-generate-design` before Figma write calls.

- [ ] **Step 2: Create one 1920×1080 primary frame**

Frame: `Talent Universe / Hybrid / 1920x1080`.

```text
Outer margin: 24 px
Top header: 86 px high
Left rail: 270 px wide
Right inspector: 430 px wide
Bottom hint row: 34 px high
Gap between side rails and constellation: 18 px
Panel corner radius: 14 px
Primary button corner radius: 10 px
```

- [ ] **Step 3: Build the reference screen**

Include the top header, left rail, center constellation, right inspector, bottom hint row, purchased/reachable route contrast, and one each of locked/unlockable/purchased/selected/legendary nodes.

- [ ] **Step 4: Validate the same geometry with Red and Plasma accents**

Do not create 15 deck-specific screens; use Red and Plasma as warm/cool stress tests.

- [ ] **Step 5: Capture a screenshot and write the code reference note**

Record these runtime tokens exactly:

```text
Panel background alpha: 0.88
Inner panel alpha: 0.78
Locked node alpha: 0.28
Unlockable node alpha: 0.72
Purchased node alpha: 1.00
Selected outline: white, 3 px at 1x
Legendary outline/glow: gold, 4 px at 1x
Background decoration: static/bounded; no object creation per frame
```

- [ ] **Step 6: Commit**

```bash
git add docs/ui/2026-09-01-hybrid-ui-reference.md
git commit -m "docs: lock hybrid talent universe visual reference"
```

---

### Task 2: Add pure UI theme and inspector-formatting modules

**Files:**
- Create: `ui_theme.lua`
- Create: `ui_inspector.lua`
- Create: `tests/test_ui_theme.lua`
- Create: `tests/test_ui_inspector.lua`
- Modify: `talent_map.lua` loader area before the final direct-HUD override

**Interfaces:**
- `DT.ui_theme.layout(screen_w, screen_h) -> geometry`
- `DT.ui_theme.mix(colour, strength, alpha) -> colour`
- `DT.ui_theme.node_style(state, legendary, accent) -> style`
- `DT.ui_inspector.format(args) -> model`
- Inspector args: `title`, `kind`, `description`, `trigger`, `scaling`, `current`, `ante8`, `status`, `cost`, `requirements`, `accent`.

- [ ] **Step 1: Write failing geometry/state tests**

```lua
local T = dofile('ui_theme.lua')
local g = T.layout(1920,1080)
assert(g.header_h == 86 and g.left_w == 270 and g.right_w == 430 and g.bottom_h == 34)
assert(g.center_w == 1136)
local locked = T.node_style('locked',false,{1,0,0,1})
local ready = T.node_style('unlockable',false,{1,0,0,1})
local owned = T.node_style('purchased',false,{1,0,0,1})
local legendary = T.node_style('purchased',true,{1,0,0,1})
assert(locked.alpha == 0.28 and ready.alpha == 0.72 and owned.alpha == 1.00)
assert(legendary.outline_width == 4)
print('ui theme OK')
```

- [ ] **Step 2: Write failing inspector test**

```lua
local I = dofile('ui_inspector.lua')
local m = I.format({
  title='Anger Management',kind='Strong',description='+2 Mult per Ante.',
  trigger='After first discard this Blind',scaling='+2 Mult / Ante',
  current='+10 Mult at Ante 5',ante8='+16 Mult',status='OWNED - ACTIVE',
  cost='3 TP',requirements={'Emergency Discard'},accent={1,0,0,1},
})
assert(m.sections[1].label == 'EFFECT')
assert(m.sections[2].label == 'TRIGGER')
assert(m.sections[3].label == 'SCALING')
assert(m.sections[4].label == 'CURRENT')
assert(m.sections[5].label == 'ANTE 8')
print('ui inspector OK')
```

- [ ] **Step 3: Run and confirm missing-module failures**

```bash
texlua tests/test_ui_theme.lua
texlua tests/test_ui_inspector.lua
```

- [ ] **Step 4: Implement the modules**

`ui_theme.layout()` uses `s = min(screen_w/1920, screen_h/1080)` and returns scaled pixel rectangles for `header`, `left`, `center`, `right`, and `bottom`, plus the scalar and the exact reference widths/heights. `node_style()` maps only `locked`, `unlockable`, `purchased`, `selected`; `legendary` is a separate boolean that overrides outline/glow strength.

`ui_inspector.format()` creates ordered sections in the order Effect, Trigger, Scaling, Current, Ante 8, Requirements. It omits only nil/empty optional sections and touches no Balatro/LÖVE globals.

- [ ] **Step 5: Load once in `talent_map.lua`**

```lua
local ui_theme_chunk = SMODS.load_file('ui_theme.lua')
if not ui_theme_chunk then error('Deck Talents could not load ui_theme.lua') end
DT.ui_theme = ui_theme_chunk()
local ui_inspector_chunk = SMODS.load_file('ui_inspector.lua')
if not ui_inspector_chunk then error('Deck Talents could not load ui_inspector.lua') end
DT.ui_inspector = ui_inspector_chunk()
```

- [ ] **Step 6: Test, parse, commit**

```bash
texlua tests/test_ui_theme.lua
texlua tests/test_ui_inspector.lua
texluac -p ui_theme.lua
texluac -p ui_inspector.lua
texluac -p talent_map.lua
git add ui_theme.lua ui_inspector.lua talent_map.lua tests/test_ui_theme.lua tests/test_ui_inspector.lua
git commit -m "test: add pure hybrid UI models"
```

---

### Task 3: Replace the direct HUD shell with the hybrid header/rails

**Files:**
- Modify: final direct-HUD block in `talent_map.lua` beginning at the v1.7.13 marker
- Create: `tests/test_direct_hud_contract.py`

**Interfaces:**
- Consumes: `DT.ui_theme.layout()`.
- Produces: `draw_header180`, `draw_left180`, `draw_bottom_hints180` and bounded plain-table hitboxes.

- [ ] **Step 1: Write the failing contract test**

```python
from pathlib import Path
src = Path('talent_map.lua').read_text(encoding='utf-8')
marker = 'Deck Talents v1.7.13 DIRECT HUD'
assert marker in src
final = src[src.index(marker):]
assert 'draw_header180' in final
assert 'draw_bottom_hints180' in final
assert 'UIBox({' not in final and 'UIBox{' not in final
assert 'G.UIT.' not in final
print('direct HUD contract OK')
```

- [ ] **Step 2: Run and confirm missing-function failure**

```bash
python tests/test_direct_hud_contract.py
```

- [ ] **Step 3: Implement the top header**

Left-to-right: deck/global label, Balatro Profile number, TP total, `HELP`, `SETTINGS`, `RESET`, `BACK`. `RESET` calls existing `G.FUNCS.dt_reset_profile_progression`; do not duplicate reset logic.

- [ ] **Step 4: Simplify the left rail**

Keep `CENTER GLOBAL`, Ante Scaling, zoom `- / % / +`, `OVERVIEW`, and a five-state legend. Move `LIVE POWER` and `LEGACY` into Help/Settings mode.

- [ ] **Step 5: Add the bottom hint row**

```text
DRAG TO PAN    •    WHEEL / +/- TO ZOOM    •    TAP A STAR TO INSPECT
```

- [ ] **Step 6: Preserve bounded hitboxes**

Reset/reuse `H.buttons` each draw. `H.button_count` equals visible controls for the current frame; no history list is retained.

- [ ] **Step 7: Test, parse, commit**

```bash
python tests/test_direct_hud_contract.py
texluac -p talent_map.lua
git add talent_map.lua tests/test_direct_hud_contract.py
git commit -m "feat: add hybrid direct HUD shell"
```

---

### Task 4: Build structured Talent/Help/Settings inspector modes

**Files:**
- Modify: final `inspector_model1713`, empty-inspector and inspector draw functions in `talent_map.lua`
- Modify: `main.lua` only if a stable trigger/scaling accessor is required
- Modify: `tests/test_ui_inspector.lua`

**Interfaces:**
- `H.mode` exact values: `talent`, `help`, `settings`; default `talent`.
- Selecting a star always sets `H.mode='talent'`.

- [ ] **Step 1: Extend inspector tests**

Assert that a non-Ante talent omits `ANTE 8`, a cosmetic can omit `CURRENT`, and a locked talent still includes state/cost/requirements.

- [ ] **Step 2: Build a cache key from actual dependencies**

Include selected scope/deck/id, profile key, current Ante, TP, owned/enabled state, `H.mode`, and compact run-state values used by the selected talent. Do not key on frame count/time.

- [ ] **Step 3: Render Talent mode in this order**

```text
Talent Name / Kind
State + Cost
EFFECT
TRIGGER
SCALING
CURRENT
ANTE 8 (when applicable)
REQUIRES
Action button
```

- [ ] **Step 4: Implement Help mode in-place**

Explain node states, pan/zoom/tap, Ante Scaling, Live Power, and Legacy. Use direct text/buttons only.

- [ ] **Step 5: Implement Settings mode in-place**

Show active profile, Ante Scaling toggle, `LIVE POWER`, `LEGACY`, and `RESET CURRENT PROFILE`. Reset retains the existing two-tap/5-second confirmation callback.

- [ ] **Step 6: Test, parse, commit**

```bash
texlua tests/test_ui_inspector.lua
python tests/test_direct_hud_contract.py
texluac -p talent_map.lua
texluac -p main.lua
git add talent_map.lua main.lua tests/test_ui_inspector.lua
git commit -m "feat: add structured direct-draw inspector"
```

---

### Task 5: Polish node states, routes, and center atmosphere

**Files:**
- Modify: node style helpers near `talent_map.lua:136-175`
- Modify: constellation node/route code around `talent_map.lua:2738-3140` and the final constellation-node override
- Modify: final `love.draw` wrapper in the direct-HUD block
- Modify: `tests/test_ui_theme.lua`
- Modify: `tests/test_direct_hud_contract.py`

**Interfaces:**
- Consumes: `DT.ui_theme.node_style()`.
- Produces: five readable visual states without route-coordinate changes.

- [ ] **Step 1: Extend node-state tests**

Test `locked`, `unlockable`, `purchased`, `selected` plus `legendary=true`. Selected uses white outline; legendary uses gold glow and a 4px reference outline.

- [ ] **Step 2: Route final node rendering through theme state**

```lua
local state = selected and 'selected' or (owned and 'purchased' or (ready and 'unlockable' or 'locked'))
local style = DT.ui_theme.node_style(state, talent.kind == 'Legendary', accent)
```

Do not change talent coordinates, prerequisite edges, or sparse-map layout.

- [ ] **Step 3: Improve route contrast**

Purchased route = full accent; reachable route = 55% accent; locked/future route retains hidden/dim behavior. Selection may brighten already-computed selected-route segments only.

- [ ] **Step 4: Add bounded center atmosphere in the correct draw order**

Create a fixed star/vignette table once at module initialization. Add `draw_backdrop180()` and call it **before** the wrapped Balatro draw, then call the HUD after Balatro draw:

```lua
love.draw = function(...)
  draw_backdrop180()
  old_draw_v1713(...)
  draw_direct_hud180()
end
```

`draw_backdrop180()` may draw only static gradient/vignette/sparse-star primitives and may not mutate/grow tables. The post-draw HUD may add a very light center vignette overlay only if it does not obscure nodes.

- [ ] **Step 5: Extend the contract test for wrapper order**

The Python test must assert the final wrapper contains `draw_backdrop180()` before `old_draw_v1713(...)`, and the HUD call after it.

- [ ] **Step 6: Test, parse, commit**

```bash
texlua tests/test_ui_theme.lua
python tests/test_direct_hud_contract.py
texluac -p talent_map.lua
git add talent_map.lua tests/test_ui_theme.lua tests/test_direct_hud_contract.py
git commit -m "feat: polish constellation state hierarchy"
```

---

### Task 6: Add explicit cache/performance regression guards

**Files:**
- Modify: direct HUD state table/functions in `talent_map.lua`
- Create: `tests/test_ui_performance_contract.py`
- Modify: `TESTING_CHECKLIST.md`

**Interfaces:**
- Produces numeric-only `H.debug={draws,inspector_builds,button_peak}`.

- [ ] **Step 1: Write the failing source contract**

Assert: no UIBox/G.UIT creation in the final direct-HUD block; no retained `table.insert(H.buttons,...)` history; `inspector_builds` increments only in the cache-rebuild path.

- [ ] **Step 2: Add bounded counters**

```lua
H.debug = H.debug or {draws=0, inspector_builds=0, button_peak=0}
```

Increment `draws` once per HUD frame; `inspector_builds` only when rebuilding a dirty/key-mismatched model; `button_peak=math.max(button_peak,H.button_count or 0)`.

- [ ] **Step 3: Add the exact Android stress sequence**

```text
1. Open Talent Universe.
2. Tap 10 different stars 20 times each (200 selections).
3. Perform 25 zoom-in + zoom-out cycles (50 camera changes).
4. Pan across all quadrants for 60 seconds.
5. Toggle Help/Settings/Talent mode 30 times.
6. Close/reopen Talent Universe 5 times.
7. PASS only if latency does not progressively increase and no side panel disappears.
```

- [ ] **Step 4: Test, parse, commit**

```bash
python tests/test_direct_hud_contract.py
python tests/test_ui_performance_contract.py
texluac -p talent_map.lua
git add talent_map.lua TESTING_CHECKLIST.md tests/test_ui_performance_contract.py
git commit -m "test: guard direct HUD performance architecture"
```

---

### Task 7: Full static QA and v1.8.0 metadata

**Files:**
- Modify: `README.md`, `CHANGELOG.md`, `DeckTalents.json`, `main.lua`
- Create: `docs/validation/2026-09-01-ui-release.md`

**Interfaces:**
- Produces v1.8.0 release-candidate metadata and validation record.

- [ ] **Step 1: Set version metadata**

`DeckTalents.json` version and `MOD.debug_info.version` become `1.8.0`. Changelog heading is exactly:

```text
Deck Talents 1.8.0 — HYBRID TALENT UNIVERSE / FULL DECK BALANCE PASS
```

- [ ] **Step 2: Update README**

Describe the hybrid direct-draw UI, per-profile progression, Red benchmark, and full non-Red strong-but-fair balance pass.

- [ ] **Step 3: Run every test from both plans**

```bash
find . -maxdepth 1 -name '*.lua' -print0 | xargs -0 -n1 texluac -p
for t in tests/test_*.lua; do texlua "$t"; done
for t in tests/test_*.py; do python "$t"; done
python -m json.tool DeckTalents.json >/dev/null
luacheck *.lua tests/*.lua
```

- [ ] **Step 4: Record exact results**

The validation note records command, exit code, Luacheck warnings retained, and that Android cumulative-lag validation is a device test of the release ZIP.

- [ ] **Step 5: Commit**

```bash
git add README.md CHANGELOG.md DeckTalents.json main.lua docs/validation/2026-09-01-ui-release.md
git commit -m "release: prepare Deck Talents 1.8.0"
```

---

### Task 8: Package and verify v1.8.0

**Files:**
- Generate: `/mnt/data/DeckTalents_v1.8.0.zip`

**Interfaces:**
- Produces one installable ZIP with top-level `DeckTalents/`.

- [ ] **Step 1: Stage release contents**

```bash
rm -rf /tmp/DeckTalents-release
mkdir -p /tmp/DeckTalents-release/DeckTalents
rsync -a --exclude '.git/' --exclude 'tests/' --exclude 'docs/superpowers/' --exclude 'docs/validation/' ./ /tmp/DeckTalents-release/DeckTalents/
```

- [ ] **Step 2: Build ZIP**

```bash
cd /tmp/DeckTalents-release
zip -qr /mnt/data/DeckTalents_v1.8.0.zip DeckTalents
```

- [ ] **Step 3: Verify integrity/structure**

```bash
unzip -t /mnt/data/DeckTalents_v1.8.0.zip
unzip -l /mnt/data/DeckTalents_v1.8.0.zip | head -80
```

Must include `main.lua`, `talents.lua`, `deck_expansion.lua`, `talent_map.lua`, `balance_runtime.lua`, `ui_theme.lua`, `ui_inspector.lua`, and `assets/1x`, `assets/2x`.

- [ ] **Step 4: Parse the packaged Lua copies**

```bash
rm -rf /tmp/decktalents-package-check
mkdir -p /tmp/decktalents-package-check
unzip -q /mnt/data/DeckTalents_v1.8.0.zip -d /tmp/decktalents-package-check
find /tmp/decktalents-package-check/DeckTalents -maxdepth 1 -name '*.lua' -print0 | xargs -0 -n1 texluac -p
```

- [ ] **Step 5: Freeze the artifact after validation**

Any code change after this step requires rebuilding and rerunning Steps 2-4. The exact ZIP produced here is the device-playtest candidate.