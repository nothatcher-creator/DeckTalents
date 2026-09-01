# Deck Talents Balance & Runtime Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebalance every non-Red Deck Talents tree to the approved strong-but-fair targets while preserving v1.7.14+ profile progression and adding deterministic runtime support for consumable-use and Erratic mechanics.

**Architecture:** Restore the complete v1.7.15 package as the implementation baseline, keep talent IDs stable, and continue using the existing data-driven `talents.lua` + `deck_expansion.lua` model. Add one pure helper module for condition checks, capped factors, per-Blind consumable flags, and discrete random choices; `main.lua` remains the Steamodded/LÖVE integration layer.

**Tech Stack:** Lua / LÖVE 11.5, Steamodded, Balatro 1.0.1o-FULL [M], `texluac -p`, Luacheck 1.2.0, Python source-contract tests, JSON metadata, Git/GitHub.

**Spec:** `docs/superpowers/specs/2026-09-01-deck-talents-ui-balance-design.md`

## Global Constraints

- Baseline artifact: `/mnt/data/DeckTalents_v1.7.15_red_tree_balance.zip`.
- Baseline SHA-256: `65b4c7b26607cb03996433f937b0ed273e585c2fc9b6a0c0c46fd1d9cdbaa4d8`.
- Red Deck balance remains unchanged unless a functional bug is found.
- Existing talent IDs and prerequisite IDs remain unchanged so purchased talents survive the update.
- Existing v1.7.14+ per-profile progression/reset behavior remains compatible.
- Magic/Ghost per-Blind flags are runtime-only and reset at Blind start.
- Hard caps in the spec remain hard caps even when other mods add resources.
- Talents not explicitly named in the exact-target sections keep their v1.7.15 behavior in this first pass.
- Descriptions, gameplay behavior, live previews, and docs must agree.
- This plan does not redesign the constellation visuals; the separate UI plan owns that work.

---

### Task 1: Restore the complete v1.7.15 source baseline

**Files:**
- Restore/create: `main.lua`, `talents.lua`, `deck_expansion.lua`, `talent_map.lua`, `sprites.lua`, `global_talents.lua`, `talent_shop.lua`, `talentless_deck.lua`
- Restore/update: `README.md`, `CHANGELOG.md`, `NEW_DECK_TALENTS.md`, `LEGENDARY_TALENTS.md`, `TESTING_CHECKLIST.md`, `DeckTalents.json`, `config.lua`, `assets/1x/*`, `assets/2x/*`

**Interfaces:**
- Consumes: v1.7.15 ZIP named above.
- Produces: repository source matching the package before balance edits.

- [ ] **Step 1: Verify the artifact**

```bash
sha256sum /mnt/data/DeckTalents_v1.7.15_red_tree_balance.zip
```

Expected: `65b4c7b26607cb03996433f937b0ed273e585c2fc9b6a0c0c46fd1d9cdbaa4d8`.

- [ ] **Step 2: Stage and sync the package while preserving planning docs**

```bash
rm -rf /tmp/decktalents-v1715
mkdir -p /tmp/decktalents-v1715
unzip -q /mnt/data/DeckTalents_v1.7.15_red_tree_balance.zip -d /tmp/decktalents-v1715
rsync -a --exclude 'docs/superpowers/' /tmp/decktalents-v1715/DeckTalents/ ./
```

- [ ] **Step 3: Parse all baseline Lua**

```bash
find . -maxdepth 1 -name '*.lua' -print0 | xargs -0 -n1 texluac -p
```

Expected: exit code 0.

- [ ] **Step 4: Commit**

```bash
git add main.lua talents.lua deck_expansion.lua talent_map.lua sprites.lua global_talents.lua talent_shop.lua talentless_deck.lua README.md CHANGELOG.md NEW_DECK_TALENTS.md LEGENDARY_TALENTS.md TESTING_CHECKLIST.md DeckTalents.json config.lua assets
git commit -m "chore: restore complete v1.7.15 source baseline"
```

