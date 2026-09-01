# Deck Talents UI + Balance Pass Design

Date: 2026-09-01
Target baseline: v1.7.15
Status: Design approved; spec ready for final user review

## Goals

1. Make the Talent Universe look polished and intentional while preserving the lag-free direct-draw architecture introduced in the v1.7.13 line.
2. Rebalance every non-Red deck around a strong-but-fair target.
3. Preserve each deck's identity while allowing full redesign of weak, redundant, contradictory, or excessively passive talents.
4. Keep permanent progression rewarding without allowing early progression to trivialize normal runs.
5. Make talent math readable without manual arithmetic by showing current scaled values in the inspector.

## Non-goals

- Reintroduce the old nested Balatro UIBox constellation menus.
- Replace Deck Talents with a different progression system.
- Remove joke names, flavor, or unusual deck mechanics for mathematical uniformity.
- Make every deck produce identical expected score.

## UI Direction

The approved visual direction is a hybrid: Balatro-native framing around a cosmic/RPG constellation.

### Screen structure

The Talent Universe uses four persistent regions plus a bottom hint row.

1. Top header bar
   - Deck name and icon.
   - Active Balatro profile number.
   - Talent Point total.
   - Back, Help/Legend, Settings, and profile-reset controls.

2. Left control rail
   - Universe/deck selector.
   - Center Global control.
   - Ante Scaling toggle.
   - Node-state legend.
   - No additional display toggles in this pass.

3. Center constellation canvas
   - Remains the primary interaction surface.
   - Direct-drawn vignette, sparse star/nebula treatment, and deck accent tint.
   - Branches visually distinguish locked, reachable, purchased, and selected paths.
   - Minor, major, and capstone nodes use distinct size/glow hierarchy.

4. Right talent inspector
   - Talent name.
   - Talent kind as subtitle.
   - Effect.
   - Trigger condition.
   - Scaling rule.
   - Current live value.
   - Ante-8 reference for Ante-scaled effects.
   - Cost and ownership state.

5. Bottom hint row
   - Pan.
   - Zoom.
   - Tap/select talent.
   - The row is always present but visually subdued.

### Visual language

- Chunky Balatro-like outer panels and controls.
- Cosmic/RPG node glow and connection language.
- Deck-specific accent treatment without changing screen geometry per deck.
- Five clear node states: locked, unlockable, purchased, selected, legendary/capstone.
- Capstones receive the strongest glow/outline treatment.

### Performance constraints

These are mandatory because the direct-draw path previously eliminated cumulative constellation interaction lag.

- Keep direct HUD rendering.
- Ordinary node selection must not rebuild the constellation overlay.
- Zoom and pan remain in-place state changes.
- Do not recreate dense nested UIBox trees for constellation side menus.
- Inspector data is cached and invalidated only when selection, profile, Ante, or relevant run state changes.
- Decorative animation must use bounded state and must not allocate new persistent objects every frame.
- Hitboxes remain compact plain tables rather than controller-heavy UI nodes.

## Balance Philosophy

Target: strong but fair.

### Global rules

- Early talents teach the deck identity and give noticeable but modest power.
- Mid-tree talents influence play decisions rather than merely adding passive score.
- Late talents and capstones are powerful, but large XMult requires a meaningful condition, timing restriction, or build state.
- Do not stack multiple easy unconditional XMult talents as a tree's main progression path.
- Easy triggers favor Chips, Mult, economy, consistency, slots, or utility.
- Economy talents do not simultaneously become top scoring talents without a demanding threshold.
- Resource-preservation talents reward efficiency without making non-use of resources automatically optimal.
- Talents that duplicate the base deck advantage must deepen or reshape that mechanic instead of only adding generic score.
- Ante scaling uses clean values when the mechanic is Ante-scaled.
- Existing talent IDs are retained wherever possible so purchases survive the update.

### Classification

Each non-Red talent is reviewed as Keep, Retune, Trigger Change, or Redesign. Talents not explicitly redesigned below retain their identity and ID; numerical retuning is allowed to bring them inside the same deck power budget.

## Red Deck Benchmark

Red is the benchmark established in v1.7.15 and remains unchanged unless a functional bug is found.

- Anger Management: +2 Mult per Ante after first discard.
- Seeing Red: X1.80 when no discards remain.
- Trash Talk: +3 Chips per Ante after first discard.
- Rage Meter: +0.5 Mult per discard used per Ante.
- Redline: first discard in each Boss Blind pays $4.
- Empty Chamber: +10 Chips per Ante when no discards remain.
- Scarlet Momentum: X1.40 after discarding.
- No Takebacks: +40 Mult with no discards remaining.
- Final Cut: X3 on the final hand with no discards remaining.
- Scarlet Apocalypse: +120 Mult and X3 with no discards remaining.

