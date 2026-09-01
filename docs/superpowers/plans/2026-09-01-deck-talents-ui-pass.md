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
- Inspector model/cache invalidates only on selection, profile, Ante, relevant run-state change, or explicit Help/Settings mode change.
- Screen geometry remains usable in Android landscape and scales from the existing `screen_scale1713` behavior.
- Existing constellation talent routes/positions are not rearranged in this pass.
- Existing profile progression/reset behavior remains unchanged.
- Figma is used to settle visual proportions/tokens; runtime visuals are recreated with lightweight LÖVE drawing and existing sprite assets.
- Final release version is `1.8.0`.

---

### Task 1: Build and approve the Figma visual reference

**Files:**
- No runtime files modified.
- Create/update project note after design: `docs/ui/2026-09-01-hybrid-ui-reference.md`

**Interfaces:**
- Consumes: approved layout from the design spec.
- Produces: one Figma Design file named `Deck Talents v1.8 — Hybrid Talent Universe` and a documented token/proportion reference for code.

- [ ] **Step 1: Load the required Figma skills before any Figma write**

Use `figma-use` and `figma-generate-design`; do not call Figma write tools before loading those skills.

- [ ] **Step 2: Create one 1920×1080 primary frame**

Frame name: `Talent Universe / Hybrid / 1920x1080`.

Use these exact region proportions:

```text
Outer margin: 24 px
Top header: 86 px high
Left rail: 270 px wide
Right inspector: 430 px wide
Bottom hint row: 34 px high
Gap between side rails and constellation: 18 px
Panel corner radius reference: 14 px
Primary button corner radius reference: 10 px
```

- [ ] **Step 3: Build the visual hierarchy in Figma**

Include: Balatro-like chunky header/panels; subtle cosmic center background; one locked, one unlockable, one purchased, one selected, and one legendary node; bright purchased path vs dim reachable path; top profile/TP controls; left rail; detailed inspector; bottom hint row.

- [ ] **Step 4: Use only two deck accent examples**

Use Red and Plasma to verify that the same geometry works for a warm and cool/purple accent. Do not build 15 separate screens.

- [ ] **Step 5: Capture a screenshot and write the implementation reference note**

`docs/ui/2026-09-01-hybrid-ui-reference.md` records the exact geometry above plus these runtime rules:

```text
Panel background alpha: 0.88
Inner panel alpha: 0.78
Locked node alpha: 0.28
Unlockable node alpha: 0.72
Purchased node alpha: 1.00
Selected outline: white, 3 px at 1x reference scale
Legendary outline/glow: gold, 4 px at 1x reference scale
Background decoration: static/bounded only; no object creation per frame
```

- [ ] **Step 6: Commit the reference note**

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
- Modify: `talent_map.lua` loader section before the final direct HUD block

**Interfaces:**
- Produces `DT.ui_theme` with:
  - `layout(screen_w, screen_h) -> table`
  - `mix(colour, strength, alpha) -> colour`
  - `node_style(state, legendary, accent) -> table`
- Produces `DT.ui_inspector` with:
  - `format(args) -> model`
- `format(args)` accepts exact keys: `title`, `kind`, `description`, `trigger`, `scaling`, `current`, `ante8`, `status`, `cost`, `requirements`, `accent`.

- [ ] **Step 1: Write failing geometry/state tests**

```lua
local T = dofile('ui_theme.lua')
local g = T.layout(1920, 1080)
assert(g.header_h == 86)
assert(g.left_w == 270)
assert(g.right_w == 430)
assert(g.bottom_h == 34)
assert(g.center_w == 1920 - 48 - 270 - 430 - 36)

local locked = T.node_style('locked', false, {1,0,0,1})
local ready = T.node_style('unlockable', false, {1,0,0,1})
local owned = T.node_style('purchased', false, {1,0,0,1})
local legendary = T.node_style('purchased', true, {1,0,0,1})
assert(locked.alpha == 0.28)
assert(ready.alpha == 0.72)
assert(owned.alpha == 1.00)
assert(legendary.outline_width == 4)
print('ui theme OK')
```

- [ ] **Step 2: Write failing inspector-formatting test**

```lua
local I = dofile('ui_inspector.lua')
local m = I.format({
  title='Anger Management', kind='Strong', description='+2 Mult per Ante.',
  trigger='After first discard this Blind', scaling='+2 Mult / Ante',
  current='+10 Mult at Ante 5', ante8='+16 Mult', status='OWNED - ACTIVE',
  cost='3 TP', requirements={'Emergency Discard'}, accent={1,0,0,1},
})
assert(m.sections[1].label == 'EFFECT')
assert(m.sections[2].label == 'TRIGGER')
assert(m.sections[3].label == 'SCALING')
assert(m.sections[4].label == 'CURRENT')
assert(m.sections[5].label == 'ANTE 8')
print('ui inspector OK')
```

