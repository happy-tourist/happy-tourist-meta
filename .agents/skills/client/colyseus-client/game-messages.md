# Tourist room messages (knowledge economy)

Read this when changing create options, `room.send` / `onMessage` game contracts, or private knowledge sync. Core Colyseus client wiring: [SKILL.md](SKILL.md). Board UX: `work-with-game-board`. Authority: server `work-with-messages` / `work-with-game`.

## Create / join

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `create('tourist', opts)` | `{ mapId, packId, taskSetIds, maxSeats, opponentMode: 'people'\|'ai' }` — omit/`people` → human table; effective AI only when `ai` ∧ maxSeats≥2 (server coerce; private + bots; join deny). **no** `grilleDensity` / `catapultDensity` (server ignores if sent; fixed exclusive seed **15% grille / 20% catapult** — SC-LOBBY-40/41 / SC-BOARD-80/81) |
| Client → server | `joinById(roomId)` | Tourist join (spectator if not waiting / full); effective AI rooms deny extra humans (`ai_room_join_denied`) |

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
| `knowledge` (owner) | `{ delivered, pieces: Record<pieceId, number>, outcome?: 'finish'\|'cut' }` | Private; avatar counter + strip TR yellow badges (+ lightbulb). Arm +N fall only after `knowledgeSynced` (SC-PRESENCE-51). `outcome` selects finish\|cut modal copy (SC-FINISH-31/32/34). |
| `allJailWarning` (owner) | `{}` | **Inert** — keep `allJailWarning = false`; teleport modal retired (SC-MOVE-119). |
| `allJailSkip` (all) | `{}` | Top-center Notify «Все туристы в решётке — ход пропущен» for every client; never open start-reset modal (SC-PRESENCE-57 / D7). |
| `peekClose` (all) | `{ sessionId, correct, row, col, reason?: 'wrong'\|'timeout' }` | Header flash + top-center Notify toast (SC-PRESENCE-36 / SC-BOARD-88). Multi turn timeout uses `'timeout'`. |
| `say` (all) | `{ sessionId, presetId, at }` | Ephemeral. |
| Schema sync | seats/pieces (+ `remainingTurns`, `isBot`), peek session, traps, `turnUntil` / `turnBudgetSeconds`, holes | `onStateChange` — see `work-with-stores/game.md`. |

## Store mirrors (Pinia `game`)

- Private: `knowledgeDelivered`, `knowledgePieces`, `knowledgeOutcome`, `knowledgeSynced` (not `steps`/`peeks`/`budgetsInfinite`).
- Public piece: `remainingTurns` (grille countdown 3…1; 0 when free).
- Turn ring: mirror `turnUntil` / `turnBudgetSeconds` (60s multi when ≥2 eligible).
- Local: board-action spent flag after successful `sendMove` / `sendReturnFromFinish`; clear when `currentTurnSessionId` leaves own seat.
- `sendPush` / `sendEndTurn` / `sendPeek` stay as no-ops or absent from UI — do not revive affordances.

## Do / Don't

- Do: Colyseus I/O only via `stores/game`; keep create payload density-free; listen `knowledge` not `budgets`; Notify on `peekClose` and `allJailSkip`.
- Don't: revive density create keys, push/end-turn/peek-open affordances, steps/peeks HUD, board yellow knowledge badge, all-jail teleport modal, or treat `finishPlace` as ranked place UX (`finishPlace` 0→1 opens delivered-knowledge modal by `outcome`).
