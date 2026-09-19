---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the tourist board UI in
  the happy-tourist client — GamePage CSS Grid (gap/radius 2), synced seat pieces
  overlay, top opponents / spectator presence + sticky bottom seated HUD (own +
  strip; no chip/q-menu), dual rings + top-center board affordances + budgets beside own
  avatar + end-turn `skip_next` right-center on own avatar, removed-task holes, grille overlays
  + trap/rescue/push + return strip icon (no modal), peek eye + Correct/Wrong modal, say bubbles
  (top markers down / own bottom up), strip row N,E,W,S or HUD ≤~420 → 2×2,
  nearest-center finish click + return anim, tile kinds (start/task/center),
  current-turn selection/hints/move submit, piece travel animation, or finish
  travel lastKnown seed (move/push→center; no finishAnimFrom at submit) for room
  tourist. Leave + match status live in App.vue header on Game route (not
  page-local).
---

# Work With Game Board

Use this skill for the **tourist board UI** on Game in the Vue 3 Quasar client (`happy-tourist.github.io`).

Product: настольная игра «Счастливый турист». On the seated client whose turn it is **and** `game.isPlaying` (`phase === 'playing'`), the board and the own strip accept piece selection, local move hints, move submit via `game.sendMove`, rescue via `game.sendRescue`, push via icons over targets → `game.sendPush`, return via green strip icon → ring targets → `game.sendReturnFromFinish`, and peek via eye → `game.sendPeek` / `sendPeekAnswer`. Before playing: no move chrome / submit. Spectators and non-current seated players remain non-interactive for moves. Authority for legality stays on the server (`work-with-game`). Preset say bubbles are separate from turn — any seated+online player may send via `game.sendSay` (server whitelist). Ready-to-start uses `game.sendReady` on own marker **top-left** when `canSendReady` (say affordance stays **top-right**). Own private budgets sit **to the right of** the own avatar (steps/peeks stacked vertically); end-turn is icon-only `skip_next` **right-center** on the own avatar (`canSendEndTurn`) — **not** inside the budgets row and **not** a labeled dock. Shared App header on Game route owns leave + match status (`work-with-pages`); GamePage has no page-local leave header and no room-id chrome.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| App shell | `src/App.vue` | On Game route only: icon-only leave left + centered match status + theme right; leave confirm + `leaveGame` → lobby. Login/Lobby: theme only (no leave/status) |
| Game page | `src/pages/GamePage.vue` | CSS Grid field in scroll region; unfinished pieces overlay (+ short center disappear); holes for `removedTaskKeys`; grille overlays from `holdingGrilleKeys` (drop/rise); rescue + **push** affordances **top-center** above pieces; **top** `.presence-row--top` tight to board (opponents / spectator all; no reserved bubble gap); sticky bottom `.game-hud` only when seated (own + strip); budgets **beside** own avatar; end-turn `skip_next` **right-center** on own avatar; local selection/hints/peek eye top-center; place / timer-vs-steps end / peek / solo-peeks∞ / all-jail; **return green icon** on finished strip (no confirm `q-dialog`); `rejoinGame(roomId)` — **no** leave/status/roomId chrome |
| Game store | `src/stores/game.ts` | Room I/O; mirror `seats` (+ piece `finished`/`trapped` / seat `finishPlace`) / `phase` / `maxSeats` / `countdownRemaining` / `currentTurnSessionId` / `removedTaskKeys` / `holdingGrilleKeys` / `sessionId`; private `steps`/`peeks`/`budgetsInfinite` (peeks∞ only)/`peekedThisTurn` (legacy)/`openPeek`/`allJailWarning` from `budgets`/`peekOpen`/`allJailWarning`; `unfinishedBoardPieces` / `isMySeatFinished` / `myFinishedStripSides`; `isMyTurn` / `isPlaying` / `canSendReady` / `canSendEndTurn`; `sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendReady` / `sendSay` + `sayEvents` |

