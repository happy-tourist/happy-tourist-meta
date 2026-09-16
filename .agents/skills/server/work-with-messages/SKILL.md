---
name: work-with-messages
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room messages in
  the happy-tourist tourist server — onMessage handlers (move, peek, peekAnswer,
  endTurn, ready, say), private budgets/peekOpen sends, validating client intents
  before mutating schema state or broadcasting ephemeral events, optional error
  feedback, or aligning message contracts with the client. Not for lobby listing
  (LobbyRoom / HTTP fallback).
---

# Work With Messages

Use this skill for **room WebSocket messages** in `happy-tourist-server` (authoritative Colyseus room handlers).

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Coordinate with: `work-with-rooms` (lifecycle / registration), `work-with-schema` (synced state), `work-with-game` (rules / budgets), `server-work-with-errors` (reject / feedback style). Client bubbles UI: `.agents/skills/client/work-with-game-board`.

## Scope Boundary

| In scope | Out of scope |
|----------|--------------|
| `this.onMessage(...)` for game actions (`move`, `peek`, `peekAnswer`, `endTurn`, `ready`, `say`) | Lobby listing — client uses **LobbyRoom** (`rooms` / `+` / `-`) |
| Private `client.send('budgets'|'peekOpen', …)` to owner | Auth join gate — `onAuth` + JWT (`server-work-with-auth`) |
| Payload shape / type guards for client intents | Express routes / `createEndpoint` |
| Validate → mutate `@colyseus/schema` state (`move` / `peekAnswer` / `ready`) or ephemeral broadcast (`say`) | Pure board-game rules (`work-with-game` / `touristMove.ts`); free-form chat |
| Optional per-client error feedback for bad actions | |

**Prefer changing the server to match the client** rather than inventing a parallel protocol. Do not treat legacy draughts `move` `{ from, to }` as product canon.

## Client Contract (today)

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `move` | `{ side: 'N'\|'E'\|'S'\|'W', row: number, col: number }` — via `sendMove` when `phase === 'playing'`, `isMyTurn`, `!isMySeatFinished`, `!isMySeatTimeExpired`. Spends a step; **does not** advance turn |
| Client → server | `peek` | `{ side }` — via `sendPeek`; own unfinished piece on present `*`; server replies with private `peekOpen` |
| Client → server | `peekAnswer` | `{ correct: boolean }` — via `sendPeekAnswer`; Correct removes tile; Incorrect KEEP |
| Client → server | `endTurn` | empty — via `sendEndTurn` when `canSendEndTurn` (multi finite only) |
| Client → server | `ready` | empty — via `sendReady` (waiting, ≥2 seated, under maxSeats, not yet ready) |
| Client → server | `say` | `{ presetId: 'hello' \| 'luck' }` — via `sendSay`; whitelist only (no free text; **not** `ready`); finished seats may still say |
| Server → owner | `budgets` | `{ steps, peeks, infinite, peekedThisTurn }` — private; `infinite` = **peeks∞ only** (steps always finite); `peekedThisTurn` legacy (no gate) |
| Server → owner | `peekOpen` | `{ side, row, col, reward: 1\|2\|3 }` — private modal payload |
| Server → clients | Schema sync | `phase` / `maxSeats` / `countdownRemaining` / `seats` (+ `ready` / `finishPlace` / piece `finished`) / `currentTurnSessionId` / `removedTaskKeys` (+ `nextFinishPlace`) → `onStateChange` |
| Server → all clients | `say` | `broadcast('say', { sessionId, presetId, at })` — ephemeral; includes readiness preset from successful `ready` |
| Server → client | room error channel | Client sets `game.error` from `room.onError` |
| Lobby | HTTP fallback | `client.http.get('/rooms/tourist')` — **not** a room message |

**`move`:** accept only when `phase === 'playing'` + seated + `finishPlace === 0` + `!timeExpired` + current turn + `steps > 0`; legal one-step per `touristMove.ts` (occupancy ignores finished; **landing on removed holes rejected**; stand/leave OK). Reject (no mutate) otherwise. On accept: update piece `row`/`col`; always spend 1 step + `sendBudgets`; if target is center → `piece.finished = true` and maybe assign `finishPlace`; **do not** `advanceTurn` solely for move — then `maybeAutoEndTurn` / solo step-loss / finish-advance.

**`peek`:** current turn + peek budget (or solo peeks∞); **no** one-peek/turn gate; piece on present task cell; set room `openPeek` + `client.send('peekOpen', …)`. Silent reject otherwise.

**`peekAnswer`:** resolve open peek for that seat; spend peek (finite peeks only); Correct adds reward steps (always, including solo) and pushes `"r,c"` to `removedTaskKeys`; Incorrect KEEP tile + hidden reward; `sendBudgets`; then maybe auto-end / solo step-loss.

**`endTurn`:** multi (≥2 eligible) + current + not finished/expired; force-close open peek as incorrect KEEP if any; `advanceTurn` (next grant: multi +1/+1; solo become-current +1 step only). Solo peeks∞ → reject.

**`ready`:** accept only in `waiting`, seated + connected, seated count ≥ 2 and `< maxSeats`, seat not already ready. On accept: `seat.ready = true`, broadcast say preset `ready` (bypass live-say cap), maybe start countdown. Silent reject otherwise.

**`say`:** accept only when seated + `connected`; `presetId` ∈ `{ hello, luck }` (**reject** `ready` — use `ready` message). At most **3** live says per `sessionId` within **10s** TTL. Display: `hello` → «Всем привет», `luck` → «Удачи», `ready` → «Готов начать!» (client i18n).

