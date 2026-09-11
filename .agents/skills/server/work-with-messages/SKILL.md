---
name: work-with-messages
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room messages in
  the happy-tourist checkers server — especially `move` (`from`/`to` `{row,col}`),
  `this.onMessage(...)`, validating client intents before mutating schema state,
  optional move-error feedback, or aligning the message contract with the client
  `stores/game.ts` protocol. Not for lobby listing (LobbyRoom / HTTP fallback).
---

# Work With Messages

Use this skill for **room WebSocket messages** in `happy-tourist-server` (authoritative Colyseus room handlers).

**Temp path:** this skill lives under `.agents/skills/server/` in this repo for now. Canonical copy may later move to `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Coordinate with: `work-with-rooms` (lifecycle / registration), `work-with-schema` (synced state), `work-with-checkers` (rules), `server-work-with-errors` (reject / feedback style).

## Scope Boundary

| In scope | Out of scope |
|----------|--------------|
| `this.onMessage('move', …)` and future room message types | Lobby listing — client uses **LobbyRoom** (`rooms` / `+` / `-`), not a gameplay room message |
| Payload shape / type guards for client intents | Auth join gate — `onAuth` + JWT (`server-work-with-auth`) |
| Validate → mutate `@colyseus/schema` state | Express routes / `createEndpoint` |
| Optional per-client error feedback for bad moves | Pure draughts rules implementation details (`work-with-checkers`) |

**Prefer changing the server to match the client** (`../happy-tourist.github.io/src/stores/game.ts`) rather than inventing a parallel protocol.

## Client Contract (source of truth)

From sibling `stores/game.ts`:

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `move` | `room.send('move', { from, to })` where `from` / `to` are `{ row: number; col: number }` |
| Server → clients | *(synced state)* | Board / turn / status / seats via schema `onStateChange` — not a reply message for success |
| Server → client | room error channel | Client sets `game.error` from `room.onError((_code, message) => …)` |
| Lobby | HTTP only | `client.http.get('/rooms/checkers')` — **not** a room message |

```ts
// Client (do not change unilaterally)
sendMove(from: { row: number; col: number }, to: { row: number; col: number }) {
  if (!this.room) return;
  this.room.send('move', { from, to });
}
```

Today the client does **not** listen for a custom `onMessage('moveError')` (or similar). Successful play is observed only through state sync. Keep server message names and payloads aligned with that store.

## Server Today

- `src/rooms/MyRoom.ts` — **no** `this.onMessage('move', …)` yet (scaffold).
- Room registered as `checkers` (+ `lobby` for live list); do not reintroduce `my_room` (registration is `work-with-rooms` / `app.config.ts`, not a message type).

## Handler Pattern

Register handlers in the room (typically `onCreate`), not in Express:

```ts
this.onMessage('move', (client, message) => {
  // 1. Shape-check message
  // 2. Resolve seat / color for client.sessionId
  // 3. Validate with checkers rules (turn, ownership, legality)
  // 4. On success: mutate this.state (board, currentTurn, status, …)
  // 5. On failure: do not mutate; follow error guidance below
});
```

### Hard rules

1. **Do not trust the client.** Treat `from` / `to` as an intent only. Never apply a move because the client “already moved” locally — client highlights are UI-only.
2. **Validate, then mutate** schema state (`work-with-schema`). Invalid or out-of-turn moves leave state unchanged so all clients stay consistent via sync.
3. **Authoritative rules** live in room / checkers helpers (`work-with-checkers`), not in the message payload.
4. **Do not** add HTTP endpoints for moves. Realtime intents stay on the room message channel.

### Suggested validation order

1. Room `status === 'playing'` (ignore or reject if `waiting` / `finished`).
2. Message shape: `from` / `to` objects with finite integer `row` / `col` in board bounds (0–7 for 8×8).
3. Sender has a seat (`players[sessionId].color`) and it is their `currentTurn`.
4. Piece at `from` belongs to that color; destination legal per russian checkers rules (including captures / multi-jump when required).
5. Apply to `board`, then update `currentTurn` / `status` as rules dictate.

## Error Feedback (align with `server-work-with-errors`)

There is **no** BFF `ServiceError` / `{ errorCode, errorMessage }` envelope on this server.

| Situation | Preferred approach |
|-----------|-------------------|
| Malformed payload / not your turn / illegal move | **Do not mutate state.** Log clearly (`console` is fine today). Silent reject is OK — client board stays correct via sync. |
| Optional user-visible feedback | Prefer patterns documented in `server-work-with-errors`. Client today surfaces **`room.onError`** into `game.error`, and does **not** handle a custom move-error message. Do not invent `client.send('error', …)` / new message types unless the **client** store is updated in the same change set. |
| Auth / join failures | Not message-handlers — `onAuth` throw / join reject (`server-work-with-auth`). |

Do not kick the player solely for one illegal move. Do not put move failures on HTTP.

## New Message Types

Before adding e.g. `resign`, `rematch`, `chat`:

1. Confirm the sibling client will `room.send` / `room.onMessage` the same name and payload.
2. Prefer matching client naming; avoid server-only aliases.
3. Still: validate → mutate schema (or broadcast) — never trust client state.
4. Keep lobby listing on HTTP `GET /rooms/:roomName`.

## Do / Don't

| Do | Don't |
|----|--------|
| Implement `this.onMessage('move', …)` with `{ from, to }` `{ row, col }` | Invent alternate payloads (`fromIndex`, chess SAN, etc.) without client update |
| Validate then mutate schema | Trust client board or apply moves blindly |
| Keep success path = state sync only | Require a success ack message the client does not handle |
| Treat lobby as LobbyRoom live list (not a `move`-style message) | Add a gameplay room message for room listing |
| Align with `../happy-tourist.github.io/src/stores/game.ts` | Change client unilaterally to match a server-only protocol |
| Follow `server-work-with-errors` for reject / feedback | Invent ServiceError-style envelopes for moves |

## Change Checklist

1. Handler registered on the checkers room (`MyRoom` / renamed room) via `this.onMessage`.
2. Payload matches client `sendMove` (`from` / `to` with `row` / `col`).
3. Illegal moves do not mutate schema; legal moves update board / turn / status.
4. Rules delegated to checkers helpers; message handler stays I/O + orchestration.
5. Error feedback consistent with `server-work-with-errors` and current client listeners (`onError` / state only unless client also gains a listener).
6. No move logic in Express routes; lobby remains HTTP listing.
7. Update `test/` / `loadtest/` when the message contract becomes testable.
8. Run `npm test` / `npm run build` from the server package root as appropriate; fix failures before claiming done.

## Related

- Room lifecycle / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Schema sync fields: `.agents/skills/server/work-with-schema/SKILL.md`
- Rules engine: `.agents/skills/server/work-with-checkers/SKILL.md`
- Reject / log / feedback: `.agents/skills/server/server-work-with-errors/SKILL.md`
- Client protocol: `../happy-tourist.github.io/src/stores/game.ts` and client `.agents/skills/client/work-with-rooms/SKILL.md`
- Package overview: `AGENTS.md` (Current vs client contract)