- [ ] **Step 3: Run tests and confirm missing-module failures**

```bash
texlua tests/test_ui_theme.lua
texlua tests/test_ui_inspector.lua
```

- [ ] **Step 4: Implement both pure modules**

`layout()` must scale the reference geometry uniformly using `min(screen_w/1920, screen_h/1080)` and return pixel-space rectangles for header, left, center, right, and bottom. `node_style()` returns deterministic alpha/outline/glow values; it must not allocate or retain global state.

`ui_inspector.format()` builds a plain-data model and omits only sections whose input is nil/empty; it never touches `G`, LÖVE, or Steamodded.

- [ ] **Step 5: Load modules once in `talent_map.lua`**

```lua
local ui_theme_chunk = SMODS.load_file('ui_theme.lua')
if not ui_theme_chunk then error('Deck Talents could not load ui_theme.lua') end
DT.ui_theme = ui_theme_chunk()

local ui_inspector_chunk = SMODS.load_file('ui_inspector.lua')
if not ui_inspector_chunk then error('Deck Talents could not load ui_inspector.lua') end
DT.ui_inspector = ui_inspector_chunk()
```

- [ ] **Step 6: Run tests/parser and commit**

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
- Modify: `talent_map.lua:5391-end` final v1.7.13 direct-HUD block
- Create: `tests/test_direct_hud_contract.py`

**Interfaces:**
- Consumes: `DT.ui_theme.layout()`.
- Produces direct-drawn functions `draw_header180`, `draw_left180`, `draw_bottom_hints180`, and existing hitbox table behavior through `button1713` or its renamed v1.8 equivalent.

- [ ] **Step 1: Write a failing static contract test**

```python
from pathlib import Path
src = Path('talent_map.lua').read_text(encoding='utf-8')
marker = 'Deck Talents v1.7.13 DIRECT HUD'
assert marker in src
final = src[src.index(marker):]
assert 'draw_header180' in final
assert 'draw_bottom_hints180' in final
assert 'UIBox({' not in final
assert 'UIBox{' not in final
assert 'G.UIT.' not in final
print('direct HUD contract OK')
```

- [ ] **Step 2: Run and confirm it fails on missing v1.8 draw functions**

```bash
python tests/test_direct_hud_contract.py
```

- [ ] **Step 3: Implement the top header**

Header content, left-to-right: active deck/global label + emblem text, Balatro Profile number, TP total, `HELP`, `SETTINGS`, `RESET`, `BACK`. `RESET` calls existing `G.FUNCS.dt_reset_profile_progression` and retains the two-tap/5-second confirmation behavior.

- [ ] **Step 4: Simplify the left rail**

Keep: Universe/deck focus label, `CENTER GLOBAL`, Ante Scaling, zoom `- / % / +`, `OVERVIEW`, and a compact five-state legend. Move `LIVE POWER` and `LEGACY` into Help/Settings utility rows so the left rail is not a button stack.

- [ ] **Step 5: Add the always-visible bottom hint row**

Exact text at 1x reference:

```text
DRAG TO PAN    •    WHEEL / +/- TO ZOOM    •    TAP A STAR TO INSPECT
```

- [ ] **Step 6: Keep hitbox allocation bounded**

`reset_buttons1713()` may reset/reuse the array count each draw, but the code must not append unbounded history. After drawing, `H.button_count` must equal the number of visible controls for that frame.

- [ ] **Step 7: Run the contract test/parser and commit**

```bash
python tests/test_direct_hud_contract.py
texluac -p talent_map.lua
git add talent_map.lua tests/test_direct_hud_contract.py
git commit -m "feat: add hybrid direct HUD shell"
```

---

### Task 4: Build the structured talent inspector and in-place Help/Settings modes

**Files:**
- Modify: `talent_map.lua` functions replacing `inspector_model1713`, `draw_empty_inspector1713`, `draw_inspector1713`
- Modify: `main.lua` only if a live-preview helper needs a stable trigger/scaling string accessor
- Modify: `tests/test_ui_inspector.lua`

**Interfaces:**
- Consumes: final balance descriptions/live values and `DT.ui_inspector.format()`.
- Produces: `H.mode` with exact values `talent`, `help`, `settings`; `H.mode='talent'` is default.