## Server Today

- `src/rooms/MyRoom.ts` — seating + start + budgets/peek + `onMessage('move'|'peek'|'peekAnswer'|'endTurn'|'ready'|'say')`.
- Pure move/peek helpers: `src/game/touristMove.ts`.
- Room registered as `tourist` (+ `lobby` for live list); do not reintroduce `my_room`.

## Handler Pattern

Register handlers in the room (typically `onCreate`), not in Express:

```ts
this.onMessage('move', (client, message) => {
  // 1. Require phase === 'playing'
  // 2. Shape-check { side, row, col }
  // 3. Resolve seat; reject if finishPlace > 0 / timeExpired / no steps; require current turn
  // 4. Reject unknown/finished piece; validate via touristMove (occupancy ignores finished)
  // 5. On success: mutate row/col; spend step; if center → finished + maybe finishPlace
  // 6. Do NOT advance solely on move; maybeAutoEndTurn / finish-advance
});

this.onMessage('peek', (client, message) => {
  // 1. playing + current + peek budget (or solo peeks∞); no one-peek gate
  // 2. piece on present *; openPeek + client.send('peekOpen', { side, row, col, reward })
});

this.onMessage('peekAnswer', (client, message) => {
  // 1. Shape-check { correct: boolean }; must own openPeek
  // 2. resolveOpenPeek → spend peek (finite); Correct → +steps + remove tile; Incorrect KEEP; maybeAutoEndTurn / solo step-loss
});

this.onMessage('endTurn', (client) => {
  // 1. multi + current + eligible; force-close open peek as incorrect KEEP if any
  // 2. advanceTurn → applyTurnGrant (multi +1/+1; solo become-current +1 step)
});

this.onMessage('ready', (client) => {
  // 1. phase === 'waiting'; seated+connected; ≥2 and < maxSeats; !seat.ready
  // 2. seat.ready = true; broadcast say preset 'ready' (bypass live cap)
  // 3. maybeStartCountdown()
});

this.onMessage('say', (client, message) => {
  // 1. Seat exists + seat.connected
  // 2. presetId ∈ hello|luck (reject 'ready')
  // 3. Prune liveSays; reject if ≥ SAY_MAX_LIVE
  // 4. broadcast('say', { sessionId, presetId, at })
});
```

### Hard rules

1. **Do not trust the client.** Treat payloads as intent only.
2. **Validate, then mutate** schema state for gameplay (`move` / peek resolve); for `say`, validate then broadcast only — do **not** put bubbles in schema. Budgets stay room-private + `client.send`.
3. **Authoritative move/peek rules** live in `work-with-game` / `touristMove.ts`, not in the message payload.
4. **Do not** add HTTP endpoints for gameplay actions.
5. **No free-form say text** — whitelist `presetId` only.

## Error Feedback (align with `server-work-with-errors`)

| Situation | Preferred approach |
|-----------|-------------------|
| Malformed / illegal action (`move` / `peek` / `say` / …) | **Do not mutate state / do not broadcast.** Silent reject is OK. |
| Optional user-visible feedback | Prefer `room.onError` patterns the client already maps to `game.error`. Do not invent new message types unless the client is updated in the same change. |
| Auth / join failures | Not message-handlers — `onAuth` / join reject. |

## New Message Types

1. Confirm the sibling client will `room.send` / `room.onMessage` the same name and payload.
2. For schema-backed actions: validate → mutate schema — never trust client state. For ephemeral events like `say`: validate → broadcast; keep room-private bookkeeping (`budgets`, `taskRewards`, `openPeek`) out of schema.
3. Keep lobby listing on LobbyRoom / HTTP `GET /rooms/:roomName`.

## Do / Don't

| Do | Don't |
|----|--------|
| Keep `move` `{ side, row, col }` lockstep with client `sendMove` | Reintroduce draughts `move`/`{from,to}` without a product change |
| Keep `peek` / `peekAnswer` / `endTurn` + private `budgets` / `peekOpen` lockstep with store | Advance turn solely because `move` succeeded |
| Keep `say` `{ presetId }` + broadcast `{ sessionId, presetId, at }` lockstep with `sendSay` / `onMessage('say')` | Accept free-form text or unknown preset ids |
| Validate then mutate schema (`move` / remove tile on Correct only); broadcast-only for `say` | Trust client board hints; sync say bubbles or steps/peeks into schema |
| Treat lobby as LobbyRoom live list | Add a gameplay room message for room listing |
| Follow `server-work-with-errors` | Invent BFF-style error envelopes for room actions |

## Change Checklist

1. Handler registered on the tourist room via `this.onMessage`.
2. Payload matches the client store (`sendMove` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendReady` / `sendSay`).
3. Illegal actions do not mutate schema (and `say` rejects do not broadcast).
4. Move/peek rules delegated to `work-with-game` / `touristMove.ts`; start/ready/countdown in room; say stays I/O + whitelist + live-limit in the room handler.
5. Update `test/` / `loadtest/` when the message contract becomes testable.
6. Run `npm test` from the server package root; fix failures before claiming done.

## Related

- Room lifecycle / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Schema sync fields: `.agents/skills/server/work-with-schema/SKILL.md`
- Rules engine: `.agents/skills/server/work-with-game/SKILL.md`
- Reject / log / feedback: `.agents/skills/server/server-work-with-errors/SKILL.md`
- Client board: `.agents/skills/client/work-with-game-board/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