---

### Task 2: Add a pure balance-runtime helper

**Files:**
- Create: `balance_runtime.lua`
- Create: `tests/test_balance_runtime.lua`
- Modify: `main.lua:1-55`, `main.lua:2918-3040`

**Interfaces:**
- Produces `DT.balance_runtime` with exact functions:
  - `reset_blind_state(game) -> nil`
  - `mark_consumable_used(game, set) -> nil`
  - `used_type_count(game) -> integer`
  - `condition_passes(condition, data) -> boolean`
  - `factor(per, data, cap) -> number`
  - `choice_index(roll, count) -> integer`
  - `resolve_choice(effect, roll) -> {stat:string, amount:number}|nil`

- [ ] **Step 1: Write the failing test**

```lua
local R = dofile('balance_runtime.lua')
local game = {}
R.reset_blind_state(game)
R.mark_consumable_used(game, 'Tarot')
R.mark_consumable_used(game, 'Spectral')
assert(R.used_type_count(game) == 2)

local d = {
  after_discard=true, discards_left=0, hands_left=5, hands_played=0,
  resource_total=12, dollars=100, jokers=3, joker_limit=7,
  empty_joker_slots=4, consumables=3, types={Tarot=true,Planet=true,Spectral=true},
  type_count=3, used_types={Tarot=true,Spectral=true}, used_type_count=2,
  hand_level=10, enhanced=5, numbered=5, faces=0, hearts=3, spades=2,
  tags=12, hand_cards=5, hand_limit=8, played_cards=5, boss=true, flush=true,
}
assert(R.condition_passes({type='after_discard'}, d))
assert(R.condition_passes({type='used_tarot'}, d))
assert(R.condition_passes({type='used_spectral'}, d))
assert(not R.condition_passes({type='used_planet'}, d))
assert(R.condition_passes({type='used_type_count_at_least',value=2}, d))
assert(not R.condition_passes({type='used_all_consumable_types'}, d))
assert(R.factor('resource_total', d, 8) == 8)
assert(R.factor('tags', d, 8) == 8)
assert(R.factor('empty_joker_slots', d, 3) == 3)
assert(R.choice_index(0.00,4) == 1)
assert(R.choice_index(0.26,4) == 2)
assert(R.choice_index(0.99,4) == 4)
local c = R.resolve_choice({choices={{stat='chips',amount=240},{stat='mult',amount=48},{stat='xmult',amount=2.5}}},0.40)
assert(c.stat == 'mult' and c.amount == 48)
print('balance runtime OK')
```

- [ ] **Step 2: Run and confirm missing-module failure**

```bash
texlua tests/test_balance_runtime.lua
```

- [ ] **Step 3: Implement `balance_runtime.lua`**

Use this exact state/choice skeleton:

```lua
local R = {}
local function n(v) return tonumber(v) or 0 end
local function capped(v, cap) return math.min(cap, math.max(0, n(v))) end

function R.reset_blind_state(game)
  game.dt_consumable_types_used = {}
  game.dt_expansion_hand_choices = {}
end

function R.mark_consumable_used(game, set)
  game.dt_consumable_types_used = game.dt_consumable_types_used or {}
  if set == 'Tarot' or set == 'Planet' or set == 'Spectral' then
    game.dt_consumable_types_used[set] = true
  end
end

function R.used_type_count(game)
  local t = game and game.dt_consumable_types_used or {}
  return (t.Tarot and 1 or 0) + (t.Planet and 1 or 0) + (t.Spectral and 1 or 0)
end

function R.choice_index(roll, count)
  count = math.max(1, math.floor(n(count)))
  roll = math.max(0, math.min(0.999999999, n(roll)))
  return math.floor(roll * count) + 1
end

function R.resolve_choice(effect, roll)
  local choices = effect and effect.choices or {}
  local choice = choices[R.choice_index(roll, #choices)]
  if not choice then return nil end
  return {stat=choice.stat, amount=n(choice.amount)}
end
```

