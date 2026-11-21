# Tourist room messages (knowledge economy)

Read this when changing create options, `room.send` / `onMessage` game contracts, or private knowledge sync. Core Colyseus client wiring: [SKILL.md](SKILL.md). Board UX: `work-with-game-board`. Authority: server `work-with-messages` / `work-with-game`.

## Create / join

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `create('tourist', opts)` | `{ mapId, packId, taskSetIds, maxSeats }` — **no** `grilleDensity` / `catapultDensity` (server ignores if sent; fixed exclusive seed 12% grille / 15% catapult — SC-LOBBY-40/41 / SC-BOARD-80/81) |
| Client → server | `joinById(roomId)` | Tourist join (spectator if not waiting / full) |

## Client → server

| Name | Payload | Behavior |
|------|---------|----------|
| `move` | `{ pieceId, row, col }` via `sendMove` | One board action per turn (`boardActionUsed`); land → paced traps → open-on-land Q&A **or** `advanceTurn`. No steps budget. |
| `rescue` | `{ pieceId }` via `sendRescue` | Free before board action; does **not** spend board action; clears grille → empty cell. |
| `returnFromFinish` | `{ pieceId, row, col }` via `sendReturnFromFinish` | Counts as board action (like move). |
| `peekPlace` / `peekSubmit` | slot / `{}` | Peeker only while shared session open (land-driven). |
| `ready` / `say` | empty / `{ presetId }` | Unchanged waiting ready + ephemeral presets. |
| `push` | — | **Retired** — `sendPush` always no-op; server rejects (SC-MOVE-114). |
| `peek` | — | **Retired as open trigger** — server rejects; modal opens only on land (SC-BOARD-84). |
| `endTurn` | — | **Retired** — `canSendEndTurn` always false; server rejects (SC-PRESENCE-55). |

## Server → client

| Name | Payload | Behavior |
|------|---------|----------|
| `knowledge` (owner) | `{ delivered: number, pieces: Record<pieceId, number> }` | Private; avatar counter + yellow BR badges. Arm +N fall only after `knowledgeSynced` (SC-PRESENCE-51). |
| `allJailWarning` (owner) | `{}` | Informational modal after all-jail reset. |
| `peekClose` (all) | `{ sessionId, correct, row, col, reason?: 'wrong' }` | No turn-timeout `'timeout'` path (deadlines retired). Header flash from reason (SC-PRESENCE-36). |
| `say` (all) | `{ sessionId, presetId, at }` | Ephemeral. |
| Schema sync | seats/pieces (+ `remainingTurns`), peek session, traps, turn, holes | `onStateChange` — see `work-with-stores/game.md`. |

## Store mirrors (Pinia `game`)

- Private: `knowledgeDelivered`, `knowledgePieces`, `knowledgeSynced` (not `steps`/`peeks`/`budgetsInfinite`).
- Public piece: `remainingTurns` (grille countdown 3…1; 0 when free).
- Local: board-action spent flag after successful `sendMove` / `sendReturnFromFinish`; clear when `currentTurnSessionId` leaves own seat.
- `sendPush` / `sendEndTurn` / `sendPeek` stay as no-ops or absent from UI — do not revive affordances.

## Do / Don't

- Do: Colyseus I/O only via `stores/game`; keep create payload density-free; listen `knowledge` not `budgets`.
- Don't: revive density create keys, push/end-turn/peek-open affordances, steps/peeks HUD, or treat `finishPlace` as ranked place UX (`finishPlace` 0→1 opens delivered-knowledge modal only).
