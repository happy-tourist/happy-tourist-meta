# Tourist board CSS (`GamePage`)

Read with the [core styles skill](SKILL.md). Behavior: `work-with-game-board`
(+ `knowledge.md` / `focus.md` / `peek.md`).

## Tourist board (`GamePage.vue`)

Board UI is custom CSS Grid (not Quasar widgets). Keep selectors local and class-driven:

| Class | Role |
|-------|------|
| `.game-page` | Column page: board scroll region (top presence + board) + sticky seated HUD |
| `.presence-row--top` | Opponents (seated) or all occupied (spectator) above the board |
| `.presence-slot--own` | Own HUD cluster: avatar \| knowledge; gap so **3** focus does not cover counter (SC-PRESENCE-25/50) |
| `.game-hud` / `__scroll` / `__bar--seated` | Sticky bottom **seated** panel (`container-name: game-hud`); translucent underlay α **0.78** (SC-PRESENCE-34); spectator has **no** bottom HUD |
| `.tourist-board` | 10×10 CSS Grid; `--cell` / `--gap: 2px`; `aspect-ratio: 1`; transparent holes; playable tiles `border-radius: 0` |
| `.grille-overlay` / `--drop` / `--rise` | Holding grille; `--grille-anim-ms` (`GRILLE_ANIM_MS = 1000`); **`filter: drop-shadow(...)`** (SC-BOARD-58) |
| `.catapult-overlay` / `--reveal` / `--intact` / `--broken-hold` / `--vanish` | Catapult presentation; `--catapult-anim-ms` (`CATAPULT_ANIM_MS = 1500`); broken vanish 300; drop-shadow SC-BOARD-58 |
| `.piece--trapped` | Trapped tourist still visible under grille |
| `.piece-badge--knowledge` / grille remaining | Yellow BR carried knowledge (own); red TR `remainingTurns` (all clients) — SC-BOARD-85 |
| `.tile` / `.tile-start` / `.tile-task` / `.tile-center` | Playable cells via `lib/tileSkins`; one finish tile per grid `7` (no span merge) |
| `.tile--selected` / `.tile--target` | Local white / red move **and** return-mode rings |
| `.tile-task.tile--removed` | Removed-task hole = page background; landable |
| `.piece` | Absolute from `--pcol`/`--prow`; ~250ms transition |
| `.presence-marker` / dual rings / `.presence-avatar` | 96×96 slot; outer turn / inner reconnect; 72px avatar sibling (SC-PRESENCE-11…13) |
| `.my-tourist-strip` / slots | Wide row; `@container game-hud (max-width: 420px)` → 2×2; **no** chip/`q-menu` |
| `.my-tourist-chrome-grille` | Strip grille when trapped; same anim + drop-shadow |
| `.presence-knowledge` / `.knowledge-counter` / `.budget-fall` | Own delivered knowledge beside avatar (SC-PRESENCE-50/51); reuse `.budget-fall` ~2s for +N; **not** steps/peeks |
| `.board-action-icon` | Shared **36×36** glyph for rescue/return/say/focus `<img>`; flag stays ~18 |
| `.rescue-affordance` / `.return-affordance` | Above sprite then **+20px down**; illustrated `rescue.png` / `return.png`; **no** peek/push affordance classes in product |
| `.say-affordance` | Own marker **12** — circumference hit-center |
| `.focus-affordance` | Own marker **3** (former end-turn slot); illustrated `focus.png` — SC-PRESENCE-53/55 |
| `.say-bubble` / `.say-picker` / `--top` / `--bottom` | Comic bubbles; picker shifted right (SC-SAY-15) |

Retired (do not revive): `.presence-budgets` steps/peeks, `.end-turn-affordance`, `.peek-affordance`, `.push-affordance`, `.presence-place-badge` place UX, focus under counters.

Behavior details live in `work-with-game-board` — do not reintroduce draughts cells, end-turn @3, focus under knowledge, density-era HUD, or finish `span 2×2` merge.

When editing board visuals:

- Prefer adjusting existing tourist classes over new global CSS.
- Preserve max tile 60px, gap 2, `border-radius: 0` on playable tiles.
- Shared map previews reuse `tileSkinCssVars()` / `TILE_SKIN_CSS_VARS` — see SC-MAP-54/71/72.
- Do not retune tile skins for app Dark mode; say chrome may use `body.body--dark`.
