# Deck Talents UI + Balance Pass Design

Date: 2026-09-01
Target baseline: v1.7.15
Status: Approved design, awaiting final spec review before implementation planning

## Goals

1. Make the Talent Universe look polished and intentional while preserving the lag-free direct-draw architecture introduced in the v1.7.13 line.
2. Rebalance every non-Red deck around a "strong but fair" target.
3. Preserve each deck's identity while allowing full redesign of weak, redundant, contradictory, or excessively passive talents.
4. Keep permanent progression rewarding without allowing early progression to trivialize normal runs.
5. Make all talent math easy to understand in the inspector, with current scaled values shown directly.

## Non-goals

- Reintroducing the old nested Balatro UIBox constellation menus.
- Replacing Deck Talents with a completely different progression system.
- Removing deck personality, joke names, or unusual mechanics in the name of balance.
- Making every deck mathematically identical.

## UI Direction

The approved visual direction is a hybrid of Balatro-native framing and a cosmic/RPG talent-tree presentation.

### Screen structure

The Talent Universe keeps a consistent four-part structure:

1. Top header bar
   - Deck name and icon.
   - Active profile indicator.
   - Talent Point total.
   - Compact utility controls for Back, Help/Legend, Settings, and profile-progression reset access.

2. Left control rail
   - Universe/deck selector.
   - Center Global / overview control.
   - Ante Scaling toggle.
   - Compact node-state legend.
   - Optional display toggles only if they can be added without layout churn.

3. Center constellation canvas
   - Remains the primary interaction surface.
   - Direct-drawn background treatment with subtle vignette, sparse stars/nebula, and deck accent tint.
   - Clearer branch states for locked, reachable, purchased, and selected paths.
   - Stronger size and glow hierarchy for minor, major, and capstone nodes.

4. Right talent inspector
   - Talent name.
   - Short category/fantasy subtitle where useful.
   - Effect.
   - Trigger condition.
   - Scaling rule.
   - Current live value.
   - Base/Ante-8 reference where relevant.
   - Cost and ownership state.

A minimal bottom hint row may show pan/zoom/tap controls where useful.

### Visual language

- Balatro-like chunky outer panels and buttons.
- Cosmic/RPG node glow and connection language.
- Deck-specific accent treatment without changing the whole layout per deck.
- Clear node states: locked, unlockable, purchased, selected, legendary/capstone.
- Capstones should be visually distinct at a glance.

### Performance constraints

These constraints are mandatory because the menu-less/direct-draw path previously eliminated cumulative interaction lag:

- Keep the direct HUD rendering model.
- Do not rebuild the full constellation for ordinary node selection.
- Keep zoom and pan in-place.
- Avoid dense nested UIBox trees for the constellation-side menus.
- Prefer cached models and targeted state changes over layout regeneration.
- New visual polish should be implemented primarily with lightweight LÖVE drawing, cached text/model data, and small hitbox tables.
- Any decorative animation must be bounded and must not allocate new objects every frame.

## Balance Philosophy

Target: strong but fair.

### Global rules

- Early talents should teach the deck's identity and provide useful but modest power.
- Mid-tree talents should meaningfully influence play decisions.
- 5-point ultimates should be exciting and build-defining, but not equivalent to a permanent unconditional free X2-X4 button.
- Easy triggers should generally grant Chips, Mult, economy, consistency, or utility.
- Large XMult should require a meaningful condition, timing restriction, resource commitment, or narrow build state.
- Avoid multiple unconditional XMult talents stacking in the same tree.
- Economy talents should not simultaneously be among the strongest scoring talents unless their trigger is genuinely demanding.
- Resource-preservation talents should reward efficiency without making "do nothing" the dominant strategy.
- Talents that merely duplicate the base deck advantage should deepen or reshape the mechanic instead of adding generic score.
- Ultimates should pay off the identity established by their preceding branches.
- Ante scaling should use clean values where possible, such as +2 Mult/Ante, +3 Chips/Ante, or +0.10 XMult bonus/Ante.
- Permanent progression is assumed, so early unlocks must remain helpful without making fresh runs automatic wins.

### Classification used during implementation

Every talent will be classified as one of:

- Keep: mechanic is healthy; only text/math cleanup needed.
- Retune: same mechanic, different values.
- Trigger change: same role, but condition must become more meaningful or less trivial.
- Redesign: mechanic is redundant, contradictory, uninteractive, or fundamentally too strong/weak.

## Deck-by-Deck Direction

### Red Deck

Red is the established benchmark from v1.7.15 and should remain unchanged unless implementation reveals a bug.

Reference outcomes:

- Anger Management: +2 Mult per Ante after first discard.
- Seeing Red: X1.80 when no discards remain.
- Trash Talk: +3 Chips per Ante after first discard.
- Rage Meter: +0.5 Mult per discard used per Ante.
- Redline: first discard in a Boss Blind gives $4.
- Empty Chamber: +10 Chips per Ante when no discards remain.
- Scarlet Momentum: X1.40 under its existing trigger.
- No Takebacks: +40 Mult.
- Final Cut: X3 on the final-hand/no-discards condition.
- Scarlet Apocalypse: +120 Mult and X3 as the first-step balance target.