Implement `condition_passes` with every existing v1.7.15 condition plus the new use-state conditions:

```lua
function R.condition_passes(c, d)
  if not c then return true end
  local k, v = c.type, n(c.value)
  if k == 'after_discard' then return d.after_discard == true end
  if k == 'no_discards' then return d.discards_left <= 0 end
  if k == 'first_hand' then return d.hands_played <= 0 end
  if k == 'final_hand' then return d.hands_left <= 0 end
  if k == 'boss' then return d.boss == true end
  if k == 'hands_left_at_least' then return d.hands_left >= v end
  if k == 'hands_left_at_most' then return d.hands_left <= v end
  if k == 'discards_left_at_least' then return d.discards_left >= v end
  if k == 'resource_total_at_least' then return d.resource_total >= v end
  if k == 'dollars_at_least' then return d.dollars >= v end
  if k == 'dollars_below' then return d.dollars < v end
  if k == 'jokers_at_least' then return d.jokers >= v end
  if k == 'jokers_at_most' then return d.jokers <= v end
  if k == 'joker_full' then return d.joker_limit > 0 and d.jokers >= d.joker_limit end
  if k == 'consumables_at_least' then return d.consumables >= v end
  if k == 'holds_tarot' then return d.types.Tarot == true end
  if k == 'holds_planet' then return d.types.Planet == true end
  if k == 'holds_spectral' then return d.types.Spectral == true end
  if k == 'type_count_at_least' then return d.type_count >= v end
  if k == 'used_tarot' then return d.used_types.Tarot == true end
  if k == 'used_planet' then return d.used_types.Planet == true end
  if k == 'used_spectral' then return d.used_types.Spectral == true end
  if k == 'used_type_count_at_least' then return d.used_type_count >= v end
  if k == 'used_all_consumable_types' then return d.used_type_count >= 3 end
  if k == 'hand_level_at_least' then return d.hand_level >= v end
  if k == 'enhanced_at_least' then return d.enhanced >= v end
  if k == 'all_enhanced_five' then return d.played_cards == 5 and d.enhanced == 5 end
  if k == 'no_faces' then return d.faces == 0 end
  if k == 'all_numbered_five' then return d.played_cards == 5 and d.numbered == 5 end
  if k == 'numbered_at_least' then return d.numbered >= v end
  if k == 'hearts_and_spades' then return d.hearts > 0 and d.spades > 0 end
  if k == 'all_heart_spade_five' then return d.played_cards == 5 and d.hearts + d.spades == 5 end
  if k == 'flush' then return d.flush == true end
  if k == 'played_cards_at_least' then return d.played_cards >= v end
  if k == 'hand_cards_at_least' then return d.hand_cards >= v end
  if k == 'hand_limit_at_least' then return d.hand_limit >= v end
  if k == 'tags_at_least' then return d.tags >= v end
  return true
end
```

Implement factor defaults exactly as v1.7.15, with `cap` overriding the default maximum:

```lua
function R.factor(per, d, cap)
  local values = {
    hands_left=d.hands_left, discards_left=d.discards_left, resource_total=d.resource_total,
    discards_used=d.discards_used, dollars_10=math.floor(n(d.dollars)/10), jokers=d.jokers,
    empty_joker_slots=d.empty_joker_slots, consumables=d.consumables, hand_level=d.hand_level,
    enhanced=d.enhanced, numbered=d.numbered, hearts=d.hearts, spades=d.spades,
    type_count=d.type_count, hand_cards=d.hand_cards, tags=d.tags, played_cards=d.played_cards,
  }
  local defaults = {
    hands_left=8, discards_left=8, resource_total=12, discards_used=8, dollars_10=20,
    jokers=10, empty_joker_slots=6, consumables=8, hand_level=20, enhanced=5,
    numbered=5, hearts=5, spades=5, type_count=3, hand_cards=12, tags=12, played_cards=5,
  }
  if not values[per] then return 1 end
  return capped(values[per], n(cap) > 0 and n(cap) or defaults[per])
end

return R
```