## Deck-by-Deck Direction and Exact Headline Targets

### Blue Deck

Identity: conserve and strategically spend hands.

- Preserve Overtime and Boss Overtime utility.
- Reduce repeated passive XMult for simply having hands left.
- First-hand and final-hand payoffs remain the strongest multiplicative moments.
- Azure Infinity: once per Blind, the first played hand while at least 4 hands remain gains +40 Chips per remaining hand and X2 Mult. It cannot trigger again during that Blind.
- Endless Shift is retuned below Azure Infinity and cannot exceed X1.60 while 2+ hands remain.

### Yellow Deck

Identity: thresholds, investment, and economy.

- Emergency Fund remains the low-money safety valve.
- $50/$75/$100 scoring thresholds are compressed so they do not stack into an automatic win.
- Golden Singularity: Boss victories pay $15; at $100+ every played hand gains X2.25 Mult.
- Money Printer: +$4 at cashout; at $100+ gain X1.60 Mult.
- Liquid Assets: at $75+ gain X1.40 Mult.
- Hostile Takeover: Boss Blind and $50+ grants X1.65 Mult.

### Green Deck

Identity: value from unused hands/discards.

- Reusable Discards and Sustainable Hands remain utility anchors.
- Scoring from remaining resources is capped to prevent modded hand/discard counts from exploding.
- Planet Saver: gain +0.08 XMult bonus per remaining hand/discard, counting at most 8 resources; at cashout earn $1 per unused hand/discard, capped at $6.
- Perpetual Motion: +0.05 XMult bonus per remaining resource, capped at 8 resources.
- Compost Dividend cashout is capped at $6.
- Carbon Credit Boss payout is capped at $10.

### Black Deck

Identity: explicit wide-vs-lean Joker management.

- Wide branch rewards owning/filling Joker slots.
- Lean branch rewards deliberately leaving Joker slots empty.
- Extra Shelf remains +1 Joker slot.
- Void Storage no longer duplicates Extra Shelf; it becomes: +20 Chips per empty Joker slot, capped at 4 empty slots.
- Black Hole Warranty becomes: with 3 or fewer Jokers, gain +12 Mult per empty Joker slot, capped at 4 empty slots. It grants no XMult.
- Full Darkness remains a full-board payoff but is retuned to X1.35.
- Event Horizon becomes X1.45 with 4 or fewer Jokers.
- Singularity Storage grants +1 Joker slot and X1.55 with 4 or fewer Jokers.
- Abyssal Crown: +1 Joker slot; +10 Mult per owned Joker; +0.12 XMult bonus per empty Joker slot, counting at most 5 empty slots.

### Magic Deck

Identity: actively use consumables during the current Blind.

Magic is separated from Zodiac by tracking consumable types used this Blind.

- Tarot-used, Planet-used, and Spectral-used flags reset at the start of every Blind.
- Is This Your Card?: after using a Tarot this Blind, played hands gain +20 Mult.
- Rabbit Hole: after using a Planet this Blind, played hands gain +80 Chips.
- Grand Illusion: after using any 2 different consumable types this Blind, gain X1.50 Mult.
- Cheap Trick remains $1 when using a Tarot.
- Tarot Storm becomes +30 Mult after using a Tarot this Blind.
- Planet Trick becomes +100 Chips after using a Planet this Blind.
- Spectral Finale becomes X1.60 after using a Spectral this Blind.
- Master Magician: after using any 2 different consumable types this Blind, gain X1.75 Mult.
- Impossible Act: after Tarot, Planet, and Spectral have each been used in the current Blind, played hands gain +120 Chips, +40 Mult, and X2.25 Mult.

### Nebula Deck

Identity: poker-hand leveling and specialization.

- Per-level additive values are reduced because level 10-20 currently creates excessive guaranteed score.
- Space Dust: +12 Chips per played-hand level, capped at level 20.
- Telescope 2.0: +3 Mult per played-hand level.
- Moon Dust: +15 Chips per played-hand level.
- Cosmic Mult: +4 Mult per played-hand level.
- Supernova: level 5+ grants X1.50 Mult.
- Pulsar: level 6+ grants X1.60 Mult.
- Event Horizon: level 8+ grants X1.75 Mult.
- Galaxy Brain: level 10+ grants +250 Chips.
- Heat Death: level 10+ gains +25 Chips per level and X2.10 Mult.

### Ghost Deck

Identity: Spectral usage plus enhanced-card synergy.

