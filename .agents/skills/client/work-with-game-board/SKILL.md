---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the tourist board UI in
  the happy-tourist client — GamePage CSS Grid from synced map `grid`
  (`lib/boardGeometry`), seat pieces by pieceId, presence HUD + focus@3,
  private knowledge HUD + remainingTurns badges (knowledge.md), shared
  open-on-land peek Q&A (peek.md), grille/catapult anim, free rescue + return
  (no push/end-turn/peek-open), say@12, strip by piece index (HUD ≤~420 → 2×2).
  Core in SKILL.md; topics peek.md / focus.md / knowledge.md. Brand-logo leave
  in App.vue.
---

# Work With Game Board

Use this skill for the **tourist board UI** on Game in the Vue 3 Quasar client (`happy-tourist.github.io`).

Product: настольная игра «Счастливый турист» (knowledge economy). On the seated client whose turn it is **and** `game.isPlaying`, the board and own strip accept piece selection by **pieceId**, local move hints, `game.sendMove` / free `sendRescue` / `sendReturnFromFinish`. Shared Q&A opens **on land** (place/submit only — no peek affordance). Authority on server (`work-with-game`). Presence: say **12** / focus **3** (end-turn retired); 60s turn ring; knowledge counter + lightbulb beside avatar; **strip-only** yellow TR carried + lightbulb; red TR `remainingTurns` above grille overlay; Notify peek toasts; finish travel ~550 / fade ~300. Affordance glyphs: `src/assets/board/{return,flag,focus,rescue,say}.png` — **no** Material colored circles; peek/push/end-turn PNGs retired.

## Specialized Topics

Read the matching file in this folder when the change involves that area
(do not load every file at once):

| Topic | File |
|-------|------|
| Shared peek modal / open-on-land / peekPlace+Submit | [peek.md](peek.md) |
| Focus nearest actionable tourist (@3) | [focus.md](focus.md) |
| Knowledge HUD / remainingTurns / finish modal | [knowledge.md](knowledge.md) |

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| App shell | `src/App.vue` | Always-button theme wordmarks (`logo-light`/`logo-dark`) height **28px** / toolbar **40** (SC-BRAND-29/30) left; on Game: logo click = leave + centered match status + theme + icon-only `header-game-leave` (aria `game.leave`, no visible label — SC-LEAVE-13); leave confirm + `leaveGame` → lobby. Non-Game: same button — auth/other → lobby, lobby noop (no Material `logout` leave) |
| Game page | `src/pages/GamePage.vue` | CSS Grid; unfinished pieces (+ finish travel ~550 / fade ~300); holes; grille/catapult overlays; board-busy; rescue/return above sprite (+20px down); **top** presence; sticky `.game-hud` (own + strip); **knowledge** + lightbulb beside avatar; say **12** / focus **3** / 60s turn ring; strip-only yellow TR knowledge; red TR `remainingTurns` above grille; open-on-land peek + Notify toasts; finish\|cut knowledge modal; return on finished strip; `rejoinGame` — no leave/status/roomId chrome |
| Game store | `src/stores/game.ts` | Room I/O; mirror seats/pieces (+ `remainingTurns`) / phase / `turnUntil` / peek session / traps; private `knowledgeDelivered`/`knowledgePieces`/`knowledgeOutcome`/`knowledgeSynced`; `allJailWarning` inert; Notify on `peekClose`; `sendMove`/`sendRescue`/`sendReturnFromFinish`/`sendPeekPlace`/`sendPeekSubmit`/`sendReady`/`sendSay` (`sendPush`/`sendEndTurn`/`sendPeek` retired no-ops) |