- [ ] **Step 4: Load it in `main.lua` and route condition/factor wrappers through it**

```lua
local balance_runtime_chunk = SMODS.load_file('balance_runtime.lua')
if not balance_runtime_chunk then error('Deck Talents could not load balance_runtime.lua') end
DT.balance_runtime = balance_runtime_chunk()
```

- [ ] **Step 5: Test, parse, commit**

```bash
texlua tests/test_balance_runtime.lua
texluac -p balance_runtime.lua
texluac -p main.lua
git add balance_runtime.lua tests/test_balance_runtime.lua main.lua
git commit -m "test: isolate balance runtime rules"
```

---

### Task 3: Integrate per-Blind state, caps, and deterministic choice effects

**Files:**
- Modify: `main.lua:2918-3310`, `main.lua:3839-3970`
- Create: `tests/test_balance_runtime_integration.py`

**Interfaces:**
- Consumes: `DT.balance_runtime`.
- Produces: optional effect `cap`, `choice_xmult`, and `choice_score` handling.

- [ ] **Step 1: Write a failing source-contract test**

```python
from pathlib import Path
s = Path('main.lua').read_text(encoding='utf-8')
for needle in [
    'effect.cap', 'dt_consumable_types_used', 'dt_expansion_hand_choices',
    "effect.stat == 'choice_xmult'", "effect.stat == 'choice_score'",
    'DT.balance_runtime.mark_consumable_used', 'DT.balance_runtime.reset_blind_state',
]:
    assert needle in s, needle
print('balance runtime integration OK')
```

- [ ] **Step 2: Run and confirm failure on the new integration needles**

```bash
python tests/test_balance_runtime_integration.py
```

- [ ] **Step 3: Add runtime snapshot fields**

`expansion_data(context)` adds `after_discard`, `used_types`, and `used_type_count`. Every scoring/cashout/Boss factor call becomes:

```lua
local factor = DT.balance_runtime.factor(effect.per, data, effect.cap)
```

- [ ] **Step 4: Reset and mark per-Blind consumable state**

At `context.setting_blind`:

```lua
DT.balance_runtime.reset_blind_state(G.GAME)
```

At `context.using_consumeable`, after determining `set` and before talent effects:

```lua
DT.balance_runtime.mark_consumable_used(G.GAME, set)
```

- [ ] **Step 5: Cache one random choice per talent/effect/hand**

Use key format:

```lua
local key = table.concat({talent.id,index,ante,blind_name,data.hands_played}, ':')
```

If `G.GAME.dt_expansion_hand_choices[key]` exists, reuse it. Otherwise call `expansion_random`, convert the roll through `DT.balance_runtime.choice_index`, and store the chosen index. `choice_xmult` applies the selected numeric multiplier; `choice_score` resolves exactly one selected `{stat, amount}` entry.

- [ ] **Step 6: Test, parse, commit**

```bash
python tests/test_balance_runtime_integration.py
texlua tests/test_balance_runtime.lua
texluac -p main.lua
git add main.lua tests/test_balance_runtime_integration.py
git commit -m "feat: add per-blind and discrete balance runtime"
```

---

### Task 4: Rebalance Blue, Yellow, Green, and Black

**Files:**
- Modify: `talents.lua:61-216`
- Modify: `deck_expansion.lua:230-1111`
- Modify: `main.lua` base scoring/live-preview branches for these decks
- Create: `tests/test_balance_resource_decks.lua`

**Interfaces:**
- Produces exact approved targets for resource/Joker decks.

- [ ] **Step 1: Write failing assertions for these exact targets**

