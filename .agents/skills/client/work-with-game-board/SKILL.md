---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the tourist board UI in
  the happy-tourist client — GamePage CSS Grid (gap/radius 2), synced seat pieces
  overlay, presence rows (top/bottom, no side columns) with dual rings + corner
  affordances, preset say bubbles toward board, personal tourist strip, tile
  kinds (start/task/center), current-turn selection/hints/move submit, or piece
  travel animation for room tourist.
---

# Work With Game Board

Use this skill for the **tourist board UI** on Game in the Vue 3 Quasar client (`happy-tourist.github.io`).

Product: настольная игра «Счастливый турист». On the seated client whose turn it is **and** `game.isPlaying` (`phase === 'playing'`), the board and strip accept piece selection, local move hints, and move submit via `game.sendMove`. Before playing: no move chrome / submit. Spectators and non-current seated players remain non-interactive for moves. Authority for legality stays on the server (`work-with-game`). Preset say bubbles are separate from turn — any seated+online player may send via `game.sendSay` (server whitelist). Ready-to-start uses `game.sendReady` on own marker **top-left** when `canSendReady` (say affordance stays **top-right**).

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Game page | `src/pages/GamePage.vue` | CSS Grid field; unfinished pieces overlay (+ short center disappear); presence + place badge + say; strip×4 with finish icons; local selection/hints; place `q-dialog`; icon-only leave; confirm when seated ∧ `playing` ∧ `finishPlace === 0` ∧ `!timeExpired` → lobby |
| Game store | `src/stores/game.ts` | Room I/O; mirror `seats` (+ piece `finished` / seat `finishPlace`) / `phase` / `maxSeats` / `countdownRemaining` / `currentTurnSessionId` / `sessionId`; `unfinishedBoardPieces` / `isMySeatFinished` / `myFinishedStripSides`; `isMyTurn` / `isPlaying` / `canSendReady`; `sendMove` / `sendReady` / `sendSay` + `sayEvents` |

| Concern | Location |
| --- | --- |
| Layout constant | `LAYOUT` string grid in `GamePage.vue` (`.` hole, `1` start, `*` task, `7` center) |
| Tile build | `buildBoardTiles()` → `div.tile` with `gridColumn` / `gridRow` |
| Pieces | `unfinishedBoardPieces` (+ short-lived disappearing finishers) → `img.piece`; PNG from `touristId` |
| Presence | Occupied seats → top/bottom `.presence-row-scroll` → `.presence-row` (no left/right); dual rings + 72px avatar; finish/ready top-left; say top-right |
| Say (game/say) | Affordance top-right on **own** online marker (hit-area ≥ ~32px); bubbles toward board from `game.sayEvents`; row overflow must not clip (SC-SAY-15) |
| Strip | Below board when `mySeat`: four slots `N→E→S→W`; finished slots inactive + finish icon (top-right) |
| Move UX | Local `selectedSide` + `legalTargets` only when `isPlaying && isMyTurn && !isMySeatFinished && !moveAnimating`; never select finished pieces |
| Finish UX | Center land → slide + fade (`disappearingKeys`); own `finishPlace` 0→N → place modal; stay in room after close |
| Center | One element with `span 2` / `span 2` (solid 2×2), not four cells |
| Room enter | Out of scope — see `work-with-lobby` / `work-with-rooms` (`rejoinGame`) |

## Layout And Tiles

- Grid: **10×10** sparse; empty corners are holes (no tile element; page background shows through).
- Kinds: `start` (green), `task` (brown), `center` (yellow).
- `.tourist-board`: `width: 100%`, `max-width: calc(10 * 60px + 9 * 2px)`, `aspect-ratio: 1`, `--gap: 2px`, `--radius: 2px`, `grid-template-*: repeat(10, 1fr)`; tile `border-radius: var(--radius)`. Do **not** size rows with `%` of auto height (tracks collapse to 0).
- Container: full width of Game content (no side presence gutters); mobile edge-to-edge relative to page content; wide screens capped by max tile 60px (via board max-width + square aspect).
- Tile colors are **fixed fills**, independent of Quasar Dark chrome (see `work-with-styles` / theme specs).

## Move Interaction (current turn only — D5 / SC-MOVE-11…13, SC-BOARD-05/06)