| Concern | Location |
| --- | --- |
| Layout constant | `LAYOUT` string grid in `GamePage.vue` (`.` hole, `1` start, `*` task, `7` center) |
| Tile build | `buildBoardTiles()` → `div.tile` with `gridColumn` / `gridRow`; removed `*` → visual hole (page background) |
| Pieces | `unfinishedBoardPieces` (+ short-lived disappearing finishers) → `img.piece`; PNG from `touristId`; `piece--trapped` when trapped |
| Grilles | Asset `src/assets/grilles/grille.png`; board overlay on `holdingGrilleKeys` **and** chrome grille on strip slots when `trapped`; drop/rise via **`GRILLE_ANIM_MS = 1000`** → CSS `--grille-anim-ms` for board + chrome (incl. leave-clear rise, SC-BOARD-18/19/21, SC-PIECE-31) |
| Presence | Seated: opponents in `.presence-row--top` above board; sticky bottom `.game-hud` = own + strip only. Spectator: all occupied in top row; **no** bottom HUD. Dual rings + 72px avatar; finish/ready top-left; say top-right; end-turn right-center |
| Say (game/say) | Affordance top-right on **own** online marker (hit-area ≥ ~32px); **top** markers → `.say-bubbles--top` (down toward board); **own bottom** → `.say-bubbles--bottom` (up toward board). HUD scroll overflow must not clip (SC-SAY-15) |
| Tourist strip | Inside HUD after own marker when `mySeat` (+ pieces exist): four `.my-tourist-slot` — wide: flex **row** N,E,W,S; `@container game-hud (max-width: 420px)`: **2×2** with smaller slots (when own+row would scroll; covers ≤320/300). **No** compact chip, **no** `q-menu`, **no** «Мои туристы» caption. Select unfinished non-trapped from slot or board; off-turn → view only. Finish flag on finished slots; grille overlay when trapped (SC-PIECE-09/31/32) |
| Move UX | Local `selectedSide` + `legalTargets` only when `isPlaying && isMyTurn && !isMySeatFinished && !moveAnimating`; **keep-focus** after non-finishing move (SC-MOVE-46); never select finished **or trapped** pieces; move does **not** end the turn |
| Rescue UX | Affordance **top-center** over own trapped when adj free own piece + steps≥1 → brief approach anim + `sendRescue(side)` (server rescuer coords unchanged) |
| Push UX | Icons **top-center** over **targets** (not pusher) when selected own free pusher + steps≥1 + legal far-side dest; click → approach/back + target travel + `sendPush(pusherSide, targetSessionId, targetSide, row, col)`; keep `selectedSide` for multi-push; eye stays on selected, push on targets (SC-MOVE-74/75); push→center: after successful `sendPush`, seed target `lastKnown` from pre-push cell (not `finishAnimFrom`) + same travel+disappear as move finish (SC-MOVE-76 / D10) |
| Return UX | Green circle + Material `undo` **top-center** over finished strip tourist when `canReturn(side)` → click sets `returningSide` (no confirm modal); ring cells use same `.tile--target` (red) as move. Strip slot **body** on finished → **noop** (does not start return — SC-FINISH-16). Dim (`.my-tourist-slot--dimmed`) **only** when finished ∧ `!canReturn`. Select another side / clear selection clears return-mode. Step only on server accept → `sendReturnFromFinish(side, row, col)`. Return travel anim: all clients slide from **nearest center** (Chebyshev; tie row, then col) to ring (`MOVE_ANIM_MS`) (SC-FINISH-09/13/15) |
| Peek UX | Eye **top-center** only on **selected** own **non-trapped** piece on present `*` (peeks remain / solo peeks∞; multi peek while peeks remain — no one-peek/turn gate); spent grille cell still peekable; keep-focus → eye without re-click (SC-BOARD-14); **no** ambient peekable tile chrome; modal from `game.openPeek` → Correct/Wrong → `sendPeekAnswer` |
| Finish UX | Center land (move **or** push) → slide from last board cell + fade (`disappearingKeys` / `finishAnimFromByKey`); own move seeds `lastKnownBoardCellByKey` at submit (D11 / SC-FINISH-18; finish watch owns `finishAnimFrom`); own `finishPlace` 0→N → place modal; stay in room after close. Click any point on visual finish 2×2 → submit **nearest legal center** vs selected piece (Chebyshev; no quadrant mapping — SC-MOVE-63) |
| All-jail | Own seat only: informational modal from `game.allJailWarning` |
| Center | One element with `span 2` / `span 2` (solid 2×2), not four cells |
| Room enter | Out of scope — see `work-with-lobby` / `work-with-rooms` (`rejoinGame`) |