```lua
local expected = {
  azure_infinity={chips_per_hands_left=40,xmult=2.0,first_hand=true,min_hands=4},
  endless_shift={xmult=1.60,min_hands=2},
  golden_singularity={boss_dollars=15,threshold=100,xmult=2.25},
  money_printer={cashout=4,threshold=100,xmult=1.60},
  liquid_assets={threshold=75,xmult=1.40},
  hostile_takeover={threshold=50,boss=true,xmult=1.65},
  planet_saver={xmult_per_resource=0.08,score_cap=8,cashout_per_resource=1,cashout_cap=6},
  perpetual_motion={xmult_per_resource=0.05,cap=8},
  compost_dividend={cashout_cap=6}, carbon_credit={boss_cap=10},
  void_storage={chips_per_empty_joker=20,cap=4},
  black_hole={max_jokers=3,mult_per_empty_joker=12,cap=4},
  full_darkness={xmult=1.35}, event_horizon={max_jokers=4,xmult=1.45},
  singularity_storage={joker_slots=1,max_jokers=4,xmult=1.55},
  abyssal_crown={joker_slots=1,mult_per_joker=10,xmult_bonus_per_empty=0.12,cap=5},
}
```

The test loads `talents.lua` and `deck_expansion.lua`, finds each ID, and asserts its conditions/effects rather than only matching description text.

- [ ] **Step 2: Run and confirm v1.7.15 mismatches**

```bash
texlua tests/test_balance_resource_decks.lua
```

- [ ] **Step 3: Update data tables, base scoring, and live previews**

Keep IDs/requirements unchanged. Encode all listed resource/Joker caps with `effect.cap`. Azure Infinity uses `first_hand` plus `hands_left_at_least=4`.

- [ ] **Step 4: Test, parse, commit**

```bash
texlua tests/test_balance_resource_decks.lua
texluac -p talents.lua
texluac -p deck_expansion.lua
texluac -p main.lua
git add talents.lua deck_expansion.lua main.lua tests/test_balance_resource_decks.lua
git commit -m "balance: retune resource and joker decks"
```

---

### Task 5: Rebalance Magic, Nebula, Ghost, and Zodiac

**Files:**
- Modify: `talents.lua:217-333`, `talents.lua:412-450`
- Modify: `deck_expansion.lua:1112-1774`, `deck_expansion.lua:2227-2454`
- Modify: `main.lua` matching base scoring/live-preview branches
- Create: `tests/test_balance_arcane_decks.lua`

**Interfaces:**
- Consumes: `used_*` conditions.
- Produces use-based Magic/Ghost, leveled Nebula, held-type Zodiac.

- [ ] **Step 1: Write failing exact-value tests**

Assert the approved spec values for every explicitly named talent, including:

```lua
is_this_your_card={condition='used_tarot',mult=20}
rabbit_hole={condition='used_planet',chips=80}
grand_illusion={used_types=2,xmult=1.50}
impossible_act={used_all=true,chips=120,mult=40,xmult=2.25}
space_dust={chips_per_level=12,cap=20}
telescope_two={mult_per_level=3}
heat_death={min_level=10,chips_per_level=25,xmult=2.10}
spirit_cabinet={condition='used_spectral',mult=40}
phantom_hand={condition='used_spectral',xmult=1.50}
beyond_the_veil={used_spectral=true,all_enhanced_five=true,chips_per_enhanced=30,xmult=2.40}
mercury_gatorade={chips_per_type=15}
big_three={held_types=3,xmult=1.60}
grand_zodiac={chips_per_type=60,held_types=3,xmult=2.25}
```

- [ ] **Step 2: Run and confirm failures**

```bash
texlua tests/test_balance_arcane_decks.lua
```

- [ ] **Step 3: Apply the table and base-scoring changes**

Base Ghost `paranormal_activity` becomes `X1.35 after using a Spectral this Blind`; it no longer checks held Spectral cards. Other unnamed talents keep v1.7.15 behavior.

- [ ] **Step 4: Test, parse, commit**

