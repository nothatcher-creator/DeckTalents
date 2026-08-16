# Deck Talents v1.6.3 — Full-Canvas Web

v1.6.3 focuses on the Talent Web renderer while deliberately leaving the working Adaptive Power, talent activation, Talent Shop, Omni Number, scoring values, costs, and save data alone.

## Talent Web

- Rebuilt the talent web as a full-screen camera-style view.
- Moved web controls into a fixed floating palette on the far-left side.
- The web is intended to pan and zoom underneath the fixed controls.
- Reoriented dependency progression left-to-right.
- Connector sprites occupy the spaces between dependency columns and form the visible progression web.
- Increased talent node/icon readability for full-screen browsing.
- Mouse wheel controls zoom.
- Left-click drag controls panning.
- Reset View restores the default camera position.
- Clicking a talent opens the detailed talent card with effect, requirements, cost, Unlock, and Enable/Disable controls.
- Legacy View remains available as a fallback.

## Adaptive Power

Ante Scaling remains unchanged from v1.6.0/v1.6.1. Scoring talents use Ante 8 as their reference strength, while XMult scales only the bonus above X1.

Individual talent enable/disable controls remain separate from ownership: a disabled talent still counts for prerequisites and mastery but contributes no gameplay effect.

## Compatibility

Primary target:

- Balatro 1.0.1o-FULL
- Steamodded 1.0.0~BETA-1814a
- LÖVE 11.5.0
- Lovely 0.9.0

## Updating

Replace the old `DeckTalents` folder with the new packaged version. Existing permanent progression and activation settings should be preserved by the configuration migration system.

## QA priorities

The most useful testing for this release is:

1. Deck and Global Web visibility at multiple resolutions.
2. Mouse-wheel zoom.
3. Left-click drag panning.
4. Talent node selection.
5. Unlock and Enable/Disable behavior.
6. Ante Scaling at early, mid, Ante 8, and Endless values.
7. Starting and deleting runs after opening/closing the Talent Web.