## Layout And Tiles

- Grid: **10×10** sparse; empty corners are holes (no tile element; page background shows through).
- Kinds: `start` (green), `task` (brown), `center` (yellow).
- After **correct** peek only: synced `game.removedTaskKeys` (`"r,c"`) → task tile omitted (same hole chrome as empty corners); cell is **not** a legal landing target (red hints exclude it). A piece already on that cell **remains** (stand-on-hole OK — SC-BOARD-12). Incorrect / forced-incorrect KEEP leaves the brown tile and hidden reward (no hole).
- `.tourist-board`: `width: 100%`, `max-width: calc(10 * 60px + 9 * 2px)`, `aspect-ratio: 1`, `--gap: 2px`, `--radius: 2px`, `grid-template-*: repeat(10, 1fr)`; tile `border-radius: var(--radius)`. Do **not** size rows with `%` of auto height (tracks collapse to 0).
- Container: full width of Game content (no side presence gutters); mobile edge-to-edge relative to page content; wide screens capped by max tile 60px (via board max-width + square aspect).
- Tile colors are **fixed fills**, independent of Quasar Dark chrome (see `work-with-styles` / theme specs).

## Move Interaction (current turn only — D5 / SC-MOVE-11…13/46/63, SC-BOARD-05/06/14)

| Rule | Behavior |
|------|----------|
| Who interacts | Only seated **non-finished** client with `isPlaying && sessionId === currentTurnSessionId` and not mid-own-animation |
| Select | Click own **unfinished non-trapped** piece on board **or** matching strip slot → `selectedSide`; white outline on that cell. Changing select clears `returningSide` |
| Hints | Legal one-step neighbors (Chebyshev 1, playable LAYOUT cell **excluding** removed-task holes, unoccupied by unfinished incl. trapped) → red outline; **local only**; hidden while `steps === 0` (steps always finite, incl. solo). Return-mode uses the **same** `.tile--target` red chrome on legal ring cells |
| Reselect | May change `selectedSide` among own unfinished non-trapped pieces until submit |
| Submit | Activate red destination → `game.sendMove(side, row, col)`; **turn stays** (end via button / auto / timeout) |
| Center finish click | Any click on visual `tile.kind === 'center'` 2×2 → `nearestLegalCenterCell(selectedCell, legalTargets)` (Chebyshev; no quadrant map); no legal center → no-op (SC-MOVE-63) |
| Keep-focus | After successful **non-finishing** move: **do not** clear `selectedSide`. After move-anim ends, white frame + red targets (if steps>0) return from the new cell without re-select (SC-MOVE-46). Center finish → clear selection |
| Peek | Eye only when **selected** non-trapped piece stands on still-present `*` and peeks allow (solo peeks∞; multi while peeks>0 — no `peekedThisTurn` gate) → `sendPeek(side)`; after keep-focus, eye appears without re-click (SC-BOARD-14). Hole / non-`*` / trapped → no eye. **No** ambient highlight of other peekable cells |
| Rescue | Affordance over trapped when adj free + steps → `sendRescue`; lock move/peek for trapped |
| Push | Icons over free adj targets of selected free pusher + steps → `sendPush`; keep selection after push |
| Return | Green `undo` icon over finished strip when `canReturn` → return-mode + ring `.tile--target` → click → `sendReturnFromFinish`. Slot body finished → noop. No confirm modal |
| Clear selection | Piece finishes / traps; lose turn / not playing; time-expired / finished seat (same watchers as before); also clears return-mode |
| Finished | Finished pieces stay off the board after disappear; finished strip slots show finish icon and never select/submit as board pieces; dim only if `!canReturn`; strip keeps all four sides after full finish (SC-FINISH-09/10) |
| Others | Spectators / not-your-turn / not playing / finished seat: no selection, no red/white move chrome, no `sendMove` / peek / push |