- Add a Spectral-used-this-Blind flag that resets each Blind.
- Holding a Spectral is no longer enough to stack several large XMult bonuses.
- Possessed Hand remains enhanced-card scoring and is retuned only if testing shows it above budget.
- Spirit Cabinet becomes +40 Mult after using a Spectral this Blind.
- Phantom Hand becomes X1.50 after using a Spectral this Blind.
- Afterlife Insurance: final hand after using a Spectral this Blind gains X1.90.
- Mass Haunting: five enhanced played cards gain X2.10.
- Beyond the Veil: after using a Spectral this Blind, a five-card hand made entirely of enhanced cards gains +30 Chips per enhanced played card and X2.40 Mult.

### Abandoned Deck

Identity: numbered-card mastery.

- "No face cards" checks are replaced when they are effectively free on Abandoned Deck.
- Kid's Table: hands with at least 4 numbered played cards gain +20 Mult.
- No Chaperones: hands with at least 4 numbered played cards gain +30 Mult.
- Quiet Classroom: five numbered played cards gain X1.50.
- Detention: final hand with at least 4 numbered played cards gains X1.90.
- Perfect Attendance remains a five-numbered-card condition and is retuned to X1.35.
- Kids Rule: five numbered cards gain +70 Mult.
- Lord of the Flies: five numbered cards gain +80 Mult and X2.40 Mult.

### Checkered Deck

Identity: Hearts/Spades and Flush manipulation.

- Generic Flush XMult is reduced because Checkered makes Flushes much easier.
- Checker Mate remains the mixed-suit entry reward at X1.25.
- Flush Rush becomes X1.30.
- Red and Black becomes X1.45 for hands containing both Hearts and Spades.
- Double Weave becomes X1.70 for five-card hands made entirely of Hearts and Spades.
- Royal Fabric becomes +50 Mult on Flushes.
- Grand Pattern becomes +180 Chips on Flushes.
- Full Board becomes X1.50 on Flushes.
- Perfect Checkmate: Flushes gain +60 Mult and X2.10 Mult.

### Zodiac Deck

Identity: held consumable-type diversity and alignment.

- Magic owns use-based consumable triggers; Zodiac owns held-type diversity.
- Mercury in Gatorade becomes +15 Chips per distinct held consumable type.
- Perfect Alignment remains 2+ types and becomes +18 Mult.
- The Big Three becomes X1.60 while all 3 types are represented.
- Spectral Moon becomes X1.25 while holding a Spectral.
- Twin Signs becomes X1.35 with 2+ held types.
- Cosmic Alignment becomes X1.75 with all 3 types.
- Fate Written becomes X2.00 against Boss Blinds while all 3 types are represented.
- Grand Zodiac: +60 Chips per distinct held consumable type; when all 3 types are represented also gain X2.25 Mult.

### Painted Deck

Identity: large hands and constrained Joker space.

- Gallery Expansion no longer immediately erases the deck weakness. It becomes: +10 Mult per empty Joker slot, capped at 3 empty slots.
- Extra Frame remains the single later +1 Joker-slot investment.
- Finger Painter remains a cards-in-hand entry reward.
- Negative Space becomes X1.35 with at least 4 cards remaining in hand.
- Gallery Lighting becomes X1.50 at hand-size limit 8+.
- Living Art becomes X1.75 with at least 5 cards remaining in hand.
- Masterpiece becomes X1.50 at hand-size limit 7+.
- Museum Piece becomes +70 Mult for a five-card played hand with at least 4 cards remaining.
- Magnum Opus: a five-card played hand with at least 4 cards remaining gains +12 Mult per card remaining in hand and X2.25 Mult.

### Anaglyph Deck

Identity: generate, hold, and eventually cash in Tags.

- Overlapping guaranteed Double Tag generation is reduced.
- Copy Machine remains a 1-in-3 Boss chance for one Double Tag.
- Tag Pocket remains a 25% Boss chance for one Double Tag.
- Echo Chamber becomes one guaranteed Double Tag on Boss victory.
- Echo Copy becomes a 50% Boss chance for one Double Tag rather than guaranteed stacking.
- Duplicate Everything becomes one guaranteed Double Tag on Boss victory, not two.
- Infinite Labels becomes +0.08 XMult bonus per held Tag, counting at most 8 Tags.
- Printing Press: every Boss victory creates 1 Double Tag; each held Tag grants +0.10 XMult bonus, counting at most 8 Tags.

### Plasma Deck

Identity: additive Chip/Mult shaping that Plasma's native balancing can exploit.

This tree receives a major redesign away from unconditional multiplier stacking.