### Blue Deck

Identity: conserve and strategically spend hands.

- Reduce passive/unconditional XMult.
- Preserve meaningful first-hand and final-hand payoffs.
- Reward having a high remaining-hand count without making unused hands a generic permanent multiplier engine.
- Azure Infinity target: once per Blind, on the first qualifying hand with 4+ hands remaining, approximately +40 Chips per remaining hand and X2 Mult.

### Yellow Deck

Identity: money thresholds, investment, and economy.

- Preserve Emergency Fund as a low-money safety net.
- Compress the existing $50/$75/$100 multiplier stack.
- Reaching $100 should feel powerful but should not by itself erase scoring requirements.
- Golden Singularity target: approximately $15 on Boss victory plus X2.25 while at $100+.

### Green Deck

Identity: squeeze value from unused resources.

- Keep extra hand/discard utility where healthy.
- Reduce uncapped resource-derived scoring and economy snowballing.
- Planet Saver target: roughly +0.08 XMult bonus per unused resource, with cashout bonus capped around $6.
- The player should be rewarded for efficiency, not for stacking resource mods into exponential score.

### Black Deck

Identity: Joker-slot pressure with a deliberate wide-vs-lean choice.

- Resolve the contradiction between talents that reward owning many Jokers and talents that reward owning few.
- Wide branch: rewards filled Joker slots / larger boards.
- Lean branch: rewards empty Joker slots / restraint.
- Black Hole Warranty should no longer be a generic free X2 for <=3 Jokers.
- Abyssal Crown target: approximately +1 Joker slot, +10 Mult per owned Joker, and +0.12 XMult bonus per empty Joker slot.
- Both branches should remain viable instead of directly invalidating each other.

### Magic Deck

Identity: actively using consumables as "casting."

- Differentiate Magic from Zodiac.
- Track consumable types used during the current Blind.
- Tarot, Planet, and Spectral use can establish different temporary benefits.
- Impossible Act target: after using all three consumable types in one Blind, approximately +120 Chips, +40 Mult, and X2.25.
- The payoff should require active sequencing rather than passive hoarding.

### Nebula Deck

Identity: poker-hand leveling and specialization.

- Reduce large per-level values that become excessive at level 10-20.
- Keep level thresholds and specialization rewards.
- Heat Death target: at level 10+, approximately +25 Chips per level and X2.1.
- The deck should strongly reward a developed favorite hand without becoming an automatic late-game multiplier tower.

### Ghost Deck

Identity: Spectral use plus enhanced-card synergy.

- Reduce passive "holding one Spectral" XMult stacking.
- Add more value to having used a Spectral during the current Blind.
- Preserve enhanced-card interactions.
- Beyond the Veil target: after relevant Spectral setup and an all-enhanced five-card hand, approximately +30 Chips per enhanced card and X2.4.

### Abandoned Deck

Identity: numbered-card mastery rather than a nearly free "no face cards" check.

- Replace trivial no-face-card conditions with meaningful numbered-card requirements.
- Use conditions such as 4+ numbered cards or a full five-card numbered hand.
- Lord of the Flies target: five numbered cards grants approximately +80 Mult and X2.4.

### Checkered Deck

Identity: Heart/Spade specialization and Flush manipulation.

- Account for Flushes being much easier with only two suits.
- Reduce generic Flush XMult.
- Preserve distinct Heart/Spade branch rewards.
- Add/pay off mixed five-card Heart/Spade patterns where useful.
- Perfect Checkmate target: Flush grants approximately +60 Mult and X2.1.

### Zodiac Deck

Identity: held consumable diversity and alignment.

- Zodiac remains the passive/held-diversity counterpart to Magic's use-based system.
- Two held consumable types should be practical and rewarding.
- All three types remain the premium condition.
- Grand Zodiac target: approximately +60 Chips per represented held type and X2.25 when all three are represented.

### Painted Deck

Identity: hand-size pressure and intentionally constrained Joker space.

- Do not immediately erase the deck's Joker-slot weakness with an early generic slot grant.
- Gallery Expansion should reward playing within the small gallery instead of simply adding a slot.
- A later Extra Frame may still grant +1 Joker slot as a meaningful investment.
- Shift more scoring toward cards left in hand / reserve size.
- Magnum Opus target: difficult five-card/large-reserve condition granting roughly +12 Mult per remaining card and X2.25.

### Anaglyph Deck

Identity: Tag generation, hoarding, and delayed payoff.

- Reduce overlapping Double Tag generation.
- Keep the fun of building a Tag stockpile.
- Prevent the same tree from both printing large quantities of Tags and multiplying them exponentially.
- Printing Press target: +1 Double Tag on Boss victory; each held Tag contributes approximately +0.10 XMult bonus, capped at 8 Tags.