Do **not** sync selection or hints — page-local refs only. Do **not** assume `sendMove` advances the turn. Do **not** reintroduce compact chip / `q-menu` picker (SC-PIECE-29/30 removed).

## Piece Travel Animation (D6 / SC-MOVE-14…15 / SC-FINISH-01/15/17/18 / D10–D11)

- Position pieces with CSS `transform` / absolute offsets from grid vars (`--prow` / `--pcol`), not tweening `grid-row`/`grid-column`.
- Duration ~200–300ms ease-out (`MOVE_ANIM_MS`); all clients animate when synced `row`/`col` changes.
- On submit: set `moveAnimating` and ignore board/strip clicks until timer ends; keep `selectedSide` unless the destination is center finish (SC-MOVE-46).
- Skip transition on first paint so pieces do not fly from origin.
- **Center finish (move or push):** sync may already place the piece on a center cell — paint one frame at `lastKnownBoardCellByKey` via `finishAnimFromByKey`, hold until painted (`nextTick` + forced layout + double `rAF`), then clear so CSS slides onto center + fade (`disappearingKeys` / `FINISH_FADE_MS`) (SC-MOVE-76 / SC-FINISH-01/17/18 / D10–D11). Track unfinished board cells with a sync-flush watcher (skip recording center coords; drop any stale `finishAnimFrom` for still-unfinished keys). **Own move onto center:** in `submitMove`, seed only `lastKnownBoardCellByKey` with the piece’s current cell **before** `sendMove` (do **not** pre-seed `finishAnimFromByKey` — silent reject would pin the piece). **Own push onto center:** after successful `sendPush`, seed target `lastKnown` from `push.targetRow/Col` the same way. Finish watch sets `finishAnimFrom` when finished arrives (SC-FINISH-18). Remote finish (other’s move/push) still uses the watch + lastKnown path.
- **Return-from-finish:** when a piece leaves the finished set, all clients animate from `nearestCenterCell(target)` (Chebyshev among `CENTER_CELLS`; tie row, then col) onto the ring — mirror of finish disappear (SC-FINISH-15).

## Presence (occupied seats — top row + seated bottom HUD)

Sync-driven markers (SC-PRESENCE-01…25 / redesign-game-hud phase 2). Page reads mirrored `game.seats` + own `steps`/`peeks`/`budgetsInfinite` only — no Colyseus I/O here (`peekedThisTurn` is legacy store mirror, unused by page). Match status strings live in `App.vue` header on Game (SC-PRESENCE-23), not on the page.

