---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the tourist board UI in
  the happy-tourist client — GamePage CSS Grid layout, synced seat pieces
  overlay, occupied presence circles, personal tourist strip, tile kinds
  (start/task/center), current-turn selection/hints/move submit, or piece travel
  animation for room tourist.
---

# Work With Game Board

Use this skill for the **tourist board UI** on Game in the Vue 3 Quasar client (`happy-tourist.github.io`).

Product: настольная игра «Счастливый турист». On the seated client whose turn it is (`game.isMyTurn`), the board and strip accept piece selection, local move hints, and move submit via `game.sendMove`. Spectators and non-current seated players remain non-interactive for moves. Authority for legality stays on the server (`work-with-game`).

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Game page | `src/pages/GamePage.vue` | CSS Grid field; pieces overlay; presence; strip×4; local selection/hints; travel animation; leave → lobby |
| Game store | `src/stores/game.ts` | Room I/O; mirror `seats` / `started` / `currentTurnSessionId` / `sessionId`; `isMyTurn`; `sendMove` |

| Concern | Location |
| --- | --- |
| Layout constant | `LAYOUT` string grid in `GamePage.vue` (`.` hole, `1` start, `*` task, `7` center) |
| Tile build | `buildBoardTiles()` → `div.tile` with `gridColumn` / `gridRow` |
| Pieces | Flatten all seats’ `pieces` → `img.piece` positioned via CSS vars/`transform`; PNG from `touristId` |
| Presence | Occupied seats only → `.presence-frame`; offline → `QCircularProgress` from `reconnectUntil` |
| Strip | Below board when `mySeat`: four slots `N→E→S→W`; clickable on own turn |
| Move UX | Local `selectedSide` + `legalTargets` (white/red chrome) only when `isMyTurn && !moveAnimating` |
| Center | One element with `span 2` / `span 2` (solid 2×2), not four cells |
| Room enter | Out of scope — see `work-with-lobby` / `work-with-rooms` (`rejoinGame`) |

## Layout And Tiles

- Grid: **10×10** sparse; empty corners are holes (no tile element; page background shows through).
- Kinds: `start` (green), `task` (brown), `center` (yellow).
- `.tourist-board`: `width: 100%`, `max-width: calc(10 * 60px + 9 * 6px)`, `aspect-ratio: 1`, `gap: 6px`, `grid-template-*: repeat(10, 1fr)`; tile `border-radius: 12px`. Do **not** size rows with `%` of auto height (tracks collapse to 0).
- Container: full width of Game content; mobile edge-to-edge relative to page content; wide screens capped by max tile 60px (via board max-width + square aspect).
- Tile colors are **fixed fills**, independent of Quasar Dark chrome (see `work-with-styles` / theme specs).

## Move Interaction (current turn only — D5 / SC-MOVE-11…13, SC-BOARD-05/06)

| Rule | Behavior |
|------|----------|
| Who interacts | Only seated client with `sessionId === currentTurnSessionId` and not mid-own-animation |
| Select | Click own piece on board or matching strip slot → `selectedSide`; white outline on that cell |
| Hints | Legal one-step neighbors (Chebyshev 1, playable LAYOUT cell, unoccupied incl. own) → red outline; **local only** |
| Reselect | May change `selectedSide` among own pieces until submit |
| Submit | Activate red destination → `game.sendMove(side, row, col)`; clear selection |
| Others | Spectators / not-your-turn: no selection, no red/white move chrome, no `sendMove` |

Do **not** sync selection or hints — page-local refs only.

## Piece Travel Animation (D6 / SC-MOVE-14…15)

- Position pieces with CSS `transform` / absolute offsets from grid vars (`--prow` / `--pcol`), not tweening `grid-row`/`grid-column`.
- Duration ~200–300ms ease-out (`MOVE_ANIM_MS`); all clients animate when synced `row`/`col` changes.
- On submit: set `moveAnimating` and ignore board/strip clicks until timer ends; clear selection.
- Skip transition on first paint so pieces do not fly from origin.

## Presence (occupied seats)

Sync-driven markers around the board (SC-PRESENCE-01…05 / design D5). Page reads mirrored `game.seats` only — no Colyseus I/O here.

| Rule | Behavior |
|------|----------|
| Who | One marker per **occupied** seat (including offline-in-grace). Empty slots not rendered. |
| Avatar | Seat `touristId` → same tourist PNG as pieces |
| Seated viewer | Self → **bottom** (home/north); other seats by join order → **top**, **left**, **right** |
| Spectator | Join order among seated → **top**, **bottom**, **left**, **right**; omit missing positions |
| Join order | Array order from sync map `forEach` as mirrored into `seats[]` |
| Offline | `!connected && reconnectUntil > 0` → wrap avatar in `QCircularProgress` (`min=0`, `max=30`, `value` = remaining seconds from `reconnectUntil − now`) |
| Online | Avatar only — no countdown ring |

Tick `nowMs` on an interval (~200 ms) while Game is mounted so the ring animates from the **server** deadline, not a local fixed “30” without `reconnectUntil`.

Spectators and seated players see the same occupied set; layouts differ as above. Do **not** add strip×4 “in game / passed” chrome here — presence only.

## Authority

- Board **geometry** (`LAYOUT`) is a client constant; server does not sync tile kinds (server mirrors playable set in `touristMove.ts`).
- **Seats**, `started`, connectivity, and **`currentTurnSessionId`** are server-authoritative; client mirrors and renders.
- Local legal-target computation is a **hint** only — server rejects illegal moves.
- Do not reintroduce legacy draughts CellValue `0…4` / `{ from, to }` encoding.

## GamePage Responsibilities

- Render `boardTiles` from `LAYOUT`.
- Overlay **all** pieces of **all** seats at `row`/`col` (0-based schema → CSS vars / transform).
- Render **presence** markers for occupied seats (layouts + offline ring above).
- Show strip «Мои туристы» only if `mySeat`; on own turn allow select + destination click.
- On own turn: white selection + red targets; submit via store `sendMove`.
- Animate piece travel for everyone; ignore input while `moveAnimating`.
- Status from store; leave → `leaveGame` + lobby; remount without room → `rejoinGame(roomId)` via store.

## Do

- Keep layout in one client constant; center as a single 2×2 grid area.
- Preserve max tile 60px, gap 6, radius 12, hole = page background.
- Keep Colyseus I/O in `stores/game`; page reads seats/turn/strip/presence from store only.
- Gate interactivity with `isMyTurn` (and `!moveAnimating`).
- Map `touristId` 1…4 to `tourist{N}.png`; strip shows only the local player’s four slots.
- Drive offline countdown from synced `reconnectUntil`.

## Don't

- Call `room.send` from the page — only `game.sendMove`.
- Show white/red move chrome to spectators or non-current players.
- Depend tile fills on Quasar Dark / theme preference.
- Invent client-local seat assignment (server assigns on join).
- Treat client hints as authority.
- Sync selection / hints to schema.
- Put lobby subscribe/create/join logic into the board skill — use `work-with-lobby`.
- Put reconnect token / `rejoinGame` details here — use `work-with-rooms`.

## Related

- Server seating / turn / move: `.agents/skills/server/work-with-game/SKILL.md`
- Schema seats + turn: `.agents/skills/server/work-with-schema/SKILL.md`
- Lobby / room name: `.agents/skills/client/work-with-lobby/SKILL.md`
- Tourist reconnect: `.agents/skills/client/work-with-rooms/SKILL.md`
- Styles / theme chrome: `.agents/skills/client/work-with-styles/SKILL.md`