### Plasma Deck

Identity: exploit Plasma's Chips/Mult balancing mechanic, not generic multipliers.

- Major redesign.
- Remove the chain of unconditional X1.3 -> X1.8 -> X1.4 -> X3 style multipliers.
- Branches should emphasize Chip bias, Mult bias, and balanced paired additive bonuses.
- Let Plasma's native balancing convert additive bonuses into value naturally.
- Late conditional XMult is allowed, but should be limited and identity-specific.
- Star Core target: large balanced additive bonus plus approximately X1.8 under a meaningful late-game condition, rather than a permanent +500/+100/X3 style effect.

### Erratic Deck

Identity: true volatility with fair expected value.

- Random XMult must be allowed to miss or roll low; randomness should not mean "always positive, sometimes huge."
- High-roll outcomes can be stronger because low/no-payoff outcomes exist.
- Absolute Pandemonium should no longer stack random Chips + random Mult + X3-X6 on every hand.
- Target: each hand rolls one major effect, approximately one of +240 Chips, +48 Mult, or X2.5.
- Preserve a modest Boss-related chance for a bonus Talent Point if it remains stable and non-abusable.

## Inspector and Math Presentation

The talent inspector must show enough information for players to understand live power without manual arithmetic.

For scalable effects, prefer fields conceptually equivalent to:

- Effect: +2 Mult per Ante after first discard.
- Current value: +10 Mult at Ante 5.
- Ante 8 reference: +16 Mult.
- Trigger: after first discard this round.

For conditional XMult, show the condition explicitly rather than embedding it in flavor text.

Descriptions, live previews, gameplay logic, README/changelog entries, and any legendary/global talent documentation must agree on values and triggers.

## State and Data Requirements

- Existing per-profile progression from v1.7.14+ must remain intact.
- Balance changes must not reset purchased talents merely because values/triggers change.
- If a talent ID is retained through a redesign, existing unlock ownership should remain valid.
- If a talent must be replaced with a new ID, provide an explicit migration from old ID to new ID.
- Ante Scaling preference and profile-specific UI state must continue to be stored per Balatro profile.

## Implementation Boundaries

Likely touched areas include:

- Talent definition/value tables.
- Trigger/effect application code.
- Per-Blind state tracking for newly use-based mechanics.
- Direct HUD draw/input code.
- Inspector model construction/caching.
- Documentation and changelog metadata.

Avoid unrelated refactors unless a current code boundary prevents safe implementation.

## Testing Strategy

### Static validation

- Parse every Lua file with an available Lua parser/compiler.
- Run Luacheck where practical, treating existing intentional globals/framework patterns separately from newly introduced warnings.
- Validate JSON metadata.
- Validate ZIP integrity for release package.

### Persistence regression

- Existing profile progression loads unchanged.
- Switching Balatro profiles rebinds the correct progression.
- Profile reset still affects only the active profile.

### Talent behavior tests

For each changed talent, verify:

- Trigger fires only when intended.
- Once-per-Blind / first-action restrictions reset correctly.
- Consumable-use state resets correctly between Blinds.
- Caps are enforced.
- Boss-only effects do not leak into normal Blinds.
- XMult and additive bonuses are applied in the intended phase/order.
- Disabled talents do not apply effects.

### UI regression

- Selecting stars does not rebuild the full overlay.
- Repeated selection does not reintroduce cumulative lag.
- Pan/zoom remains responsive on Android.
- Touch hitboxes match the visible controls.
- Inspector values update when Ante/state changes.
- No new per-frame unbounded allocation loop is introduced.

### Balance sanity checks

- Compare expected Ante-8 output of each tree against the v1.7.15 Red tree benchmark.
- Check interactions with the base deck mechanic rather than only raw tooltip values.
- Test high-synergy edge cases such as extra hands/discards, many Joker slots, high hand levels, large Tag stocks, large consumable inventories, and modded decks where feasible.

## Release Deliverables

- Updated Deck Talents mod package.
- Updated version metadata.
- Updated talent text and live inspector math.
- Updated README / relevant talent docs.
- Changelog describing UI and balance changes.
- Validation results recorded before release.
- Optional Linear issues/checklist for implementation and playtest follow-up.

## Success Criteria

The pass is successful when:

1. The Talent Universe is visibly more polished and easier to read while retaining the direct-draw no-lag interaction model.
2. No remaining deck relies on a stack of easy unconditional XMult talents as its primary identity.
3. Every deck has a clear gameplay fantasy that affects decisions during a run.
4. Capstones feel exciting but require a meaningful condition or setup.
5. Tooltip math, live values, and gameplay behavior agree.
6. Per-profile permanent progression remains compatible with existing v1.7.14+ saves.
7. Repeated Talent Universe interaction remains free of the prior cumulative lag regression.