| Rule | Behavior |
|------|----------|
| Who | One marker per **occupied** seat (including offline-in-grace). Empty slots not rendered. |
| Avatar | Seat `touristId` → same tourist PNG as pieces; **image box = 72px** (SC-PRESENCE-13) |
| Seated viewer | **Top** `.presence-row--top`: opponents only, tight to board. Sticky bottom `.game-hud`: **own** marker (+ budgets beside avatar) → **strip**. **No** opponents in bottom bar; **no** chip/`q-menu` (SC-PRESENCE-02/22) |
| Spectator | All occupied seats → **top** row L→R by join order; **no** bottom presence HUD (SC-PRESENCE-03) |
| Join order | Array order from sync map `forEach` as mirrored into `seats[]` |
| Offline | `!connected && reconnectUntil > 0` → **inner** warning `QCircularProgress` (`min=0`, `max=30`, `value` = remaining from `reconnectUntil − now`) |
| Reserved chrome | Always outer ring slot sized to outer progress (96px around 72px avatar — SC-PRESENCE-11); inactive turn/grace → transparent track/value 0 (no size jump) |
| Turn deadline | Outer determinate ring while `playing` + current turn + synced `turnUntil`/`turnBudgetSeconds`: blue (`primary`) for multi 60s; red (`negative`) when `turnBudgetSeconds === 300` (solo). **No** static blue outline / `--turn` box-shadow |
| Dual rings | Offline current-turn: outer = turn, inner = reconnect (both visible — SC-PRESENCE-10). **Siblings** only (outer absolute behind) — do **not** nest `q-circular-progress`. **Avatar:** always sibling `<img class="presence-avatar">` on top (pre-timer pattern); do **not** put avatar only in progress default slot without `show-value` (Quasar omits that slot from the DOM — SC-PRESENCE-12) |
| Match status | App header centered: «Ваш ход» / «Ход соперника» / «Ход игрока» (spectator) + waiting/connecting/countdown/finished — **not** page chrome (SC-PRESENCE-23) |
| Room id | **Never** show `roomId` (or truncated id) in Game chrome (SC-PRESENCE-24) |
| Finish place | Seat `finishPlace > 0` → numeric badge at **top-left** of marker (SC-PRESENCE-14) |
| Ready | Own marker only, **top-left** when `canSendReady` (does not overlap finish by phase) |
| Say affordance | Own online marker only, **top-right** (SC-PRESENCE-14 / SC-SAY-07); never on opponents / spectators |
| Own budgets | Own seated marker only while `playing`: **vertical stack to the right of** avatar (`.presence-budgets`); steps (always number) + peeks (∞ when `budgetsInfinite` = solo peeks∞); never on opponents / spectators (SC-PRESENCE-15/16) |
| +N anim | Local fall animation when finite `steps`/`peeks` increase (multi grant +1/+1, solo become-current +1 step only, peek Correct reward); steps always; peeks skipped while peeks∞; duration ≈ **2 s** (`BUDGET_FALL_MS` + CSS) — page-local, no sync event (SC-PRESENCE-20 / SC-MOVE-50) |
| End-turn | Icon-only `skip_next` **right-center** on own avatar when `canSendEndTurn` → `sendEndTurn` immediately (no dialog / no visible label); **not** in budgets row; **no** `.end-turn-dock`; hide in solo; keep `.presence-slot--own` gap ≥ end-turn hit so icon does not cover budgets (SC-PRESENCE-17/18/25) |
| Solo peeks∞ modal | When `budgetsInfinite` flips false→true → peeks-unlimited / steps-finite modal (SC-PRESENCE-19); close keeps player in room |
| Solo end modals | Own `timeExpired` false→true → timer-expired **or** steps-exhausted copy (infer: steps=0 ∧ ¬on live `*` → steps; else timer) — SC-PRESENCE-21 / SC-MOVE-45/48 |

Tick `nowMs` on an interval (~200 ms) while Game is mounted so turn + reconnect rings animate from **server** `turnUntil` / `reconnectUntil`.

Finish flag + return-icon chrome live on the personal **strip** inside the seated HUD — presence only shows the **place** badge.

### Presence DOM (canonical — SC-PRESENCE-04/10/11/12/13)

Inside `.presence-marker` (96×96 = outer ring, `position: relative`), **three siblings** — never nest progress in progress, never put avatar only in progress default slot without `show-value`:

```html
<q-circular-progress class="presence-progress presence-progress--outer" size="96px" … />
<q-circular-progress class="presence-progress presence-progress--inner" size="84px" … />
<img class="presence-avatar" :src="touristSrc(touristId)" alt="" />
```

