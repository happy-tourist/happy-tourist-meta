---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the tourist board UI in
  the happy-tourist client — GamePage CSS Grid layout, synced seat pieces
  overlay, personal tourist strip, tile kinds (start/task/center), or
  non-interactive board rendering for room tourist.
---

# Work With Game Board

Use this skill for the **tourist board UI** on Game in the Vue 3 Quasar client (`happy-tourist.github.io`).

Product: настольная игра «Счастливый турист». The board and pieces are **non-interactive** (no selection, targets, or `sendMove`). Seats come from synced room state; move rules remain later on the server (`work-with-game`).

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Game page | `src/pages/GamePage.vue` | CSS Grid tourist field; overlay all seats’ pieces; strip×4 «Мой турист» if seated; leave → lobby |
| Game store | `src/stores/game.ts` | Room join/leave; mirror `seats` / `started` / `sessionId` from `onStateChange` |

| Concern | Location |
| --- | --- |
| Layout constant | `LAYOUT` string grid in `GamePage.vue` (`.` hole, `1` start, `*` task, `7` center) |
| Tile build | `buildBoardTiles()` → non-button `div.tile` with `gridColumn` / `gridRow` |
| Pieces | Flatten all seats’ `pieces` (4 per seated player) → `img.piece` at `row`/`col`; PNG from seat `touristId` → `@/assets/tourists/touristN.png` |
| Strip | Below board only when `mySeat`: **four** slots `N→E→S→W` (same PNG; status chrome later); spectators: pieces yes, strip no |
| Center | One element with `span 2` / `span 2` (solid 2×2), not four cells |
| Room enter | Out of scope — see `work-with-lobby` / `work-with-rooms` |

## Layout And Tiles

- Grid: **10×10** sparse; empty corners are holes (no tile element; page background shows through).
- Kinds: `start` (green), `task` (brown), `center` (yellow).
- `.tourist-board`: `width: 100%`, `max-width: calc(10 * 60px + 9 * 6px)`, `aspect-ratio: 1`, `gap: 6px`, `grid-template-*: repeat(10, 1fr)`; tile `border-radius: 12px`. Do **not** size rows with `%` of auto height (tracks collapse to 0).
- Container: full width of Game content; mobile edge-to-edge relative to page content; wide screens capped by max tile 60px (via board max-width + square aspect).
- Tile colors are **fixed fills**, independent of Quasar Dark chrome (see `work-with-styles` / theme specs).

Tiles and pieces are non-interactive — not `button`s, no `@click`, no selection/target classes (`pointer-events: none` on board/pieces).

## Authority

- Board **geometry** (`LAYOUT`) is a client constant; server does not sync tile kinds.
- **Seats** (`touristId` + exactly four `pieces` `{ side, row, col }` keyed N/E/S/W) and `started` are server-authoritative; client only mirrors and renders.
- Do not reintroduce legacy draughts CellValue `0…4`, `getTargets`, `selected` / `targets`, or `sendMove` on Game.
- When move rules land, coordinate wire protocol with server `work-with-game` — do not invent a second client-only rules engine.

## GamePage Responsibilities

- Render `boardTiles` from `LAYOUT`.
- Overlay **all** pieces of **all** seats at `row`/`col` (0-based schema → 1-based CSS Grid).
- Show strip «Мой турист» only if `mySeat`: four imgs of that seat’s `touristId` in order `N,E,S,W` (1:1 with field sides); spectators: board pieces yes, strip no.
- Status from store (`waiting` / `playing`); leave → `leaveGame` + lobby; rejoin by `roomId` via store.

## Do

- Keep layout in one client constant; center as a single 2×2 grid area.
- Preserve max tile 60px, gap 6, radius 12, hole = page background.
- Keep Colyseus I/O in `stores/game`; page reads seats/strip from store only.
- Map `touristId` 1…4 to `tourist{N}.png`; strip shows only the local player’s four slots (not other kinds).

## Don't

- Add click handlers, move highlights, or draughts piece UX.
- Depend tile fills on Quasar Dark / theme preference.
- Invent client-local seat assignment (server assigns on join).
- Sync or invent server board encoding for the static layout without a product change.
- Put lobby subscribe/create/join logic into the board skill — use `work-with-lobby`.

## Related

- Server seating / rules: `.agents/skills/server/work-with-game/SKILL.md`
- Schema seats: `.agents/skills/server/work-with-schema/SKILL.md`
- Lobby / room name: `.agents/skills/client/work-with-lobby/SKILL.md`
- Styles / theme chrome: `.agents/skills/client/work-with-styles/SKILL.md`
