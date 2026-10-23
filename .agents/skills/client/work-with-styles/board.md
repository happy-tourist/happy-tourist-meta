# Tourist board CSS (`GamePage`)

Read with the [core styles skill](SKILL.md). Behavior: `work-with-game-board`.

## Tourist board (`GamePage.vue`)

Board UI is custom CSS Grid (not Quasar widgets). Keep selectors local and class-driven:

| Class | Role |
|-------|------|
| `.game-page` | Column page: board scroll region (top presence + board) + sticky seated HUD |
| `.presence-row--top` | Opponents (seated) or all occupied (spectator) above the board |
| `.presence-slot--own` | Own HUD cluster: avatar | budgets; gap ≥ end-turn half-hit past rim (~30px+) so **3** icon does not cover budgets (SC-PRESENCE-25) |
| `.game-hud` / `__scroll` / `__bar--seated` | Sticky bottom **seated** panel only (`container-name: game-hud`); **translucent underlay** light `rgba(255,255,255,0.78)` / dark `rgba(18,18,18,0.78)` — not opaque `#fff` / `#121212` (SC-PRESENCE-34 / D11; same alpha as SC-PACK-285); top `.presence-row--top` unchanged; scroll wrapper owns `overflow-x` so say chrome is not clipped (SC-SAY-15); spectator has **no** bottom HUD |
| `.tourist-board` | 10×10 CSS Grid; `--cell` / `--gap: 2px`; `aspect-ratio: 1`; transparent holes; playable tiles `border-radius: 0` (skin silhouette, not `--radius: 2`) |
| `.grille-overlay` / `--drop` / `--rise` | Revealed holding grille from `src/assets/grilles/grille.png`; duration via `--grille-anim-ms` (`GRILLE_ANIM_MS = 1000`) for drop/rise incl. leave-clear; **`filter: drop-shadow(...)`** edge contrast (SC-BOARD-18/19/21/58 — no opaque underlay) |
| `.catapult-overlay` / `--reveal` / `--intact` / `--broken-hold` / `--vanish` | Catapult presentation from `src/assets/catapults/catapult.png` (+ `catapult-broken.png`); successful fade via `--catapult-anim-ms` (`CATAPULT_ANIM_MS = 1500`); broken vanish via `--catapult-broken-vanish-ms` (300); **`filter: drop-shadow(...)`** (SC-BOARD-22…28/58); land→overlay→fling sequencing is page logic |
| `.piece--trapped` | Trapped tourist still visible under grille (SC-PIECE-27); no own-select chrome |
| `.tile` | Playable cell (`border-radius: 0`); `pointer-events` only when interactive; skin via `background-image` (`background-size: 100% 100%`) |
| `.tile-start` | Start skin via shared `src/lib/tileSkins.ts` → `src/assets/tiles/start.png` |
| `.tile-task` | Unbound task skin via `tileSkins` → `tile.png` |
| `.tile-task.tile--flipped` | Flipped task skin via `tileSkins` → `tile-back.png` + difficulty digit ink `#212121` (no light text-shadow) |
| `.tile-center` | Finish skin via `tileSkins` → `finish.png`; **one tile per** grid `7` cell (`kind: 'center'` — do **not** merge contiguous finish into a multi-cell `span`) |
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
| `.my-tourist-chrome-grille` / `--drop` / `--rise` | Grille overlay on strip slots when `trapped`; same `--grille-anim-ms` / `GRILLE_ANIM_MS = 1000` + **`filter: drop-shadow(...)`** as board (SC-PIECE-31 / SC-BOARD-58) |
| `.presence-place-badge` / `.ready-affordance` | Top-left corners (SC-PRESENCE-14) |
| `.presence-budgets` / `.budget-counter` / `.budget-fall` | Own steps/peeks **beside** own avatar (SC-PRESENCE-15…16); `.budget-fall` ≈ **2 s** (keep CSS in sync with `BUDGET_FALL_MS` / SC-PRESENCE-20); not on opponents; focus is **not** here (under end-turn on marker); end-turn is **not** here |
| `.board-action-icon` | Shared **36×36** `object-fit: contain` glyph filling the transparent affordance hit (`pointer-events: none`); used by peek/rescue/push/return/say/focus/end-turn `<img>`; all those hits **36×36**; **flag** stays `.my-tourist-finish-icon` ~18 (unchanged) |
| `.peek-affordance` / `.rescue-affordance` / `.push-affordance` / `.return-affordance` | Above sprite then **+20px down**: `transform: translate(-50%, calc(-100% + 20px))` (`top: … - 8px` / strip `top: -8px`); illustrated `src/assets/board/{peek,rescue,push,return}.png` as `<img class="board-action-icon">`; transparent hit **36×36**; **no** solid colored circular fill / medallion `box-shadow` (SC-BOARD-13/71 / SC-MOVE-74/77 / SC-FINISH-13); **flag** size/placement unchanged |
| `.say-affordance` | Own marker **12** — hit-center on circumference: `left:50%; top:0; transform: translate(-50%,-50%)`; illustrated `say.png`; SC-SAY-07 / SC-SAY-15 / SC-PRESENCE-14/33; hit ~56–64 (~60); transparent chrome; z-index above rings/avatar |
| `.focus-affordance` | **Under peeks** in `.budget-counters` (relative, ~60 hit); **not** on marker @9; SC-PRESENCE-30/33/37; illustrated `focus.png`; see `work-with-game-board/focus.md` |
| `.end-turn-affordance` | Own marker **3** — `left:100%; top:50%; transform: translate(-50%,-50%)`; hit ~56–64 (~60); SC-PRESENCE-17/25/33; illustrated `end-turn.png`; gap to budgets ≥ half-hit past rim |
| `.say-bubble` / `.say-picker` | Presence comic bubbles + picker; picker shifted **right** (`left: 50%`; no centering transform) so left edge of presets is not clipped by HUD overflow (SC-SAY-15); chrome follows Dark via `body.body--dark` overrides (not tile fills) |
| `.say-bubbles--top` | Top-row markers: bubbles grow **down** toward the board |
| `.say-bubbles--bottom` | Own bottom marker: bubbles grow **up** toward the board |