- [ ] **Step 1: Extend the inspector test for optional sections**

Verify a non-Ante talent omits `ANTE 8`, a cosmetic talent can omit `CURRENT`, and a locked talent still returns Cost/State and Requirements.

- [ ] **Step 2: Refactor `inspector_model1713()` to cache structured fields**

Cache key must include selected scope/deck/id, profile key, current Ante, points, enabled state, and a compact relevant-state token. Do not invalidate merely because `love.draw` ran.

- [ ] **Step 3: Render the sections in this exact order**

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

- [ ] **Step 4: Implement Help mode in the right panel**

Help mode explains the five node states, pan/zoom/tap controls, `LIVE POWER`, `LEGACY`, and Ante Scaling. It is plain direct-drawn text/buttons; no overlay/UIBox is created.

- [ ] **Step 5: Implement Settings mode in the right panel**

Settings mode shows active profile, Ante Scaling toggle, `LIVE POWER`, `LEGACY`, and `RESET CURRENT PROFILE`. Reset uses the existing double-tap callback; Settings itself does not mutate profile state directly.

- [ ] **Step 6: Test, parse, and commit**

```bash
texlua tests/test_ui_inspector.lua
python tests/test_direct_hud_contract.py
texluac -p talent_map.lua
texluac -p main.lua
git add talent_map.lua main.lua tests/test_ui_inspector.lua
git commit -m "feat: add structured direct-draw inspector"
```

---

### Task 5: Polish constellation nodes, routes, and center backdrop

**Files:**
- Modify: `talent_map.lua:136-175`, `talent_map.lua:2738-3140`, final `constellation_node` override around the v1.7.1+ section
- Modify: `tests/test_ui_theme.lua`

**Interfaces:**
- Consumes: `DT.ui_theme.node_style()` and existing deck accent colors.
- Produces: five visually distinct node states without changing positions or prerequisite graph.

- [ ] **Step 1: Add node-state assertions**

Test exact state names: `locked`, `unlockable`, `purchased`, `selected`, `legendary`. Legendary selected remains gold-dominant with white selection outline; purchased non-legendary uses deck accent.

- [ ] **Step 2: Route the final constellation node renderer through theme state**

State mapping:

```lua
if selected then state = 'selected'
elseif owned then state = 'purchased'
elseif ready then state = 'unlockable'
else state = 'locked' end
```

Legendary is a separate boolean, not a sixth ownership state.

- [ ] **Step 3: Improve route contrast without rebuilding routes on selection**

Purchased route: strongest deck/global accent. Reachable route: 55% accent. Locked/future route: existing hidden/dim behavior. Selected path can brighten using already-computed selected route state; do not alter graph coordinates.

- [ ] **Step 4: Add a static center backdrop drawn by the direct layer**

Draw only a vignette/very sparse deterministic stars using a fixed precomputed table created once when the module loads. No `math.random()` or table growth inside `love.draw`.

- [ ] **Step 5: Run tests/parser and commit**

```bash
texlua tests/test_ui_theme.lua
python tests/test_direct_hud_contract.py
texluac -p talent_map.lua
git add talent_map.lua tests/test_ui_theme.lua
git commit -m "feat: polish constellation state hierarchy"
```

---

### Task 6: Add explicit cache/performance regression instrumentation

**Files:**
- Modify: `talent_map.lua` direct HUD state table only
- Create: `tests/test_ui_performance_contract.py`
- Modify: `TESTING_CHECKLIST.md`

**Interfaces:**
- Produces debug counters in `H.debug` only: `draws`, `inspector_builds`, `button_peak`. Counters are numbers and do not retain object references.

- [ ] **Step 1: Write a failing source contract**

The Python test asserts that the direct HUD has no `UIBox` creation, no `G.UIT` construction, no `table.insert(H.buttons` history, and that `inspector_builds` increments only inside the inspector model rebuild path.

- [ ] **Step 2: Add bounded counters**

```lua
H.debug = H.debug or {draws=0, inspector_builds=0, button_peak=0}
```

Increment `draws` once per direct HUD draw; increment `inspector_builds` only when a dirty/key-mismatch model is rebuilt; update `button_peak = math.max(button_peak, H.button_count or 0)`.

- [ ] **Step 3: Add the Android regression loop to `TESTING_CHECKLIST.md`**

Exact manual sequence:

```text
1. Open Talent Universe.
2. Tap between 10 different stars 20 times each (200 selections total).
3. Perform 25 zoom-in + zoom-out cycles (50 camera changes).
4. Pan across all four quadrants for 60 seconds.
5. Toggle Help/Settings/Talent inspector 30 times.
6. Close/reopen Talent Universe 5 times.
7. PASS only if interaction latency does not progressively increase and no side panel disappears.
```