| Rule | Behavior |
|------|----------|
| Who interacts | Only seated **non-finished** client with `isPlaying && sessionId === currentTurnSessionId` and not mid-own-animation |
| Select | Click own **unfinished** piece on board or matching strip slot → `selectedSide`; white outline on that cell |
| Hints | Legal one-step neighbors (Chebyshev 1, playable LAYOUT cell, unoccupied by unfinished) → red outline; **local only** |
| Reselect | May change `selectedSide` among own unfinished pieces until submit |
| Submit | Activate red destination → `game.sendMove(side, row, col)`; clear selection |
| Finished | Finished pieces stay off the board after disappear; finished strip slots show finish icon and never select/submit; finished seat has no move chrome |
| Others | Spectators / not-your-turn / not playing / finished seat: no selection, no red/white move chrome, no `sendMove` |

Do **not** sync selection or hints — page-local refs only.

## Piece Travel Animation (D6 / SC-MOVE-14…15)

- Position pieces with CSS `transform` / absolute offsets from grid vars (`--prow` / `--pcol`), not tweening `grid-row`/`grid-column`.
- Duration ~200–300ms ease-out (`MOVE_ANIM_MS`); all clients animate when synced `row`/`col` changes.
- On submit: set `moveAnimating` and ignore board/strip clicks until timer ends; clear selection.
- Skip transition on first paint so pieces do not fly from origin.

## Presence (occupied seats)

Sync-driven markers in rows above/below the board (SC-PRESENCE-01…14 / design D5/D9–D11). Page reads mirrored `game.seats` only — no Colyseus I/O here.

| Rule | Behavior |
|------|----------|
| Who | One marker per **occupied** seat (including offline-in-grace). Empty slots not rendered. |
| Avatar | Seat `touristId` → same tourist PNG as pieces; **image box = strip tourist (72px)** (SC-PRESENCE-13) |
| Seated viewer | Self → **bottom row** alone; other seats → **one top row** L→R by join order among others. **No left/right columns** (SC-PRESENCE-02 / D9) |
| Spectator | All occupied seats → **one top row** L→R by join order; bottom empty (SC-PRESENCE-03) |
| Join order | Array order from sync map `forEach` as mirrored into `seats[]` |
| Offline | `!connected && reconnectUntil > 0` → **inner** warning `QCircularProgress` (`min=0`, `max=30`, `value` = remaining from `reconnectUntil − now`) |
| Reserved chrome | Always outer ring slot sized to outer progress (96px around 72px avatar — SC-PRESENCE-11); inactive turn/grace → transparent track/value 0 (no size jump) |
| Turn deadline | Outer determinate ring while `playing` + current turn + synced `turnUntil`/`turnBudgetSeconds`: blue (`primary`) for multi 60s; red (`negative`) when `turnBudgetSeconds === 300` (solo). **No** static blue outline / `--turn` box-shadow |
| Dual rings | Offline current-turn: outer = turn, inner = reconnect (both visible — SC-PRESENCE-10). **Siblings** only (outer absolute behind) — do **not** nest `q-circular-progress`. **Avatar:** always sibling `<img class="presence-avatar">` on top (pre-timer pattern); do **not** put avatar only in progress default slot without `show-value` (Quasar omits that slot from the DOM — SC-PRESENCE-12) |
| Header | «Ваш ход» / «Ход соперника» / «Ход игрока» (spectator) from turn, not from outline |
| Finish place | Seat `finishPlace > 0` → numeric badge at **top-left** of marker (SC-PRESENCE-14) |
| Ready | Own marker only, **top-left** when `canSendReady` (does not overlap finish by phase) |
| Say affordance | Own online marker only, **top-right** (SC-PRESENCE-14 / SC-SAY-07); never on opponents / spectators |

Tick `nowMs` on an interval (~200 ms) while Game is mounted so turn + reconnect rings animate from **server** `turnUntil` / `reconnectUntil`.

Spectators and seated players see the same occupied set; layouts differ as above. Strip finish chrome lives on the personal strip — presence only shows the **place** badge.

### Presence DOM (canonical — SC-PRESENCE-04/10/11/12/13)

Inside `.presence-marker` (96×96 = outer ring, `position: relative`), **three siblings** — never nest progress in progress, never put avatar only in progress default slot without `show-value`:

```html
<q-circular-progress class="presence-progress presence-progress--outer" size="96px" … />
<q-circular-progress class="presence-progress presence-progress--inner" size="84px" … />
<img class="presence-avatar" :src="touristSrc(touristId)" alt="" />
```

CSS: outer ring `position: absolute; inset: 0; z-index: 0`; inner ring absolute centered (`top/left: 50%`, `transform: translate(-50%, -50%)`, `z-index: 1`); avatar `position: relative; z-index: 2` (**72×72**, matches `.my-tourist-slot`). Rings and avatar use `pointer-events: none` so affordances / marker clicks pass through. Finish badge / ready (top-left) and say (top-right) sit above with `z-index: 3+` (say affordance / picker higher, `pointer-events: auto`). Say affordance hit-area ≥ ~32 CSS px (icon glyph may be smaller — SC-SAY-15).

Layout shell: `.presence-frame` is a **column flex** — `.presence-row-scroll` (top) → `.tourist-board` → `.presence-row-scroll` (bottom) (omit empty rows). Inside each scroll wrapper: `.presence-row` with `gap: 48px`, **`overflow: visible`**, and `pointer-events: auto`. Horizontal scroll lives on `.presence-row-scroll` only (wrapper stays `pointer-events: none` so padding/margin Y absorb does not steal board clicks) — do **not** put `overflow-x: auto` on the same box as the markers (browsers force Y clip and hide say chrome — D15 / SC-SAY-15). Do **not** shrink avatar below strip size.

## Say Bubbles At Presence (game/say — D3/D4/D11 / SC-SAY-07…12)

Ephemeral preset phrases near presence markers. Protocol + Pinia I/O: `work-with-stores` / `colyseus-client` / server `work-with-messages`. UI stays on `GamePage` beside markers — **not** Quasar Notify / viewport toasts.

| Rule | Behavior |
|------|----------|
| Who may send | Seated + `connected` only; anytime (not turn-gated). Spectators / offline grace: no affordance |
| Affordance | Only on marker with `sessionId === game.sessionId` and connected (`canSendSay`) — **top-right** (`chat_bubble_outline`); hit-area ≥ ~32px; `pointer-events: auto` + z-index above rings/avatar; row overflow MUST NOT clip (SC-SAY-15) |
| Ready | **Top-left** on own marker when `canSendReady` → `game.sendReady()` (not `sendSay('ready')`); i18n `game.readyButton` |
| Countdown | Full-screen overlay while `phase === 'countdown'` from synced `countdownRemaining` (all clients) |
| Picker | Click affordance → two presets «Всем привет» (`hello`) / «Удачи» (`luck`) via i18n `game.say.*`; choose → `game.sendSay(presetId)` and close immediately; click away closes |
| Bubbles | From `game.sayEvents` filtered by sender `sessionId`; label via `$t('game.say.' + presetId)` (incl. `ready` → «Готов начать!») |
| TTL | Disappear after **10s** from server `at` (`SAY_TTL_MS`); prefer `at`, not local receive time |
| Max live | At most **3** per sender; store refuses 4th locally; server enforces the same |
| Stack toward board | Top-row markers → bubbles **below** avatar (`.say-bubbles--top`); bottom self → bubbles **above** (`.say-bubbles--bottom`). Newer closer to avatar. **No left/right side-slot stacks** (SC-SAY-11/12 / D11) |
| Row gap | Horizontal gap between markers so concurrent neighbor bubbles do not overlap; horizontal scroll on `.presence-row-scroll` wrapper OK if needed (row itself stays `overflow: visible`) |
| I/O | Page never `room.send` / `room.onMessage` — only `sendSay` / `sendReady` + read `sayEvents` |

Picker open state (`sayPickerOpen`) is page-local; close if the local seat is lost or goes offline.

## Authority

- Board **geometry** (`LAYOUT`) is a client constant; server does not sync tile kinds (server mirrors playable set in `touristMove.ts`).
- **Seats**, `phase` / `maxSeats` / `countdownRemaining`, connectivity, **`currentTurnSessionId`**, **`turnUntil` / `turnBudgetSeconds`**, and seat **`timeExpired` / `finishPlace`** are server-authoritative; client mirrors and renders.
- Local legal-target computation is a **hint** only — server rejects illegal moves.
- Say bubbles are **ephemeral room messages**, not schema state; server whitelist + live limit are authoritative.
- Do not reintroduce legacy draughts CellValue `0…4` / `{ from, to }` encoding.

