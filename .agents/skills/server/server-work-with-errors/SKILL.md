---
name: server-work-with-errors
description: >-
  Use when adding, changing, reviewing, or debugging error handling in the
  happy-tourist Colyseus tourist server — JWT onAuth failures, invalid room
  messages (`move` reject), HTTP /health and createEndpoint responses, console
  lifecycle logging, or aligning actionable game failures with the client store
  error + q-banner UX.
---

# Work With Errors

Use this skill when working with auth, room, and HTTP failures in this package
(`happy-tourist-server`).

Stack: Colyseus 0.18 (`defineServer` / `defineRoom`), `@colyseus/auth` + JWT,
Express 5 inside `defineServer({ express })`, TypeScript (`"type": "module"`,
NodeNext), Node `>= 22`. This is a **realtime game server**, not a REST BFF.

**Paths:** this skill currently lives in **this server repo** at
`.agents/skills/server/` (temporary). Canonical skills/OpenSpec will move to
**happy-tourist-meta** when present (`project-map.md` key `happy-tourist-meta`).
Runtime `src/…` paths are relative to **this repository root**. Sibling client is
**`../happy-tourist.github.io`**.

Related skills (by name only — load when that area is in scope):
`server-work-with-auth`, `work-with-rooms`, `work-with-messages`,
`work-with-game`, `work-with-routes`, `work-with-middleware`,
`server-work-with-structure`, `server-work-with-test`. Sibling client:
`client-work-with-errors`.

## Core Model

There is **no** `ServiceError`, **no** `{ errorCode, errorMessage }` envelope,
**no** global Express error middleware catalog, and **no** pino/Prometheus stack
unless you add them intentionally for a product decision.

Failures split by channel:

| Channel | What fails | How the client sees it today |
|---------|------------|------------------------------|
| Room `onAuth` | Bad/missing JWT | Join/create rejects; Pinia `game.error` / `auth` path via thrown Error |
| Room gameplay | Illegal / out-of-turn `move` | Not implemented yet; when added, send an actionable string the client can put in `game.error` + `q-banner` |
| Room protocol | Disconnect / room exception | Client `room.onError` → `game.error` |
| HTTP | `/health`, `/api/hello`, lobby `GET /rooms/:name` | Promise reject → store `error` string; keep bodies simple |

Prefer **clear, channel-appropriate rejection** over inventing a BFF-style code map.

```
Auth gate:  JWT.verify throws → Colyseus denies join
Moves:      validate authoritatively → client.send('error', { message }) (preferred)
HTTP:       simple JSON / status; no ServiceError envelope
Logs:       console.log / console.warn / console.error with a room/tag prefix
```

## Auth Failures (`onAuth`)

Scaffold (`src/rooms/MyRoom.ts`):

```ts
static async onAuth(token: string, _options: any, _context: any) {
  const userdata = await JWT.verify(token);
  return userdata;
}
```

- Let `JWT.verify` **throw** on invalid/expired/missing token. Do not catch-and-return
  `false` unless you have a deliberate Colyseus reason; throwing matches the scaffold
  comment and blocks the client from joining.
- Return verified userdata to `onJoin` on success.
- Do not invent custom `{ errorCode }` payloads here — the client already maps join
  failures to `e instanceof Error ? e.message : String(e)`.
- Auth HTTP (`/auth/*` from `@colyseus/auth`) is framework-owned; do not wrap it in a
  local ServiceError layer.

## Room Gameplay Failures

Authoritative rules live on the server. **Never trust** client board state or
“this move is legal” claims.

### Preferred approach (align with scaffold + intended tourist)

When implementing `move` `{ from, to }`:

1. **Validate** in the message handler (turn, color ownership, russian-tourist rules).
2. On failure: **do not mutate** synced state.
3. **Notify the offending client** with a short human-readable message:

```ts
this.onMessage("move", (client, message) => {
  const result = tryApplyMove(/* … */);
  if (!result.ok) {
    console.warn(`[tourist] move rejected ${client.sessionId}: ${result.reason}`);
    client.send("error", { message: result.reason });
    return;
  }
  // mutate schema state only on success
});
```

Why `client.send("error", { message })` (not silent ignore, not `throw` in the handler) **when product wants a banner**:

- Client UX is store `error: string | null` + `q-banner` (`client-work-with-errors`).
- Optional user-visible game failures need a **server → client** channel the game store can map to `game.error`.
- Today `onMessage('move')` may **silently reject** (no mutate) — that is OK and keeps clients consistent via schema sync; add `error` messages only when product asks for banners on illegal moves.
- Throwing inside an `onMessage` handler is a poor fit for expected rule violations
  (noise, possible disconnect semantics). Reserve hard failures for auth / fatal room
  conditions.

Coordinate the message name/payload with the client when wiring listeners
(`work-with-messages` / sibling `client-work-with-errors`). Prefer one stable type
such as `"error"` with `{ message: string }` unless the client already listens for
another name — then match the client.

### When silence or hard reject is OK

| Situation | Pattern |
|-----------|---------|
| Malformed payload / wrong shape / wrong types | Log + return (optional short `error` send) |
| Not this player's turn / not their color | `client.send("error", { message })` |
| Illegal tourist move (rules) | Same — actionable `message` |
| Duplicate / spam after disconnect | Log + ignore |
| Auth at join | Throw from `onAuth` (scaffold) |
| Fatal room inconsistency | Log `console.error`; end match / dispose rather than sync lying state |

Optional: also update synced `status` / winner fields when the game ends; that is
state sync, not a substitute for per-move rejection feedback.

## HTTP Surface

Keep responses simple. Do **not** invent `{ errorCode, errorMessage }` unless product
and the client already consume it (they do not).

| Endpoint | Pattern |
|----------|---------|
| `GET /health` | `{ status, uptime }` — stay healthy JSON; no error envelope |
| `GET /hi` | Plain text smoke |
| `createEndpoint` `/api/hello` | Return plain JSON objects; on failure let the framework/status reflect it — no BFF catalog |
| `GET /rooms/:roomName` | Colyseus listing; client treats HTTP errors as Error messages |

CORS stays first in the Express hook (`work-with-middleware` / `work-with-config`).
Do not add a global “map every err to errorCode” BFF-style middleware.

## Logging

Today rooms use prefixed `console.log` in lifecycle (`onCreate` / `onJoin` /
`onLeave` / `onDispose`). Prefer that style:

- Prefix with room name/tag: `[MyRoom]`, later `[tourist]`.
- `console.warn` for rejected moves / soft failures.
- `console.error` for unexpected exceptions / corrupt state.
- Include `sessionId` when the failure is client-specific.

Do **not** introduce pino, structured metrics, or Prometheus “because BFF had them”
unless there is an explicit decision to add observability.

## Checklist For New Failures

1. Identify the channel: `onAuth` / room message / HTTP / fatal room.
2. Auth: let `JWT.verify` throw; return userdata on success.
3. Moves: validate server-side; on reject, no state mutation + `client.send("error", { message })` with actionable text.
4. HTTP: simple JSON/status; no ServiceError / code catalog.
5. Log with a clear prefix; keep `console.*` unless observability is an intentional add.
6. Align message strings / event names with sibling client (`game.error` + `q-banner`).
7. Update tests when join-auth or move-reject contracts change (`server-work-with-test`).

## Common Mistakes

- Porting `ServiceError` + `{ errorCode, errorMessage }` BFF envelopes into this server.
- Catching `JWT.verify` and allowing join anyway.
- Trusting client board / action legality and only “syncing what they sent”.
- Throwing on every illegal action and disconnecting the player.
- Building a global Express error middleware for `/health` and demo routes.
- Adding pino/Prometheus solely to mirror another project.
- Putting Russian/English copy only in server logs while sending empty or coded
  payloads the client cannot show in `q-banner` (when an error message type exists).
- Changing room name / `move` protocol without coordinating the client (`tourist`; lockstep).

## Key Files

| Path | Role |
|------|------|
| `src/rooms/MyRoom.ts` | `onAuth` JWT gate; lifecycle logs; `onMessage('move')` (silent reject OK) |
| `src/game/touristMove.ts` | Pure validate — reject reasons stay server-side unless product wires banners |
| `src/rooms/schema/MyRoomState.ts` | Synced state — mutate only after valid actions |
| `src/app.config.ts` | Rooms, `createEndpoint`, `/health`, CORS |
| `src/db/schema.ts` | User defaults (auth register/login MUST NOT fail on NOT NULL) |
| `test/MyRoom.test.ts` | JWT connect happy path; extend for reject cases |
| Sibling `../happy-tourist.github.io` stores/pages | `game.error` / `auth.error` + `q-banner` consumers |