- [ ] **Step 4: Run static checks and commit**

```bash
python tests/test_direct_hud_contract.py
python tests/test_ui_performance_contract.py
texluac -p talent_map.lua
git add talent_map.lua TESTING_CHECKLIST.md tests/test_ui_performance_contract.py
git commit -m "test: guard direct HUD performance architecture"
```

---

### Task 7: Full static QA and documentation sync

**Files:**
- Modify: `README.md`
- Modify: `CHANGELOG.md`
- Modify: `DeckTalents.json`
- Modify: `main.lua` debug metadata/version note
- Create: `docs/validation/2026-09-01-ui-release.md`

**Interfaces:**
- Consumes: completed balance + UI changes.
- Produces: v1.8.0 release candidate metadata and validation record.

- [ ] **Step 1: Bump release metadata to `1.8.0`**

`DeckTalents.json` version and `MOD.debug_info.version` must both be `1.8.0`. Changelog heading: `Deck Talents 1.8.0 — HYBRID TALENT UNIVERSE / FULL DECK BALANCE PASS`.

- [ ] **Step 2: Update README**

Describe the hybrid direct-draw UI, per-profile progression, Red benchmark, and that all remaining vanilla deck trees received the strong-but-fair pass.

- [ ] **Step 3: Run all tests from both plans**

```bash
find . -maxdepth 1 -name '*.lua' -print0 | xargs -0 -n1 texluac -p
for t in tests/test_*.lua; do texlua "$t"; done
for t in tests/test_*.py; do python "$t"; done
python -m json.tool DeckTalents.json >/dev/null
luacheck *.lua tests/*.lua
```

- [ ] **Step 4: Confirm no forbidden direct-HUD construction**

```bash
python tests/test_direct_hud_contract.py
python tests/test_ui_performance_contract.py
```

- [ ] **Step 5: Record exact command outputs in the release validation note**

Include parser/test/Luacheck/JSON results and note that Android cumulative-lag testing remains a device-level validation item until run in Balatro.

- [ ] **Step 6: Commit**

```bash
git add README.md CHANGELOG.md DeckTalents.json main.lua docs/validation/2026-09-01-ui-release.md
git commit -m "release: prepare Deck Talents 1.8.0"
```

---

### Task 8: Package v1.8.0 and verify the archive

**Files:**
- Generated artifact: `DeckTalents_v1.8.0.zip`

**Interfaces:**
- Consumes: clean v1.8.0 worktree.
- Produces: installable Deck Talents mod ZIP with a single top-level `DeckTalents/` directory.

- [ ] **Step 1: Create a clean staging folder**

```bash
rm -rf /tmp/DeckTalents-release
mkdir -p /tmp/DeckTalents-release/DeckTalents
rsync -a --exclude '.git/' --exclude 'tests/' --exclude 'docs/superpowers/' --exclude 'docs/validation/' ./ /tmp/DeckTalents-release/DeckTalents/
```

- [ ] **Step 2: Build the ZIP**

```bash
cd /tmp/DeckTalents-release
zip -qr /mnt/data/DeckTalents_v1.8.0.zip DeckTalents
```

- [ ] **Step 3: Verify archive integrity and structure**

```bash
unzip -t /mnt/data/DeckTalents_v1.8.0.zip
unzip -l /mnt/data/DeckTalents_v1.8.0.zip | head -80
```

Expected: no archive errors; `DeckTalents/main.lua`, `DeckTalents/talents.lua`, `DeckTalents/deck_expansion.lua`, `DeckTalents/talent_map.lua`, `DeckTalents/balance_runtime.lua`, `DeckTalents/ui_theme.lua`, `DeckTalents/ui_inspector.lua`, and both asset-resolution directories are present.

- [ ] **Step 4: Parse the packaged Lua files, not only the worktree copies**

```bash
rm -rf /tmp/decktalents-package-check
mkdir -p /tmp/decktalents-package-check
unzip -q /mnt/data/DeckTalents_v1.8.0.zip -d /tmp/decktalents-package-check
find /tmp/decktalents-package-check/DeckTalents -maxdepth 1 -name '*.lua' -print0 | xargs -0 -n1 texluac -p
```

- [ ] **Step 5: Commit any final validation-note update, but never modify code after the package check without rebuilding/rechecking the ZIP**

The generated ZIP is the user-facing deliverable. The next action after this task is device playtesting of the exact ZIP, not another silent code edit.