CSS: outer ring `position: absolute; inset: 0; z-index: 0`; inner ring absolute centered (`top/left: 50%`, `transform: translate(-50%, -50%)`, `z-index: 1`); avatar `position: relative; z-index: 2` (**72×72**). Rings and avatar use `pointer-events: none` so affordances / marker clicks pass through. Finish badge / ready (top-left), say (top-right), and end-turn (right-center) sit above with `z-index: 3+` (say/end-turn higher, `pointer-events: auto`). Say / end-turn hit-area ≥ ~32 CSS px (icon glyph may be smaller — SC-SAY-15 / SC-PRESENCE-17).

Layout shell: `q-page.game-page` column — **board scroll region** (flex; includes top presence + board) → sticky bottom `.game-hud` when seated (`position: sticky; bottom: 0`; `container-type` / `container-name: game-hud` for strip CQ). Inside HUD: `.game-hud__scroll` (`overflow-x: auto`, `overflow-y: visible`, `pointer-events: none`) → `.game-hud__bar--seated` (`overflow: visible`, `pointer-events: auto`, own marker + strip). Do **not** put `overflow-x: auto` on the same box as the markers (browsers force Y clip and hide say chrome — SC-SAY-15). Do **not** put all markers in the bottom bar, reintroduce chip/`q-menu`, labeled end-turn dock, or leave opponents in the HUD.

## Say Bubbles At Presence (game/say — SC-SAY-07…12/15)

Ephemeral preset phrases near presence markers. Protocol + Pinia I/O: `work-with-stores` / `colyseus-client` / server `work-with-messages`. UI stays on `GamePage` beside markers — **not** Quasar Notify / viewport toasts.

| Rule | Behavior |
|------|----------|
| Who may send | Seated + `connected` only; anytime (not turn-gated). Spectators / offline grace: no affordance |
| Affordance | Only on marker with `sessionId === game.sessionId` and connected (`canSendSay`) — **top-right** (`chat_bubble_outline`); hit-area ≥ ~32px; `pointer-events: auto` + z-index above rings/avatar; HUD overflow MUST NOT clip (SC-SAY-15) |
| Ready | **Top-left** on own marker when `canSendReady` → `game.sendReady()` (not `sendSay('ready')`); i18n `game.readyButton` |
| Countdown | Full-screen overlay while `phase === 'countdown'` from synced `countdownRemaining` (all clients) |
| Picker | Click affordance → two presets «Всем привет» (`hello`) / «Удачи» (`luck`) via i18n `game.say.*`; choose → `game.sendSay(presetId)` and close immediately; click away closes |
| Bubbles | From `game.sayEvents` filtered by sender `sessionId`; label via `$t('game.say.' + presetId)` (incl. `ready` → «Готов начать!») |
| TTL | Disappear after **10s** from server `at` (`SAY_TTL_MS`); prefer `at`, not local receive time |
| Max live | At most **3** per sender; store refuses 4th locally; server enforces the same |
| Stack | **Top** markers (opponents / spectator): `.say-bubbles--top` — grow **down** toward the board. **Own** bottom marker: `.say-bubbles--bottom` — grow **up** toward the board. Newer closer to avatar. **No** left/right side-slot stacks (SC-SAY-11/12) |
| Row gap | Horizontal gap between markers so concurrent neighbor bubbles do not overlap; horizontal scroll on `.game-hud__scroll` OK (bar stays `overflow: visible`) |
| I/O | Page never `room.send` / `room.onMessage` — only `sendSay` / `sendReady` + read `sayEvents` |

Picker open state (`sayPickerOpen`) is page-local; close if the local seat is lost or goes offline.

## Authority