- Balanced Breakfast: +30 Chips and +6 Mult every played hand.
- Equal Rights: +50 Chips and +10 Mult every played hand; no XMult.
- Fusion Reactor: +100 Chips and +20 Mult every played hand; no unconditional XMult.
- Calibration: +40 Chips and +8 Mult.
- Blue Bias: +90 Chips.
- Red Bias: +18 Mult.
- Stable Core: +60 Chips and +12 Mult; no XMult.
- Balanced Load: +80 Chips and +16 Mult; no XMult.
- Overclock: first played hand gains +160 Chips and +32 Mult.
- Critical Mass: Boss Blind hands gain X1.50 Mult.
- Superconductor: final available hand gains X1.75 Mult.
- Plasma Storm: +180 Chips and +36 Mult every played hand.
- Star Core: every played hand gains +240 Chips and +48 Mult; the final available hand also gains X1.80 Mult.

### Erratic Deck

Identity: true volatility with a fair average result.

- Randomness must include misses/low rolls rather than always-positive multiplicative ranges.
- RNGesus becomes a four-way roll each hand: no XMult, X1.5, X2.0, or X2.5, with equal probability.
- Unstable Reality uses the same four-way XMult roll.
- House Always Loses final-hand roll becomes X1.5, X2.0, X2.5, or X3.0 with equal probability.
- Existing flat random Chip/Mult effects remain but are retuned only if expected value testing exceeds budget.
- Absolute Pandemonium: every played hand randomly selects exactly one of three mutually exclusive effects with equal probability: +240 Chips, +48 Mult, or X2.50 Mult. It never applies more than one of these three effects to the same hand. Boss victories have a 20% chance to award 1 bonus Talent Point.

## Inspector and Math Presentation

For every talent, the inspector separates Effect, Trigger, Scaling, Current Value, and Cost/State.

Ante-scaled example:

- Effect: +2 Mult per Ante.
- Trigger: after first discard this Blind.
- Current value: +10 Mult at Ante 5.
- Ante 8 reference: +16 Mult.

Conditional XMult must show its exact condition on a separate Trigger line.

Descriptions, gameplay logic, live previews, README/changelog text, and Legendary/New Talent documentation must agree.

## State and Migration Requirements

- Existing v1.7.14+ per-profile progression remains intact.
- Retuning or redesigning a talent does not revoke an existing purchase.
- Retain talent IDs for redesigns unless technically impossible.
- If an ID must change, add an explicit old-ID to new-ID migration.
- Ante Scaling preference and talent UI state remain profile-specific.
- New per-Blind flags for Magic/Ghost are runtime-only and reset at Blind start; they are not permanent progression data.

## Implementation Boundaries

Expected areas:

- `talents.lua` base talent descriptions/values.
- `deck_expansion.lua` expanded talent descriptions/effects.
- `main.lua` effect evaluation, new conditions/state tracking, direct HUD draw/input, inspector model/cache.
- Documentation and version/changelog metadata.

Do not perform unrelated refactors.

## Testing Strategy

### Static validation

- Parse every Lua file with an available Lua compiler/parser.
- Run Luacheck and separate known framework globals from newly introduced warnings.
- Validate JSON metadata.
- Validate release ZIP integrity.

### Persistence regression

- Existing profile progression loads unchanged.
- Switching profiles rebinds the correct progression.
- Profile reset affects only the active profile.

### Talent behavior

For each changed talent verify trigger, cap, once-per-Blind behavior, Boss-only restrictions, scoring phase/order, disabled-state handling, and state reset.

Magic/Ghost specifically verify consumable-use flags reset every Blind and cannot leak across runs.

### UI regression

- Node selection does not rebuild the full overlay.
- Repeated selection does not recreate cumulative lag.
- Pan/zoom remains responsive on Android.
- Touch hitboxes match visible controls.
- Inspector values update with Ante/run state.
- No unbounded per-frame allocation is introduced.

### Balance sanity

- Compare Ante-8 expected output against the v1.7.15 Red benchmark.
- Test high-synergy edge cases: extra hands/discards, expanded Joker slots, high poker-hand levels, large Tag stocks, large consumable inventories, and modded deck/resource counts.
- Cap-based talents must honor their stated cap even when other mods add resources.

## Release Deliverables

- Updated mod package.
- Updated version metadata.
- Updated talent descriptions and live inspector math.
- Updated README and relevant talent docs.
- Changelog covering UI and balance changes.
- Validation results before release.
- Linear implementation/playtest issues if useful after the implementation plan is approved.

## Success Criteria

1. Talent Universe is visibly more polished and easier to read while retaining the direct-draw no-lag interaction model.
2. No remaining deck uses a stack of easy unconditional XMult as its primary identity.
3. Every deck's progression changes run decisions in a way consistent with the deck fantasy.
4. Capstones feel exciting but require setup, timing, thresholds, or a narrow state.
5. Tooltip math, live inspector values, and gameplay behavior agree.
6. Existing per-profile progression remains compatible.
7. Repeated Talent Universe interaction remains free of the previous cumulative lag regression.
