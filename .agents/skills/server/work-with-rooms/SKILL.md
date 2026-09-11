---
name: work-with-rooms
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus Room handlers in
  the happy-tourist checkers server: MyRoom lifecycle (onAuth / onCreate /
  onJoin / onLeave / onDispose), room registration in app.config, maxClients /
  seat colors / disconnect handling, JWT room gate, or aligning room name with
  client CHECKERS_ROOM.
---

# Work With Rooms

Use this skill for **authoritative Colyseus Room handlers** in `happy-tourist-server`.

Room lifecycle and join gate live in **`src/rooms/MyRoom.ts`**. Registration lives in **`src/app.config.ts`**. Do **not** put board rules or move validation in Express routes.

Skills path for now: `.agents/skills/server/` in this repo (canonical copy may later live under `happy-tourist-meta`). Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Client counterpart (Pinia connect / leave / listeners):  
`../happy-tourist.github.io/.agents/skills/client/work-with-rooms/SKILL.md`.

Coordinate with sibling skills when they exist: `work-with-schema`, `work-with-messages`, `work-with-checkers`.

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Registration | `src/app.config.ts` | `rooms: { my_room: defineRoom(MyRoom) }` — lobby lists by this key |
| Handler | `src/rooms/MyRoom.ts` | `Room` subclass: `onAuth`, `onCreate`, `onJoin`, `onLeave`, `onDispose` |
| Schema | `src/rooms/schema/MyRoomState.ts` | Synced state (scaffold today; see `work-with-schema`) |
| Tests | `test/MyRoom.test.ts` | Boot `appConfig`, JWT, create/connect room |
| Loadtest | `loadtest/example.ts` | `joinOrCreate`; pass `--room` when name changes |

Today the room is registered as **`my_room`**. Client expects **`checkers`** (`CHECKERS_ROOM`). Prefer renaming the registration key (and tests/loadtest) when implementing checkers — do not invent a second room type.

## Lifecycle Flow

```text
client create / joinById / joinOrCreate('checkers')
        │
        ▼
  static onAuth(token)     ← JWT.verify; fail → reject join
        │  returns userdata → onJoin(…, auth)
        ▼
  onCreate(options)        ← once per room instance
        │  setState, maxClients = 2, onMessage handlers
        ▼
  onJoin(client, options, auth)
        │  seat color, players map, maybe start match
        ▼
  onLeave(client, code?)
        │  pause / forfeit / allowReconnect window
        ▼
  onDispose()              ← room empty / locked shut
```

### Hook responsibilities

| Hook | Do here | Don't |
|------|---------|--------|
| `static onAuth` | `JWT.verify(token)`; return userdata | Trust client-supplied identity without JWT |
| `onCreate` | `this.setState(...)`, `this.maxClients = 2`, register `onMessage` | Mutate board from HTTP |
| `onJoin` | Assign seat (`white` / `black`), write `players[sessionId]`, transition `status` when 2 seated | Let a third client in if `maxClients` should be 2 |
| `onLeave` | Handle disconnect (reconnect window, forfeit, reset to `waiting`) | Leave stale `players` entries forever without a policy |
| `onDispose` | Cleanup timers / logs | Assume clients still connected |

## Current Scaffold (`MyRoom.ts`)

```ts
export class MyRoom extends Room {
  static async onAuth(token: string, _options: any, _context: any) {
    const userdata = await JWT.verify(token);
    return userdata;
  }

  onCreate(_options: any) { /* init state, maxClients = 2 */ }
  onJoin(client: Client, _options: any, auth: any) { /* seat white/black */ }
  onLeave(client: Client, _code?: number) { /* disconnect policy */ }
  onDispose() { /* room closed */ }
}
```

- `onAuth` is **static**; invalid JWT throws → client cannot connect.
- `auth` in `onJoin` is the userdata returned from `onAuth`.
- Comments already describe intended checkers flow; implement against the **client contract**, not a parallel protocol.

## Registration And Lobby

```ts
// src/app.config.ts
rooms: {
  my_room: defineRoom(MyRoom), // today
  // prefer: checkers: defineRoom(MyRoom) when implementing
},
```

