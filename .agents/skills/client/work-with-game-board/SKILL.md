---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the static tourist board
  UI in the happy-tourist client — GamePage CSS Grid layout, tile kinds
  (start/task/center), sizes/colors, sparse holes, or non-interactive board
  rendering for room tourist.
---

# Work With Game Board

Use this skill for the **static tourist board UI** on Game in the Vue 3 Quasar client (`happy-tourist.github.io`).

Product: настольная игра «Счастливый турист». The board is **non-interactive** (no piece selection, targets, or `sendMove`). Game rules and synced board authority come later on the server (`work-with-game`).

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Game page | `src/pages/GamePage.vue` | Static CSS Grid tourist field; status label; leave → lobby |
| Game store | `src/stores/game.ts` | Room join/leave / lobby wiring (`TOURIST_ROOM`); not board geometry |

| Concern | Location |
| --- | --- |
| Layout constant | `LAYOUT` string grid in `GamePage.vue` (`.` hole, `1` start, `*` task, `7` center) |
| Tile build | `buildBoardTiles()` → non-button `div.tile` with `gridColumn` / `gridRow` |
| Center | One element with `span 2` / `span 2` (solid 2×2), not four cells |
| Room enter | Out of scope — see `work-with-lobby` / `work-with-rooms` |

## Layout And Tiles

- Grid: **10×10** sparse; empty corners are holes (no tile element; page background shows through).
- Kinds: `start` (green), `task` (brown), `center` (yellow).
- CSS vars on `.tourist-board`: `--tile: min(60px, calc((100% - 9 * var(--gap)) / 10))`, `--gap: 6px`; `border-radius: 12px` on tiles.
- Container: full width of Game content; mobile edge-to-edge relative to page content; wide screens capped by max tile 60px.
- Tile colors are **fixed fills**, independent of Quasar Dark chrome (see `work-with-styles` / theme specs).

Tiles are non-interactive `div`s — not `button`s, no `@click`, no selection/target classes.

## Authority

- Board **geometry** is a client constant for this phase; server does not sync the tourist layout yet.
- Do not reintroduce legacy draughts CellValue `0…4`, `getTargets`, `selected` / `targets`, or `sendMove` on Game.
- When rules land, coordinate wire protocol with server `work-with-game` — do not invent a second client-only rules engine.

## GamePage Responsibilities

- Render `boardTiles` from `LAYOUT`.
- Show connection/status copy (neutral until rules exist).
- Leave → store `leaveGame` + navigate lobby; rejoin by `roomId` on refresh via store.

## Do

- Keep layout in one client constant; center as a single 2×2 grid area.
- Preserve max tile 60px, gap 6, radius 12, hole = page background.
- Keep Colyseus I/O in `stores/game`; page does not call `client.*` for board.

## Don't

- Add click handlers, move highlights, or piece rendering from the old draughts UX.
- Depend tile fills on Quasar Dark / theme preference.
- Sync or invent server board encoding for the static layout without a product change.
- Put lobby subscribe/create/join logic into the board skill — use `work-with-lobby`.

## Related

- Server rules (later): `.agents/skills/server/work-with-game/SKILL.md`
- Lobby / room name: `.agents/skills/client/work-with-lobby/SKILL.md`
- Styles / theme chrome: `.agents/skills/client/work-with-styles/SKILL.md`
