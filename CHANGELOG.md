# Changelog

## 1.6.3 — Full-Canvas Web

- Rebuilt the Talent Web as a full-screen camera layer.
- Moved web controls into a fixed floating palette on the far-left side.
- Reoriented dependency progression left-to-right.
- Connector sprites now form the visible progression paths between talent columns.
- Improved talent node/icon readability.
- Mouse-wheel zoom and left-click drag panning remain the primary navigation controls.
- Kept Adaptive Power, individual talent toggles, Talent Shop, Omni Number, scoring values, costs, and saves unchanged.

## 1.6.2 — Fullscreen Web Renderer Hotfix

- Replaced the broken split-pane renderer with a screen-filling Talent Web.
- Removed the permanent right-side inspector.
- Talent details return to the proven separate card view.
- Added direct mouse-wheel zoom and left-click drag panning.
- Rebuilt Deck and Global layouts around dependency-aware lanes and connector sprites.

## 1.6.1 — Adaptive Power Hotfix

- Fixed run-start/run-delete crashes caused by invalid custom UIBox instance types.
- Fixed fullscreen previous/next deck navigation.
- Fixed Legacy View page navigation.
- Restored Deck, Global, and Global Cosmetic talent enable/disable behavior.
- Left Ante Scaling behavior unchanged.

## 1.6.0 — Adaptive Power

- Added optional Ante scaling for Deck and Global scoring talents.
- Existing scoring values are treated as Ante 8 reference strength.
- XMult scales only the bonus above X1.
- Talents can scale simultaneously from Ante and their original condition/resource.
- Added persistent Ante Scaling ON/OFF configuration.
- Added per-talent enable/disable controls without removing ownership, prerequisites, or mastery.
- Existing saves migrate with previously unlocked talents enabled.
- Updated Live Power to report Ante scaling and disabled talents.

## 1.5.0 — Fullscreen Tree

- Introduced dedicated fullscreen Deck and Global skill-tree pages.
- Added zooming and panning.
- Added in-tree talent inspector behavior.
- Preserved Talent Shop, mastery, art, and save compatibility.

## 1.4.0 — Pathways

- Replaced the default page-by-page browser with a pannable node-map experiment.
- Added clickable talent nodes and multiple zoom levels.
- Added Global Talent map support.
- Kept Legacy View as a fallback.

## 1.3.3 — Polished

- Low-risk readability pass on the stable page-based UI.
- Enlarged connector graphics slightly.
- Improved page labels, PREVIEW/CURRENT wording, prerequisites, and Legendary presentation.
- Corrected several talent icon assignments.
- Added additional mastery dialogue and a small QA easter egg.

## 1.3.2

- Fixed effect-description containment.
- Restored artwork proportions.
- Reduced edge shimmer/texture bleed.
- Preserved Mastery, Live Power, custom assets, balance, and saves.

## 1.3.0 — Mastered

- Added Deck and Global mastery/progress tracking.
- Added Live Power readouts.
- Added clearer prerequisite states and stronger Legendary presentation.
- Added mastery celebrations and QA easter eggs.

## 1.2.x — Art & Talent Shop UI

- Added deck-specific talent backgrounds, Global background, and Talent Shop background.
- Added dedicated Talent Shop offer icons.
- Added custom Talent Point token art.
- Stabilized Talent Shop vertical alignment.

## 1.1.x — Stability & Balance

- Added transactional config saves and rollback.
- Added config sanitization and talent lookup caches.
- Improved prerequisite and shortage messaging.
- Improved Boss/skip reward reliability.
- Added responsive UI behavior.
- Improved Talent Shop compatibility.
- Fixed several menu callbacks and layout regressions.
- Rebalanced Green Deck and Blue Deck's Azure Infinity.

## 1.0.0 — Talent Shop

- Added Talent Points from skipped Small/Big Blinds.
- Added the Talent Shop with 11 offers.
- Added Royal Forge, Numerical Paradox, Omni Number, Talent Battery, Vault Expansion, and Bossbreaker Sigil.

## 0.9.0 — Animated Talent Point Reward

- Added the custom eight-frame Talent Point acquisition animation.

## 0.8.0 — Talent Art

- Added talent-tree backgrounds and learned/unlearned talent artwork.

## 0.7.0 — Live Sprite Integration

- Integrated deck emblems, talent type/state icons, menu icons, and Talentless artwork throughout the UI.

## 0.6.0 — Sprite Sheets

- Added original deck emblem and UI symbol atlases.

## 0.5.0 — Graphical Overhaul

- Introduced the neon-fantasy visual theme and redesigned talent UI.

## 0.4.0 — Talentless Deck

- Added the Talentless Deck, which copies one random vanilla deck while suppressing permanent talent effects.

## 0.3.0 — Legendary Deck Expansion

- Expanded every vanilla deck from 4 to 15 talents.
- Added one Legendary capstone per deck.
- Increased the project to 225 Deck Talents plus 30 Global Talents.

## 0.2.0 — Global Talents

- Added 30 Global Talents and shared progression.

## 0.1.0 — Initial Release

- Added permanent talent trees to all 15 vanilla decks.
- Added shared Talent Points from Boss Blind victories.