## GamePage Responsibilities

- Render `boardTiles` from `LAYOUT`.
- Overlay **unfinished** pieces (plus short-lived disappearing finishers) at `row`/`col` (0-based schema → CSS vars / transform).
- Render **presence** markers for occupied seats in top/bottom rows (dual turn/reconnect rings + reserved 96px chrome; place badge top-left when `finishPlace > 0`).
- On own online marker: say affordance **top-right** + picker (finished / time-expired keep say); ready **top-left** when `canSendReady`; for every marker: live say bubbles from `sayEvents` (TTL / stack toward board).
- Full-screen countdown overlay while `phase === 'countdown'`.
- Show strip «Мои туристы» only if `mySeat` **and** own pieces exist (empty before `playing` materialize); finished slots inactive + finish icon; on own turn **in playing** select only unfinished.
- Board overlay: unfinished pieces only when seats have pieces (none in waiting/countdown).
- On own turn in playing (not finished / not time-expired): white selection + red targets; submit via store `sendMove`.
- Animate piece travel; on center finish keep DOM key for slide then fade (`FINISH_FADE_MS`); ignore input while `moveAnimating`.
- Own `finishPlace` 0→N → place `q-dialog` (`game.finishPlaceModal*`); own `timeExpired` false→true → timeout `q-dialog` (`game.timeExpiredModal*`); close keeps player in room; clear selection on expiry.
- Header status: turn labels only in playing; exit **icon-only** Material `logout` with accessible name via `aria-label` / i18n `game.leave` (no visible text label); confirm copy via `leaveConfirm` / `leaveCancel` / `leaveExit`; confirm only when seated ∧ `playing` ∧ `!finishPlace` ∧ `!timeExpired` (else immediate leave); remount without room → `rejoinGame(roomId)` via store.

## Do

- Keep layout in one client constant; center as a single 2×2 grid area.
- Preserve max tile 60px, gap 2, radius 2, hole = page background.
- Keep Colyseus I/O in `stores/game`; page reads seats/phase/turn/strip/presence/`sayEvents` from store only.
- Gate move interactivity with `isPlaying && isMyTurn && !isMySeatFinished && !isMySeatTimeExpired` (and `!moveAnimating`); never select finished pieces/slots; gate say with own seated+connected; ready via `sendReady`.
- Map `touristId` 1…4 to `tourist{N}.png`; strip only after pieces exist; finish icon on finished sides.
- Drive turn/reconnect countdowns from synced `turnUntil` / `reconnectUntil`; countdown overlay from `countdownRemaining`.
- Expire bubbles from server `at` + `SAY_TTL_MS`; keep max 3 live per sender in UI.
- Confirm leave only when seated ∧ playing ∧ `finishPlace === 0` ∧ `!timeExpired`; finished / time-expired leave immediately.

## Don't

- Call `room.send` from the page — only `game.sendMove` / `game.sendSay` / `game.sendReady`.
- Show white/red move chrome before `playing`, to spectators, non-current players, finished seats, or finished pieces.
- Show say send affordance on other players’ markers or to spectators.
- Use Quasar Notify / screen-edge toasts as the say carrier.
- Depend tile fills on Quasar Dark / theme preference.
- Invent client-local seat assignment (server assigns on join).
- Treat client hints as authority.
- Sync selection / hints / say bubbles to schema.
- Put lobby subscribe/create/join logic into the board skill — use `work-with-lobby`.
- Put reconnect token / `rejoinGame` details here — use `work-with-rooms`.

## Related

- Server seating / turn / move: `.agents/skills/server/work-with-game/SKILL.md`
- Room messages (`move` / `say`): `.agents/skills/server/work-with-messages/SKILL.md`
- Schema seats + turn: `.agents/skills/server/work-with-schema/SKILL.md`
- Lobby / room name: `.agents/skills/client/work-with-lobby/SKILL.md`
- Tourist reconnect: `.agents/skills/client/work-with-rooms/SKILL.md`
- Pinia `sendSay` / `sayEvents`: `.agents/skills/client/work-with-stores/SKILL.md`
- Styles / theme chrome: `.agents/skills/client/work-with-styles/SKILL.md`