| Concern | Location |
| --- | --- |
| Board geometry | Synced `game.grid` (100 chars) → `parseBoardGeometry` / `buildBoardTiles` in `src/lib/boardGeometry.ts` (legacy `TOURIST_LAYOUT` fallback until first sync) |
| Tile build | `buildBoardTiles(grid)` → `div.tile`; removed `*` → hole; flipped digit from `flippedCells` |
| Pieces | `unfinishedBoardPieces` (+ short-lived disappearing finishers) → `img.piece`; PNG from `touristId`; `piece--trapped` when trapped |
| Grilles | Asset `src/assets/grilles/grille.png`; board overlay on `holdingGrilleKeys` **and** chrome grille on strip slots when `trapped`; drop/rise via **`GRILLE_ANIM_MS = 1000`** → CSS `--grille-anim-ms` for board + chrome (incl. leave-clear rise, SC-BOARD-18/19/21, SC-PIECE-31); **`filter: drop-shadow(...)`** on `.grille-overlay` + `.my-tourist-chrome-grille` (SC-BOARD-58; no opaque underlay); **new drops deferred** while catapult hop queue is presenting — flush on queue idle (SC-BOARD-29/30) |
| Catapults | Assets `src/assets/catapults/catapult.png` + `catapult-broken.png`; hidden until `revealingCatapultKeys`; **every viewer** (player + spectator): **land/arrival** onto catapult cell (synthesize if sync already relocated; no fake step if already on cell) → overlay → fling/finish travel; **sequential queue** (no chain cap); **yield/cancel own `moveAnimating`** when queue starts; pin on catapult cell during overlay → after full vanish → CSS travel `MOVE_ANIM_MS` to sync dest (**always finish travel** if finished/center even when finish sync arrives after reveal enqueue — SC-FINISH-20 / SC-BOARD-31); successful fade ~**`CATAPULT_ANIM_MS = 1500`** (CSS `--catapult-anim-ms`); broken: intact → **450 ms** → broken → **450 ms** → vanish **300 ms** (`--catapult-broken-vanish-ms`; no travel); **`filter: drop-shadow(...)`** on `.catapult-overlay` (SC-BOARD-58); **D13:** store `$patch` seats+revealing atomically; watch `flush: 'pre'`; attribute by coords transition / last-known / sole mover; **no silent skip** (enqueue overlay even without piece key); **defer grille drop** while catapult queue busy (`pendingGrilleDropKeys` — SC-BOARD-29/30) (SC-BOARD-22…31/58, SC-FINISH-19/20) |
| Board lock | Continuous `isBoardBusy` → `isInteractive` false: **intent lock before `sendMove`**, own move travel, land-before-overlay / move→trap gap, grille **drop/rise** (+ pipeline settle into hold so there is no unlock flash before drop), deferred grille pending, catapult overlay/holds/queue/pin/fling travel, finish disappear, return travel; **settled grille `hold` alone unlocks** so rescue affordances work while the grille stays visible; strip under lock; **say / leave outside** (SC-BOARD-27/29/30/32) |
| Presence | Seated: opponents top; sticky `.game-hud` = own + strip. Spectator: all top; no bottom HUD. Dual rings + 72px avatar; ready top-left; clock: say **12** / focus **3**; knowledge counter beside avatar (SC-PRESENCE-50/53) — see [knowledge.md](knowledge.md) |
| Say (game/say) | Affordance **12** on **own** online marker (hit-center on circumference; hit ~56–64); picker shifted **right** so left edge stays visible; **top** markers → `.say-bubbles--top` (down toward board); **own bottom** → `.say-bubbles--bottom` (up toward board). HUD scroll overflow must not clip (SC-SAY-15) |
| Tourist strip | Inside HUD after own marker when `mySeat` (+ pieces exist): `.my-tourist-slot` per pieceId — wide row by index; `@container game-hud (max-width: 420px)`: **2×2**. **No** chip/`q-menu`. Select unfinished non-trapped; finish flag + grille when trapped |
| Move UX | Local `selectedPieceId` + `legalTargets` when `isPlaying && isMyTurn && !isMySeatFinished && !isBoardBusy`; **keep-focus** after non-finishing move (SC-MOVE-46); never select finished/trapped; move does **not** end the turn |
| Rescue UX | Affordance above trapped sprite (+20px down) when adj free + board action not spent → `sendRescue(pieceId)` (free; SC-BOARD-71 / SC-MOVE-113) |
| Push UX | **Retired** — no push affordance (SC-MOVE-114) |
| Return UX | Illustrated `return.png` above finished strip tourist (+20px down) when `canReturn` → return-mode + `.tile--target` → `sendReturnFromFinish` (SC-FINISH-09/13/15/16) |
| Peek UX | See [peek.md](peek.md) — open-on-land shared Q&A (`sendPeekPlace` / `sendPeekSubmit` only) |
| Focus UX | See [focus.md](focus.md) — `focus.png` @3 → nearest actionable pieceId |
| Knowledge UX | See [knowledge.md](knowledge.md) — delivered counter + lightbulb; strip TR only; finish\|cut modal |
| Finish UX | Center land → slide + fade; own `finishPlace` 0→1 → **knowledge** modal by `outcome` finish\|cut (not place); stay in room. Activate clicked finish cell only (SC-MOVE-65 / SC-FINISH-31/34) |
| All-jail | **Retired** teleport modal — server waits on `remainingTurns`; client ignores `allJailWarning` |
| Center | One `BoardTile` per grid `7` with `kind: 'center'` + finish skin; **no** contiguous merge / `span 2` (SC-BOARD-55) |
| Room enter | Out of scope — see `work-with-lobby` / `work-with-rooms` (`rejoinGame`) |