```bash
texlua tests/test_balance_arcane_decks.lua
texluac -p talents.lua
texluac -p deck_expansion.lua
texluac -p main.lua
git add talents.lua deck_expansion.lua main.lua tests/test_balance_arcane_decks.lua
git commit -m "balance: differentiate arcane and level decks"
```

---

### Task 6: Rebalance Abandoned, Checkered, Painted, and Anaglyph

**Files:**
- Modify: `talents.lua:334-528`
- Modify: `deck_expansion.lua:1775-2876`
- Modify: `main.lua` matching base scoring/live-preview branches
- Create: `tests/test_balance_pattern_decks.lua`

**Interfaces:**
- Produces numbered-card, two-suit, reserve-hand, and Tag-hoarding behavior from the spec.

- [ ] **Step 1: Write failing effect-table assertions**

Assert every explicitly named target from the spec, including:

```lua
kids_table={numbered_at_least=4,mult=20}
quiet_classroom={all_numbered_five=true,xmult=1.50}
lord_of_the_flies={all_numbered_five=true,mult=80,xmult=2.40}
checker_mate={hearts_and_spades=true,xmult=1.25}
full_board={flush=true,xmult=1.50}
perfect_checkmate={flush=true,mult=60,xmult=2.10}
gallery_expansion={mult_per_empty_joker=10,cap=3}
magnum_opus={played_cards=5,min_hand_cards=4,mult_per_hand_card=12,xmult=2.25}
infinite_labels={xmult_bonus_per_tag=0.08,cap=8}
printing_press={boss_double_tags=1,xmult_bonus_per_tag=0.10,cap=8}
```

Also assert `echo_copy.chance == 0.50` and `duplicate_everything` produces exactly one Double Tag.

- [ ] **Step 2: Run and confirm failures**

```bash
texlua tests/test_balance_pattern_decks.lua
```

- [ ] **Step 3: Apply exact data/scoring/preview changes**

Use `numbered_at_least`/`all_numbered_five` instead of no-face checks only where the spec names the replacement. Use `effect.cap=8` for Tag multipliers.

- [ ] **Step 4: Test, parse, commit**

```bash
texlua tests/test_balance_pattern_decks.lua
texluac -p talents.lua
texluac -p deck_expansion.lua
texluac -p main.lua
git add talents.lua deck_expansion.lua main.lua tests/test_balance_pattern_decks.lua
git commit -m "balance: retune pattern hand and tag decks"
```

---

### Task 7: Rebuild Plasma and Erratic scoring

**Files:**
- Modify: `talents.lua:529-606`
- Modify: `deck_expansion.lua:2877-3310`
- Modify: `main.lua` Plasma/Erratic base scoring/live preview
- Create: `tests/test_balance_plasma_erratic.lua`

**Interfaces:**
- Consumes: `choice_xmult`, `choice_score`.
- Produces additive Plasma identity and mutually-exclusive Erratic outcomes.

- [ ] **Step 1: Write failing exact tests**

Assert all Plasma values from the spec through:

```lua
star_core={chips=240,mult=48,final_hand_xmult=1.80}
```

Assert Erratic arrays exactly:

```lua
rngesus={1.0,1.5,2.0,2.5}
unstable_reality={1.0,1.5,2.0,2.5}
house_always_loses={1.5,2.0,2.5,3.0}
absolute_pandemonium={
  {stat='chips',amount=240},
  {stat='mult',amount=48},
  {stat='xmult',amount=2.5},
}
```

and assert its Boss `talent_point` effect has `chance=0.20`, `amount=1`.

- [ ] **Step 2: Run and confirm failures**

```bash
texlua tests/test_balance_plasma_erratic.lua
```

- [ ] **Step 3: Remove old unconditional Plasma XMult and implement discrete Erratic choices**

`Equal Rights` and `Fusion Reactor` become additive only. `Absolute Pandemonium` is one `choice_score` effect, not three independent random effects.

