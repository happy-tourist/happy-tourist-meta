---
name: work-with-checkers
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative russian
  checkers rules, board encoding, move validation, or synced board/turn/status
  updates in the happy-tourist Colyseus server — Room move handlers, schema
  state, or pure rules modules.
---

# Work With Checkers

Use this skill for **authoritative russian checkers** (board, moves, validation, state updates) in `happy-tourist-server`.

Server owns truth; client highlights are UI-only. Do not trust client board or `getTargets`. Target ruleset: **standard russian draughts (русские шашки)** — mark product-specific choices as open questions when not yet fixed in code.

Pair with client board UX: `../happy-tourist.github.io/.agents/skills/client/work-with-game-board/SKILL.md` (hints only; this package remains source of truth).

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` (intended rename/align with `checkers`) | Lifecycle, `onMessage('move')`, apply validated results to state |
| Schema | `src/rooms/schema/MyRoomState.ts` | Synced `board`, `currentTurn`, `status`, `players` |
| Rules (preferred) | new pure module e.g. `src/rooms/checkers/` or `src/game/` | Validate / apply moves without Colyseus I/O |
| Registration | `src/app.config.ts` | Room name must be `checkers` for client lobby |

Scaffold today: room/schema are stubs (`my_room`, `mySynchronizedProperty`). Guide implementation toward the client contract below; do not treat unimplemented behavior as shipped product fact.

## Authority

- Server state is the only board truth. Reject illegal moves; leave state unchanged.
- Client `getTargets` / selection highlights are **UI hints** (simplified; may omit multi-jump chains / flying kings). Never mirror client-only logic as authority.
- Wire move: `room.send('move', { from, to })` with `{ row, col }` (0–7). No alternate move protocols without client lockstep.
- Do not put rules in Express HTTP routes.

## Cell Values And Board

| Value | Meaning |
| --- | --- |
| `0` | empty |
| `1` | white man |
| `2` | black man |
| `3` | white king |
| `4` | black king |

- Board: 8×8, values `0`–`4`. Prefer a representation easy to sync (e.g. flat `ArraySchema` of 64 cells or nested arrays — match whatever schema style the room adopts; keep encoding identical to client).
- Coordinates: `{ row, col }` with `0…7`. Align indexing with client `GamePage` / `stores/game.ts` (`board[row][col]`).

### Russian draughts board conventions (target)

- Play only on **dark** squares. Client UI treats dark as `(row + col) % 2 === 1` — prefer the same playable-square rule server-side unless product changes both ends.
- **Open question (product convention not fully fixed in server code):** white at bottom (high `row`) moving toward decreasing `row`, black toward increasing `row`. Client hint `getTargets` already assumes white `dr = -1`, black `dr = +1`. Adopt that unless product decides otherwise — document any flip in both repos.
- Initial setup: standard russian placement on dark squares (three back ranks each). Exact initial array is an implementation detail; must match client expectations once defined.

## Client Contract (align server to this)

| Field / message | Client expectation |
| --- | --- |
| Room name | `checkers` |
| `board` | 8×8 cells `0`–`4` |
| `currentTurn` | `'white' \| 'black'` |
| `status` (synced) | `'waiting' \| 'playing' \| 'finished'` |
| `players[sessionId].color` | `'white' \| 'black'` |
| `move` | `{ from: { row, col }, to: { row, col } }` |

Client-local only (do **not** put on schema): `idle`, `connecting` in Pinia. Server should emit `waiting` → `playing` → `finished`.

Client `canMove`: `status === 'playing' && myColor && currentTurn === myColor`.

## Move Validation (authoritative)

When handling `move`, validate in order; on any failure ignore / optionally notify without mutating board:

1. **Status** — only while `playing`.
2. **Turn** — `currentTurn` matches the seat color of `client.sessionId`.
3. **Ownership** — `from` holds a piece of that color (`1`/`3` white, `2`/`4` black).
4. **Bounds / square** — `from`/`to` in 0–7; `to` empty; move stays on playable (dark) squares.
5. **Legality under russian draughts (target):**
   - **Quiet man move:** one diagonal step forward (direction per color convention above).
   - **Man capture:** jump over adjacent opponent to empty landing; in russian rules men **may capture backward**.
   - **Multi-jump:** if russian rules require continuing captures in the same turn, either accept a sequence of `move` messages that stay “in capture chain” for that side, or define an explicit chain protocol — **open question** until product picks one; do not invent a second wire shape without client sync. Prefer documenting chosen approach in code comments.
   - **Mandatory capture:** standard russian draughts requires capturing when possible (and typically maximizing). Treat as **target rule**; if product ships a softer mode, mark it explicitly — do not silently drop obligation.
   - **Kings (дамки):** target = **flying kings** (long diagonal moves/captures). Client hints today only show adjacent steps — server must still enforce the real rules once implemented; UI may lag.
   - **Promotion:** man reaching the last rank becomes king (`1→3`, `2→4`) per russian promotion timing (including mid-capture rules if multi-jump applies) — follow standard russian draughts; note mid-capture promotion edge cases as implementation checklist items, not invent ad-hoc product law.

Prefer returning a structured result from pure functions (`ok` + next board / turn / status, or `error` reason) so the Room only applies success paths.

## State Updates After A Legal Move

On success, update synced state atomically from the rules result:

1. **board** — new cell values (moved piece, removed captured, promotion).
2. **currentTurn** — opposite color, unless a multi-jump continuation keeps the same side (if that model is chosen).
3. **status** — stay `playing` until terminal; then `finished` (no legal moves / no pieces / resign — resign may be a separate message later; do not invent unless specified).

Also keep seat map `players` consistent on join (`waiting` until two players → `playing`, assign colors). Disconnect / forfeit policy: **open question** (scaffold comments only today).

## Architecture Preference

```
onMessage('move') → parse payload → pure validate/apply(board, turn, move, color)
                 → if ok: write schema fields
```

- Keep **pure functions** for rules (testable without Colyseus) separate from Room I/O (`onAuth`, join/leave, `onMessage`).
- Room: auth, seats, timeouts, applying results, broadcasting via schema.
- Do not duplicate a second authoritative engine on the client.

## Implementation Checklist (scaffold → product)

- [ ] Register room as `checkers`; replace scaffold schema with board/turn/status/players.
- [ ] Initial board + `status: waiting` / `playing` when both seated; set `currentTurn` (usually white).
- [ ] `onMessage('move', …)` wired to pure rules.
- [ ] Captures, promotion, turn flip, `finished` detection.
- [ ] Unit-test pure rules; update `test/MyRoom.test.ts` / loadtest room name.
- [ ] Confirm orientation and capture-obligation choices with client if still open.

## Do

- Align room name, encoding `0…4`, `{ from, to }`, and status strings with the client.
- Reject illegal moves server-side even if the client highlighted the square.
- Prefer pure rules modules + thin Room glue.
- State russian draughts standards as the **target**; label undecided product choices as open questions.

## Don't

- Trust client highlights or locally mutated boards.
- Invent unsupported product rules (custom king length, optional captures, alternate payloads) and present them as hard fact.
- Put authoritative logic only in Express routes or only in the Vue `getTargets` helper.
- Change cell encoding or move shape without updating the client in lockstep.

## Related

- Client board skill: `../happy-tourist.github.io/.agents/skills/client/work-with-game-board/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `AGENTS.md` (Current vs client contract)