## Layout And Tiles

- Grid: **10×10** sparse; empty corners are holes (no tile element; page background shows through).
- Kinds: `start` / `task` / `center` (finish) — PNG skins from `src/assets/tiles/` via shared `src/lib/tileSkins.ts` (`start.png`, `tile.png`, `tile-back.png` when flipped, `finish.png`); bind `tileSkinCssVars()` on `.tourist-board`; see `work-with-styles/board.md`.
- After **correct** peek only: synced `game.removedTaskKeys` (`"r,c"`) → task tile omitted (same hole chrome as empty corners); cell stays **landable** (red hints include it; layout never-playable stays out). A piece already on that cell **remains** (stand-on-hole OK — SC-BOARD-12). Incorrect / forced-incorrect KEEP leaves the unbound/flipped task skin and hidden reward (no hole).
- `.tourist-board`: `width: 100%`, `max-width: calc(10 * 60px + 9 * 2px)`, `aspect-ratio: 1`, `--gap: 2px`, `grid-template-*: repeat(10, 1fr)`; playable tile `border-radius: 0` (do not clip notched skins with `--radius: 2`). Do **not** size rows with `%` of auto height (tracks collapse to 0).
- Container: full width of Game content (no side presence gutters); mobile edge-to-edge relative to page content; wide screens capped by max tile 60px (via board max-width + square aspect).
- Tile skins are **theme-independent**, not retinted by Quasar Dark chrome (see `work-with-styles` / theme specs).

## Move Interaction (current turn only — D5 / SC-MOVE-11…13/46/65, SC-BOARD-05/06/14)

| Rule | Behavior |
|------|----------|
| Who interacts | Only seated **non-finished** client with `isPlaying && sessionId === currentTurnSessionId` and not mid-board-animation (`!isBoardBusy`) |
| Select | Click own **unfinished non-trapped** piece on board **or** matching strip slot → `selectedPieceId`; white outline. Changing select clears return-mode |
| Hints | Legal one-step neighbors (Chebyshev 1, incl. removed holes, unoccupied) → red outline; **local only**; hide when board-action spent / board busy. Return-mode same `.tile--target` |
| Reselect | May change `selectedPieceId` among own unfinished non-trapped until submit |
| Submit | Activate red destination → `game.sendMove(pieceId, row, col)`; **turn stays** |
| Center finish click | Activate that finish cell only (`cellKey` / legal-key membership); **no** nearest-legal remap (SC-MOVE-65) |
| Keep-focus | After successful **non-finishing** move: **do not** clear `selectedPieceId` (SC-MOVE-46). Center finish → clear |
| Peek / Focus | See [peek.md](peek.md) / [focus.md](focus.md) |
| Rescue / Return | `sendRescue` / `sendReturnFromFinish` with **pieceId** payloads (no push) |
| Clear selection | Piece finishes / traps; lose turn / not playing; time-expired / finished seat; also clears return-mode |
| Finished | Off board after disappear; strip finish icon; dim only if `!canReturn` |
| Others | Spectators / not-your-turn / not playing / finished seat: no move/peek/push chrome |

Do **not** sync selection or hints — page-local refs only. Do **not** assume `sendMove` advances the turn. Do **not** reintroduce compact chip / `q-menu` picker (SC-PIECE-29/30 removed).

## Piece Travel Animation (SC-MOVE-14…15 / SC-FINISH-01/15/17/18/19/20)