Current-turn interactivity, dual turn/reconnect rings, top presence + seated sticky `.game-hud` (own + strip), own budgets beside avatar / end-turn icon @3 / focus under end-turn at avatar square corner, peek/rescue/push/return clearance above sprite, removed-task holes, grille overlays + catapult sequential presentation CSS + strip chrome grille + return-icon, say top↓/own↑ + say-picker right shift, **per-cell** finish click/outline (SC-MOVE-65), and travel/return/push-finish animation (`FINISH_TRAVEL_MS` ~550 / `FINISH_FADE_MS` ~300) live in `work-with-game-board` — do not reintroduce draughts `.cell` / selection classes, all-markers-bottom HUD, compact chip / `q-menu`, page-local `.game-header`, orange-only return targets, return confirm modal, `.end-turn-dock`, focus under peeks / focus@9 / HUD `padding-left` for focus, `GRILLE_ANIM_MS = 1500`, a static blue turn outline, flat hex tile fills, finish `span 2×2` merge, or nearest-center finish remapping.

When editing board visuals:

- Prefer adjusting existing tourist classes over new global CSS.
- Preserve max tile 60px, gap 2, `border-radius: 0` on playable tiles; holes show page background; do not style `.tile--removed` as a red landing target.
- Shared map previews / editor palette reuse the same kind PNGs via `tileSkinCssVars()` / `TILE_SKIN_CSS_VARS` from `src/lib/tileSkins.ts` (`MapGridPreview`, palette faces also `border-radius: 0`) — see `content/maps` SC-MAP-54/71/72. Do **not** copy divergent `url(...)` strings into page CSS.
- Do not replace the board with Quasar grid components unless explicitly asked.
- Do **not** retune tile skins for app Dark mode — chrome theme must not
  change the tourist board look.
- Say bubbles/picker **may** use `body.body--dark` overrides (readable chrome);
  keep them comic bubbles near presence, not Quasar Notify toasts.
- See `work-with-game-board` for layout constants / per-cell finish / say rules.
