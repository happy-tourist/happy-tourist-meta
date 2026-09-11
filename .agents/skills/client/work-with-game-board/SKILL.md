---
name: work-with-game-board
description: >-
  Use when creating, changing, reviewing, or debugging the checkers board UI and
  moves in the happy-tourist client — GamePage cell rendering, piece selection,
  local move highlights, canMove, sendMove / room.send('move'), or board
  CellValue encoding.
---

# Work With Game Board

Use this skill for the checkers **board UI and move UX** in the Vue 3 Quasar client (`happy-tourist.github.io`).

Board truth is SERVER; client highlights are UI hints only. Do not invent alternate move protocols without server sync.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Game page | `src/pages/GamePage.vue` | 8×8 board, selection, local targets, click → `sendMove` |
| Game store | `src/stores/game.ts` | `board`, `myColor`, `currentTurn`, `status`, `canMove`, `sendMove` |

| Concern | Location |
| --- | --- |
| Cell / board types | `CellValue`, `Board` in `stores/game.ts` |
| Turn gate | getter `canMove` |
| Wire protocol | `sendMove` → `room.send('move', { from, to })` |
| Local UI state | `selected` / `targets` refs in `GamePage` |

Room join / leave / lobby listing are out of scope unless they affect board state sync.

## Cell Values

| Value | Meaning |
| --- | --- |
| `0` | empty |
| `1` | white |
| `2` | black |
| `3` | white king |
| `4` | black king |

Type: `CellValue = 0 | 1 | 2 | 3 | 4`. Board: `CellValue[][]` (8×8).

Piece CSS classes on GamePage:

- white: `1` or `3`
- black: `2` or `4`
- king: `3` or `4`

## Authority And Protocol

- Server state (`room.onStateChange`) owns `board`, `currentTurn`, `status`, and `players[sessionId].color`.
- Client may highlight possible landing squares via `getTargets` — that is a **hint**, not legal-move enforcement.
- Only move message today: `room.send('move', { from, to })` where `from` / `to` are `{ row, col }`.
- Do not invent alternate move protocols without server sync (no extra message types, payloads, or optimistic board mutation as source of truth).

## canMove

Getter in `stores/game.ts`:

```ts
canMove: (state) =>
  state.status === 'playing' && state.myColor !== null && state.currentTurn === state.myColor,
```

Meaning: `status===playing && myColor && currentTurn===myColor`.

Use `game.canMove` to:

- Gate clicks in `onCellClick` (`if (!game.canMove) return`).
- Drive turn copy (“Ваш ход” / “Ход соперника”).
- Apply board disabled class when `!canMove`.

## sendMove

```ts
sendMove(from: { row: number; col: number }, to: { row: number; col: number }) {
  if (!this.room) {
    return;
  }
  this.room.send('move', { from, to });
}
```

Flow: `sendMove(from, to)` → `room.send('move', { from, to })`.

Page clears `selected` / `targets` after calling `sendMove`. Board updates arrive via `onStateChange`, not from local rewrite.

## Local Selection And Highlights

Page-local refs (not in Pinia):

```ts
const selected = ref<{ row: number; col: number } | null>(null);
const targets = ref<Array<{ row: number; col: number }>>([]);
```

| Helper | Role |
| --- | --- |
| `isOwnPiece` | Own color only (white: 1/3, black: 2/4) |
| `getTargets` | Client-side hint: quiet steps + single-jump captures (russian checkers, simplified) |
| `isTarget` | Whether cell is in `targets` |
| `onCellClick` | Select own piece → retarget / deselect → if target, `sendMove` |

`selected` / `targets` local refs; `getTargets` is client-side hint. Recompute targets on deep `watch` of `game.board` while something is selected. Clear both on unmount.

Template classes:

- `.selected` on the chosen cell
- `.target` when `isTarget(r, c)`
- `.board.disabled` when `!game.canMove` (`opacity: 0.92`)

## Click Flow

1. If `!canMove` → ignore.
2. No selection → select only if `isOwnPiece`; set `targets = getTargets(...)`.
3. Click same cell → clear selection.
4. Click another own piece → reselect + new targets.
5. Click non-target → ignore.
6. Click target → `game.sendMove(selected, { row, col })`, then clear selection.

## Do

- Keep Colyseus I/O in `stores/game` (`sendMove`); page only calls the store.
- Treat `getTargets` as optional UX; server may reject illegal moves.
- Preserve cell encoding `0…4` and `{ from, to }` move shape unless server schema changes in lockstep.
- Disable interaction visually and logically via `canMove`.

## Don't

- Mutate `game.board` locally to “apply” a move — wait for server state.
- Bypass `canMove` or send moves when not the player’s turn.
- Invent alternate move protocols without server sync.
- Move `selected` / `targets` into the store unless shared UI needs them.
- Duplicate full rules engine on the client as authority.