- Position pieces with CSS `transform` / absolute offsets from grid vars (`--prow` / `--pcol`), not tweening `grid-row`/`grid-column`.
- Ordinary board moves: ~250ms ease-out (`MOVE_ANIM_MS`); all clients animate when synced `row`/`col` changes.
- Finish slide: ~**550 ms** (`FINISH_TRAVEL_MS`); finish fade: ~**300 ms** (`FINISH_FADE_MS`, CSS delay = slide) — all enter-center paths (ordinary move incl. center click, catapult→center). Catapult overlay (~1500 ms) unchanged.
- On submit: set presentation **intent lock before `sendMove`** (continuous `isBoardBusy` through move→trap/catapult/grille); ignore board/strip clicks until idle; keep `selectedPieceId` unless the destination is center finish (SC-MOVE-46 / SC-BOARD-27/32).
- Skip transition on first paint so pieces do not fly from origin.
- **Catapult fling (SC-BOARD-23…31 / SC-FINISH-19/20):** every viewer: **land onto catapult cell first** (synthesize when sync already at fling dest; skip fake step if already on cell / after in-flight arrival) → overlay → travel. While overlay runs, pin piece on catapult cell; **only after** full vanish start travel to dest / finish travel+fade; chain = sequential land→overlay→travel→next (every hop animated; finish travel only after **last** vanish to center); broken holds 450+450 then vanish 300 ms with no travel. Enqueue must not silent-skip when piece attribution is weak. When finish sync arrives **after** reveal enqueue, still run finish travel after vanish (`shouldFinishTravelAfterVanish` / `markDeferredFinish` while pin/queue owns the piece). In `releaseDeferredFinishAfterVanish`, add `disappearingKeys` **before** clearing `catapultDeferFinishKeys` so `boardPieces` never drops the finisher for a tick. When the catapult queue takes over, **cancel own `moveAnimating`** (clear timer / flag) so land→overlay→fling owns the piece — do not let the ordinary move travel race the synthesized land.
- **Center finish (move):** sync may already place the piece on a center cell — paint one frame at `lastKnownBoardCellByKey` via `finishAnimFromByKey`, hold until painted, then clear so CSS slides onto center + fade (SC-FINISH-01/17/18). **Own move onto center:** seed only `lastKnownBoardCellByKey` before `sendMove`. Finish watch sets `finishAnimFrom` when finished arrives. **Catapult→center:** defer finish travel until catapult vanish (SC-FINISH-20 / SC-BOARD-31). Push→center path retired.
- **Return-from-finish:** when a piece leaves the finished set, all clients animate from `nearestCenterCell(target)` onto the ring (SC-FINISH-15).

## Presence (occupied seats — top row + seated bottom HUD)

Sync-driven markers. Page reads mirrored `game.seats` + private `knowledgeDelivered` / `knowledgePieces` / `knowledgeSynced` — no Colyseus I/O here. Match status in `App.vue` header (SC-PRESENCE-23). Details: [knowledge.md](knowledge.md) / [focus.md](focus.md).

| Rule | Behavior |
|------|----------|
| Who | One marker per **occupied** seat (incl. offline-in-grace) |
| Avatar | Seat `touristId` → tourist PNG; **72px** (SC-PRESENCE-13) |
| Seated viewer | Top opponents row (reserve ~96px); sticky `.game-hud` α 0.78: own + strip; **no** chip/`q-menu` |
| Spectator | All occupied → top; **no** bottom HUD |
| Offline / rings | Inner reconnect ring from `reconnectUntil`; outer turn ring from `turnUntil` / `turnBudgetSeconds` (60s multi when ≥2 eligible; 0 leftover/single-player — SC-PRESENCE-54/10) |
| Dual rings | Sibling progress + sibling avatar (SC-PRESENCE-10/12) — do not nest |
| Match status | App header; `peekClose` → `peekStatusFlash` 10s + store Notify toast (SC-PRESENCE-36 / SC-BOARD-88) |
| Room id | Never show in Game chrome |
| Finish place badge | **Retired** — no ranked place chrome |
| Ready | Own marker top-left when `canSendReady` |
| Say | Own online **12** |
| Focus | Own **3** (former end-turn slot) — [focus.md](focus.md) |
| Knowledge | Own delivered counter beside avatar; +N fall on delivered growth after sync — [knowledge.md](knowledge.md) |
| End-turn / leftover ∞ / solo timer modals | **Retired** |

Finish flag + return-icon + red TR grille badges (board overlay above grille + strip); yellow knowledge badge **strip TR only** (+ lightbulb); presence shows delivered knowledge counter + lightbulb.

### Presence DOM (canonical — SC-PRESENCE-04/10/11/12/13)

Inside `.presence-marker` (96×96 = outer ring, `position: relative`), **three siblings** — never nest progress in progress, never put avatar only in progress default slot without `show-value`:

```html
<q-circular-progress class="presence-progress presence-progress--outer" size="96px" … />
<q-circular-progress class="presence-progress presence-progress--inner" size="84px" … />
<img class="presence-avatar" :src="touristSrc(touristId)" alt="" />
```

CSS: outer ring absolute inset 0; inner centered; avatar sibling z-index 2 (**72×72**). Ready (top-left), say (**12**), focus (**3**) at `z-index: 3+`. Shared `.board-action-icon` **36×36** for say/focus/rescue/return (peek/push/end-turn retired).

Layout shell: board scroll region → sticky `.game-hud` when seated (`container-name: game-hud`). Do **not** put `overflow-x: auto` on the marker box (SC-SAY-15); do not revive chip/`q-menu` / end-turn dock / opponents in HUD.

## Say Bubbles At Presence (game/say — SC-SAY-07…12/15)

