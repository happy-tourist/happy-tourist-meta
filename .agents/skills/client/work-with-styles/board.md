# Tourist board CSS (`GamePage`)

Read with the [core styles skill](SKILL.md). Behavior: `work-with-game-board`.

## Tourist board (`GamePage.vue`)

Board UI is custom CSS Grid (not Quasar widgets). Keep selectors local and class-driven:

| Class | Role |
|-------|------|
| `.game-page` | Column page: board scroll region (top presence + board) + sticky seated HUD |
| `.presence-row--top` | Opponents (seated) or all occupied (spectator) above the board |
| `.end-turn-affordance` | Icon-only `skip_next` **right-center** on own avatar (`right: -8px`, 36px hit) — **not** in budgets row, **no** labeled dock (SC-PRESENCE-17/25) |
| `.presence-slot--own` | Own HUD cluster: avatar | budgets; **`gap: 20px`** (≥ end-turn hit so `skip_next` does not cover budgets — SC-PRESENCE-25) |
| `.game-hud` / `__scroll` / `__bar--seated` | Sticky bottom **seated** panel only (`container-name: game-hud`); scroll wrapper owns `overflow-x` so say chrome is not clipped (SC-SAY-15); spectator has **no** bottom HUD |
| `.tourist-board` | 10×10 CSS Grid; `--cell` / `--gap` / `--radius`; `aspect-ratio: 1`; transparent holes |
| `.grille-overlay` / `--drop` / `--rise` | Revealed holding grille from `src/assets/grilles/grille.png`; duration via `--grille-anim-ms` (`GRILLE_ANIM_MS = 1000`) for drop/rise incl. leave-clear (SC-BOARD-18/19/21) |
| `.catapult-overlay` / `--reveal` / `--intact` / `--broken-hold` / `--vanish` | Catapult presentation from `src/assets/catapults/catapult.png` (+ `catapult-broken.png`); successful fade via `--catapult-anim-ms` (`CATAPULT_ANIM_MS = 1000`); broken timeline uses intact → hold → broken-hold → vanish classes; land→overlay→fling sequencing is page logic (SC-BOARD-22…28) |
| `.piece--trapped` | Trapped tourist still visible under grille (SC-PIECE-27); no own-select chrome |
| `.tile` | Rounded tile (`border-radius: var(--radius)`); `pointer-events` only when interactive |
| `.tile-start` | Green start tile (`#4caf50`) |
| `.tile-task` | Brown task tile (`#8d6e63`) |
| `.tile-center` | Yellow center (`#ffeb3b`); one element with `span 2` / `span 2` |
| `.tile--selected` / `.tile--target` | Local white / red move **and** return-mode ring chrome (same red class — SC-FINISH-13 / SC-MOVE-12) |
| `.tile-task.tile--removed` | Removed-task hole = page background; may keep `.tile--selected` while piece stands; **never** combine with `.tile--target` (holes not landable — SC-BOARD-15) |
| `.piece` | Absolute `left`/`top` from `--pcol`/`--prow` + `--cell`; ~250ms transition (incl. return from nearest center) |
| `.presence-marker` | Occupied seat chrome; reserved **96×96** `position: relative` slot for dual rings + 72px avatar (stable layout — SC-PRESENCE-11/13) |
| `.presence-progress--outer` | Turn ring (96px); `position: absolute; inset: 0; z-index: 0`; `pointer-events: none` |
| `.presence-progress--inner` | Reconnect ring (84px); absolute centered; `z-index: 1`; `pointer-events: none` |
| `.presence-avatar` | **Sibling** tourist PNG (**72px**) on top of rings — `z-index: 2`; `pointer-events: none`; no static `--turn` box-shadow; do **not** rely on progress default slot without `show-value` (SC-PRESENCE-12) |
| `.presence-slot` | Marker wrapper in top row or seated HUD |
| `.my-tourist-strip` / `.my-tourist-slots` / `.my-tourist-slot` / `-img` / `-finish-icon` | Four strip slots in HUD: wide flex row N,E,W,S; `@container game-hud (max-width: 420px)` → 2×2 smaller slots when own+row would overflow (covers ≤320/300 — SC-PIECE-09/32). **No** chip / `q-menu` / `-return-btn` |
| `.my-tourist-slot--dimmed` / `--returnable` / `--returning` | Dim finished **only** when `!canReturn`; returnable / returning chrome (SC-FINISH-09/13) |
| `.my-tourist-chrome-grille` / `--drop` / `--rise` | Grille overlay on strip slots when `trapped`; same `--grille-anim-ms` / `GRILLE_ANIM_MS = 1000` as board (SC-PIECE-31) |
| `.presence-place-badge` / `.ready-affordance` | Top-left corners (SC-PRESENCE-14) |
| `.presence-budgets` / `.budget-counter` / `.budget-fall` | Own steps/peeks **beside** own avatar (SC-PRESENCE-15…16); `.budget-fall` ≈ **2 s** (keep CSS in sync with `BUDGET_FALL_MS` / SC-PRESENCE-20); not on opponents; end-turn is **not** here |
| `.peek-affordance` / `.rescue-affordance` / `.push-affordance` | Top-center above piece (same family as `.return-affordance`; SC-BOARD-13 / SC-MOVE-74/77) |
| `.say-affordance` | Own marker top-right, raised (`top: -32px`) so hit-area does not overlap focus (SC-SAY-07 / SC-SAY-15 / SC-PRESENCE-33); hit-area ≥ ~32 CSS px (glyph may be smaller); z-index above rings/avatar |
| `.focus-affordance` | Own marker between say and end-turn (`top: 32%`, `translateY(-50%)`, `right: -8px`, 36px hit — SC-PRESENCE-30/33); see `work-with-game-board/focus.md` |
| `.say-bubble` / `.say-picker` | Presence comic bubbles + picker; chrome follows Dark via `body.body--dark` overrides (not tile fills) |
| `.say-bubbles--top` | Top-row markers: bubbles grow **down** toward the board |
| `.say-bubbles--bottom` | Own bottom marker: bubbles grow **up** toward the board |

Current-turn interactivity, dual turn/reconnect rings, top presence + seated sticky `.game-hud` (own + strip), own budgets beside avatar / end-turn icon right-center, peek/rescue/push top-center, removed-task holes, grille overlays + catapult sequential presentation CSS + strip chrome grille + return-icon, say top↓/own↑, nearest-center finish click, and travel/return/push-finish animation live in `work-with-game-board` — do not reintroduce draughts `.cell` / selection classes, all-markers-bottom HUD, compact chip / `q-menu`, page-local `.game-header`, orange-only return targets, return confirm modal, `.end-turn-dock`, `GRILLE_ANIM_MS = 1500`, or a static blue turn outline.

When editing board visuals:

- Prefer adjusting existing tourist classes over new global CSS.
- Preserve max tile 60px, gap 2, radius 2; holes show page background; do not style `.tile--removed` as a red landing target.
- Do not replace the board with Quasar grid components unless explicitly asked.
- Do **not** retune tile fills for app Dark mode — chrome theme must not
  change the tourist board look.
- Say bubbles/picker **may** use `body.body--dark` overrides (readable chrome);
  keep them comic bubbles near presence, not Quasar Notify toasts.
- See `work-with-game-board` for layout constant / center span / say rules.
