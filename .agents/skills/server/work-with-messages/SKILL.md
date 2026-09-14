---
name: work-with-messages
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room messages in
  the happy-tourist tourist server — onMessage handlers, validating client
  intents before mutating schema state, optional error feedback, or aligning
  message contracts with the client. Gameplay messages are deferred until rules
  land. Not for lobby listing (LobbyRoom / HTTP fallback).
---

# Work With Messages

Use this skill for **room WebSocket messages** in `happy-tourist-server` (authoritative Colyseus room handlers).

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Coordinate with: `work-with-rooms` (lifecycle / registration), `work-with-schema` (synced state), `work-with-game` (rules), `server-work-with-errors` (reject / feedback style).

## Scope Boundary

| In scope | Out of scope |
|----------|--------------|
| `this.onMessage(...)` for future game actions | Lobby listing — client uses **LobbyRoom** (`rooms` / `+` / `-`) |
| Payload shape / type guards for client intents | Auth join gate — `onAuth` + JWT (`server-work-with-auth`) |
| Validate → mutate `@colyseus/schema` state | Express routes / `createEndpoint` |
| Optional per-client error feedback for bad actions | Pure board-game rules (`work-with-game`) |

**Prefer changing the server to match the client** rather than inventing a parallel protocol. Today the client has **no** Game move messages (seating syncs via schema); do not treat legacy draughts `move` `{ from, to }` as current product canon.

## Client Contract (today)

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | *(none for Game moves)* | Seating via join lifecycle; add messages when move rules land |
| Server → clients | Schema sync | `started` + `seats` Map → client `onStateChange` |
| Server → client | room error channel | Client sets `game.error` from `room.onError` |
| Lobby | HTTP fallback | `client.http.get('/rooms/tourist')` — **not** a room message |

When implementing move rules, document the chosen message shape here and lockstep with client `stores/game.ts`.

## Server Today

- `src/rooms/MyRoom.ts` — seating in `onJoin`/`onLeave`; **no** gameplay `onMessage` yet.
- Room registered as `tourist` (+ `lobby` for live list); do not reintroduce `my_room`.

## Handler Pattern (when rules land)

Register handlers in the room (typically `onCreate`), not in Express:

```ts
this.onMessage('<action>', (client, message) => {
  // 1. Shape-check message
  // 2. Resolve seat for client.sessionId
  // 3. Validate with work-with-game rules
  // 4. On success: mutate this.state
  // 5. On failure: do not mutate; follow error guidance
});
```

### Hard rules

1. **Do not trust the client.** Treat payloads as intent only.
2. **Validate, then mutate** schema state (`work-with-schema`).
3. **Authoritative rules** live in room / game helpers (`work-with-game`), not in the message payload.
4. **Do not** add HTTP endpoints for gameplay actions.

## Error Feedback (align with `server-work-with-errors`)

| Situation | Preferred approach |
|-----------|-------------------|
| Malformed / illegal action | **Do not mutate state.** Silent reject is OK — clients stay consistent via sync. |
| Optional user-visible feedback | Prefer `room.onError` patterns the client already maps to `game.error`. Do not invent new message types unless the client is updated in the same change. |
| Auth / join failures | Not message-handlers — `onAuth` / join reject. |

## New Message Types

1. Confirm the sibling client will `room.send` / `room.onMessage` the same name and payload.
2. Validate → mutate schema — never trust client state.
3. Keep lobby listing on LobbyRoom / HTTP `GET /rooms/:roomName`.

## Do / Don't

| Do | Don't |
|----|--------|
| Add messages only with client lockstep | Reintroduce draughts `move`/`{from,to}` as product without a rules change |
| Validate then mutate schema | Trust client board UI |
| Treat lobby as LobbyRoom live list | Add a gameplay room message for room listing |
| Follow `server-work-with-errors` | Invent BFF-style error envelopes for room actions |

## Change Checklist

1. Handler registered on the tourist room via `this.onMessage`.
2. Payload matches the client store.
3. Illegal actions do not mutate schema.
4. Rules delegated to `work-with-game`; handler stays I/O + orchestration.
5. Update `test/` / `loadtest/` when the message contract becomes testable.
6. Run `npm test` from the server package root; fix failures before claiming done.

## Related

- Room lifecycle / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Schema sync fields: `.agents/skills/server/work-with-schema/SKILL.md`
- Rules engine: `.agents/skills/server/work-with-game/SKILL.md`
- Reject / log / feedback: `.agents/skills/server/server-work-with-errors/SKILL.md`
- Client board: `.agents/skills/client/work-with-game-board/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