| Client action | Needs server room name |
|---------------|------------------------|
| `client.create('checkers')` | Key `checkers` in `rooms` |
| `client.joinOrCreate('checkers')` | Same |
| `client.http.get('/rooms/checkers')` | Same (Colyseus listing) |
| `joinById(roomId)` | Any registered room that still exists |

When renaming `my_room` → `checkers`, also update `test/MyRoom.test.ts` and loadtest `--room`.

## Intended Match Rules (Product)

Align with client Pinia expectations when filling stubs:

| Concern | Target |
|---------|--------|
| Capacity | `this.maxClients = 2` in `onCreate` |
| Seats | First joiner → `white`, second → `black` (or explicit seat options if added later) |
| Status | `waiting` until 2 players; then `playing`; end → `finished` |
| Disconnect | Decide explicitly: allow reconnect for a window **or** forfeit / return to `waiting` — document in code; client `onLeave` resets Pinia |
| Authority | Server mutates schema state; client only sends intents |

Do not trust client board / turn / color. Client `sendMove` is an intent; validation belongs in the room (see `work-with-messages` / `work-with-checkers`).

## Client Protocol Expectations

From client `work-with-rooms` — server must eventually provide:

| Expectation | Notes |
|-------------|--------|
| Room type `checkers` | Rename registration when implementing |
| State `board` | `0`–`4` cells (empty / white / black / kings) — schema skill |
| State `currentTurn` | `'white' \| 'black'` |
| State `status` | `'waiting' \| 'playing' \| 'finished'` |
| State `players[sessionId].color` | Client maps to `myColor` via `room.sessionId` |
| Message `move` `{ from, to }` | Handler via `this.onMessage('move', …)` — messages skill |

Client always leaves before enter; may rejoin by `roomId` after refresh (`joinById`). Keep rooms joinable by id while the match should continue (avoid disposing too eagerly on brief disconnect if reconnect is intended).

## Auth Gate

```ts
static async onAuth(token: string, _options: any, _context: any) {
  return await JWT.verify(token);
}
```

- Client sets `colyseus.sdk.auth.token` (JWT from `@colyseus/auth` login/anonymous).
- Tests: `JWT.sign({…})` then `colyseus.sdk.auth.token = token` before `createRoom` / `connectTo`.
- Secrets: `JWT_SECRET` (and related) in `.env.*` — not room-handler concerns beyond verify.

## Do / Don't

| Do | Don't |
|----|--------|
| Keep lifecycle in `MyRoom.ts` | Put match logic in `express(app)` routes |
| Prefer room name `checkers` when implementing | Leave `my_room` forever while client calls `checkers` |
| Set `maxClients = 2` and seat colors in join | Allow unbounded clients into a 1v1 match |
| Return userdata from `onAuth` for `onJoin` | Skip JWT verify for "dev convenience" in committed code |
| Coordinate schema / `move` / rules with sibling skills + client | Change state field names unilaterally |
| Update `test/MyRoom.test.ts` when room name or auth changes | Assume tests still pass after rename |

## Change Checklist

1. Belongs in `MyRoom` lifecycle or `app.config` `rooms` map — not a new HTTP BFF.
2. If product checkers: registration key is `checkers` (or documented transitional dual-register — prefer single name).
3. `onAuth` still `JWT.verify`; userdata reaches seat assignment.
4. `maxClients`, seats, disconnect policy are explicit.
5. Synced fields / `move` payload match client `work-with-rooms` (and schema/messages skills).
6. Tests + loadtest use the registered room name.
7. Propose `npm test` / `npm run build`; wait for user «готово».

## Related

- Schema state: `.agents/skills/server/work-with-schema/SKILL.md` (when present)
- Messages (`move`, …): `.agents/skills/server/work-with-messages/SKILL.md` (when present)
- Checkers rules: `.agents/skills/server/work-with-checkers/SKILL.md` (when present)
- Locate files only: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Server overview: `AGENTS.md` (Current vs client contract)
- Client rooms (Pinia): `../happy-tourist.github.io/.agents/skills/client/work-with-rooms/SKILL.md`