- [ ] **Step 4: Test, parse, commit**

```bash
texlua tests/test_balance_plasma_erratic.lua
python tests/test_balance_runtime_integration.py
texluac -p talents.lua
texluac -p deck_expansion.lua
texluac -p main.lua
git add talents.lua deck_expansion.lua main.lua tests/test_balance_plasma_erratic.lua
git commit -m "balance: rebuild plasma and erratic identities"
```

---

### Task 8: Synchronize previews, docs, and profile regression

**Files:**
- Modify: `main.lua` preview helpers
- Modify: `README.md`, `CHANGELOG.md`, `NEW_DECK_TALENTS.md`, `LEGENDARY_TALENTS.md`
- Create: `tests/test_profile_progression.lua`
- Create: `tests/test_docs_values.py`

**Interfaces:**
- Produces matching gameplay text and regression coverage.

- [ ] **Step 1: Add the profile regression harness**

The harness must prove: legacy shared config migrates to active Profile 2; Profile 2 points/unlocks survive switching to Profile 1; Profile 1 can gain points independently; resetting Profile 2 leaves Profile 1 unchanged.

- [ ] **Step 2: Add exact documentation assertions**

`tests/test_docs_values.py` checks headline strings for Azure Infinity, Golden Singularity, Planet Saver, Abyssal Crown, Impossible Act, Heat Death, Beyond the Veil, Lord of the Flies, Perfect Checkmate, Grand Zodiac, Magnum Opus, Printing Press, Star Core, and Absolute Pandemonium in the relevant Markdown and data files.

- [ ] **Step 3: Update live preview logic**

Magic/Ghost preview conditions use `used_types`; capped talents display the capped current value; Ante-scaled effects show current scaled value and Ante-8 reference data for the UI plan.

- [ ] **Step 4: Run tests and commit**

```bash
texlua tests/test_profile_progression.lua
python tests/test_docs_values.py
texluac -p main.lua
git add main.lua README.md CHANGELOG.md NEW_DECK_TALENTS.md LEGENDARY_TALENTS.md tests/test_profile_progression.lua tests/test_docs_values.py
git commit -m "docs: synchronize balanced talent behavior"
```

---

### Task 9: Static validation checkpoint

**Files:**
- Create: `.luacheckrc`
- Create: `docs/validation/2026-09-01-balance-runtime.md`

**Interfaces:**
- Produces a green balance/runtime checkpoint consumed by the UI plan.

- [ ] **Step 1: Verify the supplied Luacheck package**

```bash
sha256sum /mnt/data/luacheck-1.2.0.zip
```

Expected: `955a172a64dfaceb52d824ca71aab7cbdf4a89012a24164870600800a350e96b`.

- [ ] **Step 2: Add `.luacheckrc`**

```lua
std = 'lua51'
globals = {
  'SMODS','G','Event','Tag','Card','UIBox','love','DeckTalents','pseudorandom',
  'ease_dollars','add_tag','play_sound','attention_text','to_number'
}
```

Do not globally suppress unused locals, redefinitions, or undefined variables.

- [ ] **Step 3: Run all checks**

```bash
find . -maxdepth 1 -name '*.lua' -print0 | xargs -0 -n1 texluac -p
for t in tests/test_*.lua; do texlua "$t"; done
for t in tests/test_*.py; do python "$t"; done
python -m json.tool DeckTalents.json >/dev/null
luacheck *.lua tests/*.lua
```

- [ ] **Step 4: Record command, exit code, and any intentionally retained pre-existing warnings in the validation note**

- [ ] **Step 5: Commit**

```bash
git add .luacheckrc docs/validation/2026-09-01-balance-runtime.md
git commit -m "test: validate full balance runtime pass"
```

This commit is the required input to `docs/superpowers/plans/2026-09-01-deck-talents-ui-pass.md`. Do not package the final release before the UI/performance plan passes.