Ephemeral preset phrases near presence markers. Protocol + Pinia I/O: `work-with-stores` / `colyseus-client` / server `work-with-messages`. UI stays on `GamePage` beside markers — **not** Quasar Notify / viewport toasts.

| Rule | Behavior |
|------|----------|
| Who may send | Seated + `connected` only; anytime (not turn-gated). Spectators / offline grace: no affordance |
| Affordance | Only on marker with `sessionId === game.sessionId` and connected (`canSendSay`) — **12** circumference (illustrated `say.png`, no solid colored circle); hit ~56–64; `pointer-events: auto` + z-index above rings/avatar; HUD overflow MUST NOT clip (SC-SAY-15) |
| Ready | **Top-left** on own marker when `canSendReady` → `game.sendReady()` (not `sendSay('ready')`); i18n `game.readyButton` |
| Countdown | Full-screen overlay while `phase === 'countdown'` from synced `countdownRemaining` (all clients) |
| Picker | Click affordance → two presets «Всем привет» (`hello`) / «Удачи» (`luck`) via i18n `game.say.*`; choose → `game.sendSay(presetId)` and close immediately; click away closes; shift **right** so left edge is fully visible in HUD (SC-SAY-15) |
| Bubbles | From `game.sayEvents` filtered by sender `sessionId`; label via `$t('game.say.' + presetId)` (incl. `ready` → «Готов начать!») |
| TTL | Disappear after **10s** from server `at` (`SAY_TTL_MS`); prefer `at`, not local receive time |
| Max live | At most **3** per sender; store refuses 4th locally; server enforces the same |
| Stack | **Top** markers (opponents / spectator): `.say-bubbles--top` — grow **down** toward the board. **Own** bottom marker: `.say-bubbles--bottom` — grow **up** toward the board. Newer closer to avatar. **No** left/right side-slot stacks (SC-SAY-11/12) |
| Row gap | Horizontal gap between markers so concurrent neighbor bubbles do not overlap; horizontal scroll on `.game-hud__scroll` OK (bar stays `overflow: visible`) |
| I/O | Page never `room.send` / `room.onMessage` — only `sendSay` / `sendReady` + read `sayEvents` |

Picker open state (`sayPickerOpen`) is page-local; close if the local seat is lost or goes offline.

## Authority

- Board **geometry** from synced `game.grid` via `lib/boardGeometry`. Holes from `removedTaskKeys`; flipped digits from `flippedCells`.
- Seats / phase / maxSeats / peek session / traps / `finishPlace` / piece `remainingTurns` are server-authoritative.
- Own **knowledge** is private (`knowledge` message); never show others’ counters. Shared peek session is schema.
- Local legal targets are hints only. Say is ephemeral (not schema).
- Do not reintroduce draughts `{ from, to }` / steps-peeks budgets / push / end-turn / peek-open.

## GamePage Responsibilities

- Render board + unfinished pieces + grille/catapult overlays; sticky HUD when seated.
- Presence: say@12 + focus@3 + knowledge counter + lightbulb; strip with finish/return/grille TR + knowledge TR (+ lightbulb).
- Own turn: select + `sendMove` / free `sendRescue` / `sendReturnFromFinish`; open-on-land peek place/submit; board-action local gate.
- `finishPlace` 0→1 → knowledge modal; remount → `rejoinGame`; leave/status in App.

## Do

- Geometry in `lib/boardGeometry`; Colyseus I/O in `stores/game` only.
- Gate moves with `isPlaying && isMyTurn && !isMySeatFinished && !isBoardBusy` (+ local board-action spent).
- Keep focus@3; knowledge HUD; leave/status in App.

## Don't

- Call `room.send` from the page — only store sends (`sendMove` / `sendRescue` / `sendReturnFromFinish` / `sendPeekPlace` / `sendPeekSubmit` / `sendSay` / `sendReady`).
- Revive push / end-turn / peek-open / density / leftover ∞ / place / timer-end modals.
- Put knowledge counters on opponents; nest progress; chip/`q-menu`; finish span merge.
- Assume move advances turn by itself — server advances after land resolve / peek close / all-jail.
- Use `GRILLE_ANIM_MS = 1500` — board/chrome share **1000** ms; defer grille drop while catapult queue busy.

## Related

- Server game / knowledge: `.agents/skills/server/work-with-game/SKILL.md`
- Messages: `.agents/skills/server/work-with-messages/SKILL.md`
- Schema: `.agents/skills/server/work-with-schema/SKILL.md`
- Lobby / rooms / stores / styles: sibling client skills; topics `knowledge.md` / `peek.md` / `focus.md`
