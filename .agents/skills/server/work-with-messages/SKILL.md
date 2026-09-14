---
name: work-with-messages
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room messages in
  the happy-tourist tourist server — onMessage handlers (move, say), validating
  client intents before mutating schema state or broadcasting ephemeral events,
  optional error feedback, or aligning message contracts with the client. Not
  for lobby listing (LobbyRoom / HTTP fallback).
---

# Work With Messages

Use this skill for **room WebSocket messages** in `happy-tourist-server` (authoritative Colyseus room handlers).

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Coordinate with: `work-with-rooms` (lifecycle / registration), `work-with-schema` (synced state), `work-with-game` (rules), `server-work-with-errors` (reject / feedback style). Client bubbles UI: `.agents/skills/client/work-with-game-board`.

## Scope Boundary

| In scope | Out of scope |
|----------|--------------|
| `this.onMessage(...)` for game actions (`move`, `say`) | Lobby listing — client uses **LobbyRoom** (`rooms` / `+` / `-`) |
| Payload shape / type guards for client intents | Auth join gate — `onAuth` + JWT (`server-work-with-auth`) |
| Validate → mutate `@colyseus/schema` state (`move`) or ephemeral broadcast (`say`) | Express routes / `createEndpoint` |
| Optional per-client error feedback for bad actions | Pure board-game rules (`work-with-game` / `touristMove.ts`); free-form chat |

**Prefer changing the server to match the client** rather than inventing a parallel protocol. Do not treat legacy draughts `move` `{ from, to }` as product canon.

## Client Contract (today)

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `move` | `{ side: 'N'\|'E'\|'S'\|'W', row: number, col: number }` — via game store `sendMove` only when `isMyTurn` |
| Client → server | `say` | `{ presetId: 'hello' \| 'luck' }` — via game store `sendSay`; whitelist only (no free text) |
| Server → clients | Schema sync | `started` + `seats` + `currentTurnSessionId` → client `onStateChange` |
| Server → all clients | `say` | `broadcast('say', { sessionId, presetId, at })` — ephemeral; not schema |
| Server → client | room error channel | Client sets `game.error` from `room.onError` |
| Lobby | HTTP fallback | `client.http.get('/rooms/tourist')` — **not** a room message |

**`move`:** accept only when seated + current turn; legal one-step per `touristMove.ts`. Reject (no mutate) otherwise. On accept: update piece `row`/`col`, advance turn.

**`say`:** accept only when seated + `connected` (not spectator / offline grace); `presetId` ∈ `{ hello, luck }`; at most **3** live says per `sessionId` within **10s** TTL (`SAY_TTL_MS` / `SAY_MAX_LIVE` in `MyRoom.ts`). Room-private `liveSays: { sessionId, at }[]` (not schema). Silent reject otherwise. On accept: `broadcast('say', { sessionId, presetId, at })` where `at` is server ms timestamp. Turn ownership is **not** required. Display strings live on the client only (`hello` → «Всем привет», `luck` → «Удачи»).

## Server Today

- `src/rooms/MyRoom.ts` — seating lifecycle + `onMessage('move')` → `handleMove`; `onMessage('say')` → `handleSay`.
- Pure move rules: `src/game/touristMove.ts`.
- Room registered as `tourist` (+ `lobby` for live list); do not reintroduce `my_room`.

## Handler Pattern

Register handlers in the room (typically `onCreate`), not in Express:

```ts
this.onMessage('move', (client, message) => {
  // 1. Shape-check { side, row, col }
  // 2. Resolve seat; require currentTurnSessionId === client.sessionId
  // 3. Validate/apply via touristMove pure rules
  // 4. On success: mutate piece row/col + advance turn
  // 5. On failure: do not mutate; silent reject OK
});

this.onMessage('say', (client, message) => {
  // 1. Seat exists + seat.connected
  // 2. presetId ∈ whitelist hello|luck
  // 3. Prune liveSays by SAY_TTL_MS; reject if ≥ SAY_MAX_LIVE for sessionId
  // 4. Push { sessionId, at }; broadcast('say', { sessionId, presetId, at })
  // 5. Never mutate schema for say
});
```

### Hard rules

1. **Do not trust the client.** Treat payloads as intent only.
2. **Validate, then mutate** schema state for gameplay (`move`); for `say`, validate then broadcast only — do **not** put bubbles in schema.
3. **Authoritative move rules** live in `work-with-game` / `touristMove.ts`, not in the message payload.
4. **Do not** add HTTP endpoints for gameplay actions.
5. **No free-form say text** — whitelist `presetId` only.

## Error Feedback (align with `server-work-with-errors`)

| Situation | Preferred approach |
|-----------|-------------------|
| Malformed / illegal action (`move` or `say`) | **Do not mutate state / do not broadcast.** Silent reject is OK. |
| Optional user-visible feedback | Prefer `room.onError` patterns the client already maps to `game.error`. Do not invent new message types unless the client is updated in the same change. |
| Auth / join failures | Not message-handlers — `onAuth` / join reject. |

## New Message Types

1. Confirm the sibling client will `room.send` / `room.onMessage` the same name and payload.
2. For schema-backed actions: validate → mutate schema — never trust client state. For ephemeral events like `say`: validate → broadcast; keep room-private bookkeeping out of schema.
3. Keep lobby listing on LobbyRoom / HTTP `GET /rooms/:roomName`.

## Do / Don't

| Do | Don't |
|----|--------|
| Keep `move` `{ side, row, col }` lockstep with client `sendMove` | Reintroduce draughts `move`/`{from,to}` without a product change |
| Keep `say` `{ presetId }` + broadcast `{ sessionId, presetId, at }` lockstep with `sendSay` / `onMessage('say')` | Accept free-form text or unknown preset ids |
| Validate then mutate schema (`move`); broadcast-only for `say` | Trust client board hints; sync say bubbles into schema |
| Treat lobby as LobbyRoom live list | Add a gameplay room message for room listing |
| Follow `server-work-with-errors` | Invent BFF-style error envelopes for room actions |

## Change Checklist

1. Handler registered on the tourist room via `this.onMessage`.
2. Payload matches the client store (`sendMove` / `sendSay`).
3. Illegal actions do not mutate schema (and `say` rejects do not broadcast).
4. Move rules delegated to `work-with-game` / `touristMove.ts`; say stays I/O + whitelist + live-limit in the room handler.
5. Update `test/` / `loadtest/` when the message contract becomes testable.
6. Run `npm test` from the server package root; fix failures before claiming done.

## Related

- Room lifecycle / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Schema sync fields: `.agents/skills/server/work-with-schema/SKILL.md`
- Rules engine: `.agents/skills/server/work-with-game/SKILL.md`
- Reject / log / feedback: `.agents/skills/server/server-work-with-errors/SKILL.md`
- Client board: `.agents/skills/client/work-with-game-board/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