- Board **geometry** (`LAYOUT`) is a client constant; server does not sync tile kinds (server mirrors playable set in `touristMove.ts`). Removed task holes come from synced `removedTaskKeys`.
- **Seats**, `phase` / `maxSeats` / `countdownRemaining`, connectivity, **`currentTurnSessionId`**, **`turnUntil` / `turnBudgetSeconds`**, **`removedTaskKeys`**, and seat **`timeExpired` / `finishPlace`** are server-authoritative; client mirrors and renders.
- Own **steps/peeks** are private room messages (`budgets` / `peekOpen`) — not schema; never show others’ counters.
- Local legal-target / eye affordance computation is a **hint** only — server rejects illegal moves/peeks.
- Say bubbles are **ephemeral room messages**, not schema state; server whitelist + live limit are authoritative.
- Do not reintroduce legacy draughts CellValue `0…4` / `{ from, to }` encoding.

## GamePage Responsibilities

- Render `boardTiles` from `LAYOUT`, omitting removed task keys as holes, in a scroll region above the HUD.
- Overlay **unfinished** pieces (plus short-lived disappearing finishers / return travelers) at `row`/`col` (0-based schema → CSS vars / transform).
- Render **top presence** (seated opponents / spectator all occupied) and, when seated, sticky bottom `.game-hud` (own + strip) — dual turn/reconnect rings + reserved 96px chrome; place badge top-left when `finishPlace > 0`.
- On own online marker: say affordance **top-right** + picker (finished / time-expired keep say); ready **top-left** when `canSendReady`; end-turn `skip_next` **right-center** when `canSendEndTurn` (immediate, no dialog); budgets **beside** avatar; for every marker: live say bubbles from `sayEvents` (TTL / direction per stack rule above).
- Full-screen countdown overlay while `phase === 'countdown'`.
- Show strip inside HUD when `mySeat` and pieces exist (**no** caption; **no** chip/`q-menu`); finish flag on finished slots; dim finished only if `!canReturn`; return via **green strip icon** (no confirm modal; slot body finished → noop); grille chrome on strip when trapped (`GRILLE_ANIM_MS = 1000`); wide row / HUD ≤~420 → 2×2; on own turn **in playing** select unfinished non-trapped from strip or board.
- Board overlay: unfinished pieces only when seats have pieces (none in waiting/countdown).
- On own turn in playing (not finished / not time-expired): white selection + red targets; center click → nearest legal center; submit via store `sendMove`; push icons over legal targets of selected pusher → `sendPush`; keep-focus after non-finishing move/push; peek eye (selected present `*` only — no ambient peekable chrome) + Correct/Wrong modal via `sendPeek` / `sendPeekAnswer`.
- Animate piece travel; on center finish (move or push) keep DOM key for slide from last board cell then fade (`FINISH_FADE_MS` / `finishAnimFromByKey`; own move/push seeds `lastKnown` only — D10/D11); clear selection; on return animate from nearest center; ignore input while `moveAnimating`.
- Own `finishPlace` 0→N → place `q-dialog` (`game.finishPlaceModal*`); own `timeExpired` false→true → dual end `q-dialog` (`game.timeExpiredModal*` vs `game.stepsExhaustedModal*`); solo peeks∞ modal when `budgetsInfinite` becomes true; **no** return-confirm dialog; close keeps player in room; clear selection on expiry.
- Budget +N fall ≈ 2 s (`BUDGET_FALL_MS` / `.budget-fall` CSS — SC-PRESENCE-20).
- Remount without room → `rejoinGame(roomId)` via store. Leave confirm + status + icon-only exit live in **`App.vue`** (`work-with-pages` / `work-with-rooms`) — do **not** reintroduce page-local leave header or room-id chrome.

## Do

- Keep layout in one client constant; center as a single 2×2 grid area.
- Preserve max tile 60px, gap 2, radius 2, hole = page background (incl. removed `*`).
- Keep Colyseus I/O in `stores/game`; page reads seats/phase/turn/strip/presence/`sayEvents`/`steps`/`peeks`/`budgetsInfinite`/`openPeek`/`removedTaskKeys`/`holdingGrilleKeys` from store only.
- Gate move interactivity with `isPlaying && isMyTurn && !isMySeatFinished && !isMySeatTimeExpired` (and `!moveAnimating`); never select finished pieces/slots as board movers; gate say with own seated+connected; ready via `sendReady`; end-turn via `canSendEndTurn` icon on own avatar.
- Map `touristId` 1…4 to `tourist{N}.png`; strip only after pieces exist; finish icon on finished sides; return via green strip icon + same red targets as move (no confirm modal).
- Drive turn/reconnect countdowns from synced `turnUntil` / `reconnectUntil`; countdown overlay from `countdownRemaining`.
- Expire bubbles from server `at` + `SAY_TTL_MS`; keep max 3 live per sender in UI; stack top markers down / own bottom up.
- Keep leave/status in App Game chrome; confirm only when seated ∧ playing ∧ `finishPlace === 0` ∧ `!timeExpired`.

## Don't

- Call `room.send` from the page — only `game.sendMove` / `sendPush` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendSay` / `sendReady` / `sendRescue` / `sendReturnFromFinish`.
- Show white/red move chrome before `playing`, to spectators, non-current players, finished seats, or finished pieces (except return-mode red ring targets).
- Show say send affordance or budget counters on other players’ markers or to spectators.
- Put **all** markers in the bottom HUD; reintroduce compact chip / `q-menu` / `touristChipAria` picker; put return **confirm modal**; use orange-only return targets; dim finished slots when `canReturn` is true; map finish 2×2 clicks by quadrant; start return from finished slot body click.
- Put end-turn inside the budgets row or reintroduce `.end-turn-dock` / labeled «Завершить ход» button — keep icon-only `skip_next` right-center on the own avatar.
- Assume a successful move ends the turn — use end-turn / auto / timeout.
- Clear `selectedSide` after every successful non-finishing move — keep-focus (SC-MOVE-46).
- Ambient-highlight other peekable cells — eye only on the selected piece (SC-BOARD-14).
- Pre-seed `finishAnimFromByKey` in `submitMove` / `onPushClick` — seed only `lastKnownBoardCellByKey`; finish watch owns `finishAnimFrom` (silent reject would pin via `pieceStyle` — D11 / SC-FINISH-18).
- Record a center cell into `lastKnownBoardCellByKey` (sync may land on center before `finished`; never use center as travel `from`).
- Use Quasar Notify / screen-edge toasts as the say carrier; force all bubbles “always above” with only one stack class.
- Depend tile fills on Quasar Dark / theme preference.
- Invent client-local seat assignment (server assigns on join).
- Treat client hints as authority.
- Sync selection / hints / say bubbles / steps-peeks to schema.
- Put lobby subscribe/create/join logic into the board skill — use `work-with-lobby`.
- Put reconnect token / `rejoinGame` details here — use `work-with-rooms`.
- Put leave confirm / match status chrome here — use `work-with-pages` (`App.vue`).
- Use `GRILLE_ANIM_MS = 1500` (wrong) — board and chrome share **1000** ms.

## Related

- Server seating / turn / move / budgets: `.agents/skills/server/work-with-game/SKILL.md`
- Room messages (`move` / `rescue` / `push` / `peek` / `endTurn` / `say`): `.agents/skills/server/work-with-messages/SKILL.md`
- Schema seats + turn + removed tiles: `.agents/skills/server/work-with-schema/SKILL.md`
- Lobby / room name: `.agents/skills/client/work-with-lobby/SKILL.md`
- Tourist reconnect: `.agents/skills/client/work-with-rooms/SKILL.md`
- Pinia `sendPeek` / budgets: `.agents/skills/client/work-with-stores/SKILL.md`
- Styles / theme chrome: `.agents/skills/client/work-with-styles/SKILL.md`
