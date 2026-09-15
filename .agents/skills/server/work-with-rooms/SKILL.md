---
name: work-with-rooms
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus Room handlers in
  the happy-tourist tourist server: MyRoom lifecycle (onAuth / onCreate /
  onJoin / onDrop / onReconnect / onLeave / onDispose), tourist reconnect grace
  vs LobbyRoom fire-and-forget, room registration in app.config (lobby +
  tourist + enableRealtimeListing), maxSeats / seat connectivity / start phase,
  JWT room gate, or aligning room name with client TOURIST_ROOM.
---

# Work With Rooms

Use this skill for **authoritative Colyseus Room handlers** in `happy-tourist-server`.

Room lifecycle and join gate live in **`src/rooms/MyRoom.ts`**. Registration lives in **`src/app.config.ts`**. Do **not** put board rules or move validation in Express routes.

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Client counterpart (Pinia connect / leave / listeners):  
`happy-tourist-meta/.agents/skills/client/work-with-rooms/SKILL.md` (and lobby list: `work-with-lobby`).

**There is no separate `server/work-with-lobby` skill** — lobby policy for this package is documented here under Registration / LobbyRoom.

Coordinate with sibling skills when they exist: `work-with-schema`, `work-with-messages`, `work-with-game`.

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Registration | `src/app.config.ts` | `lobby: defineRoom(LobbyRoom)`; `tourist: defineRoom(MyRoom).enableRealtimeListing()` |
| Game handler | `src/rooms/MyRoom.ts` | `Room` subclass: `onAuth`, `onCreate`, `onJoin`, `onDrop`, `onReconnect`, `onLeave`, `onDispose`; `onMessage('move'|'ready'|'say')` |
| Schema | `src/rooms/schema/MyRoomState.ts` | Synced: `phase` / `maxSeats` / `countdownRemaining` + legacy `started` + `seats` Map (`connected` / `reconnectUntil` / `ready` / `finishPlace` + piece `finished`) + `currentTurnSessionId` + `nextFinishPlace` |
| Tests | `test/MyRoom.test.ts` | JWT, tourist connect; capacity/start (SC-START-*); seating (SC-PIECE-*); turn/move (SC-MOVE-*); finish (SC-FINISH-*); say (SC-SAY-*); lobby listing (SC-LOBBY-02/03) |
| Loadtest | `loadtest/example.ts` | `joinOrCreate`; `--room tourist` |

Registered room keys today: **`lobby`** (built-in listing) and **`tourist`** (playable `MyRoom` with realtime listing). Client `TOURIST_ROOM` / `LOBBY_ROOM` match these names — do not reintroduce `my_room`.

Constants: `RECONNECT_GRACE_SECONDS = 30`, `COUNTDOWN_SECONDS = 5` exported from `MyRoom.ts`.

## Lifecycle Flow

```text
client create({ maxSeats }) / joinById / joinOrCreate / reconnect('tourist')
        │
        ▼
  static onAuth(token)     ← JWT.verify; fail → reject join
        │  returns userdata → onJoin(…, auth)
        ▼
  onCreate(options)        ← once per room instance
        │  setState; parse maxSeats (2|3|4, default 2); phase=waiting
        │  refreshMetadata({ title, status, maxSeats, seats })
        │  onMessage('move'|'ready'|'say')
        │  (enableRealtimeListing publishes to LobbyRoom subscribers)
        ▼
  onJoin(client, options, auth)
        │  seat while seats.size < maxSeats (any phase); else spectator
        │  touristId + 4 pieces; connected=true; reconnectUntil=0; ready=false
        │  maybeStartCountdown (full table → countdown)
        ▼
  onDrop(client)           ← unexpected disconnect (seated only)
        │  connected=false; reconnectUntil=now+30s; allowReconnection(client, 30)
        │  spectators: no hold; does not cancel countdown
        ▼
  onReconnect(client)      ← within grace
        │  connected=true; reconnectUntil=0; same seat/pieces
        ▼
  onLeave(client)          ← consented leave, grace timeout, or reconnect denied
        │  delete seat; refreshMetadata; maybeStartCountdown (remaining all-ready)
        │  if seats.size===0 → disconnect() (spectators do not hold room)
        ▼
  onDispose()              ← room empty / locked shut → lobby `-` update
```

### Hook responsibilities

| Hook | Do here | Don't |
|------|---------|--------|
| `static onAuth` | `JWT.verify(token)`; return userdata | Trust client-supplied identity without JWT |
| `onCreate` | `setState`; parse `maxSeats`; `phase=waiting`; `refreshMetadata`; register `move`/`ready`/`say`; do **not** set `maxClients = maxSeats` | Mutate board from HTTP |
| `onJoin` | Assign seat while under `maxSeats` in any phase; unique `touristId` + 4 pieces; online connectivity; `ready=false`; maybe start countdown when full | Cap the room with `maxClients`; gate seating on legacy `started` alone |
| `onDrop` | Seated: mark offline + `allowReconnection(client, 30)`; hold seat/pieces; do not cancel countdown | Treat unexpected drop as immediate seat delete; grace for spectators or LobbyRoom |
| `onReconnect` | Restore seat online (`connected=true`, `reconnectUntil=0`) | Re-assign a new seat / touristId |
| `onLeave` | Permanent remove seat; seats reopen while under `maxSeats`; refreshMetadata; maybeStartCountdown; if zero seats → `disconnect()` | Leave stale seats; let spectators keep an empty-seated room alive; clear others' `ready` on leave |
| `onDispose` | Cleanup timers / logs | Assume clients still connected |

## Current Room (`MyRoom.ts`)

```ts
export const RECONNECT_GRACE_SECONDS = 30;
export const COUNTDOWN_SECONDS = 5;

export class MyRoom extends Room<{ state: MyRoomState }> {
  static async onAuth(token: string, _options: any, _context: any) {
    const userdata = await JWT.verify(token);
    return userdata;
  }

  onCreate(options: any) {
    this.setState(new MyRoomState());
    this.state.maxSeats = parseMaxSeats(options); // 2|3|4, else 2
    this.state.phase = "waiting";
    this.refreshMetadata(); // title / status / maxSeats / seats
    this.onMessage("move", …);
    this.onMessage("ready", …);
    this.onMessage("say", …);
  }
  onJoin(client: Client, _options: any, auth: any) { /* seat if under maxSeats; maybeStartCountdown */ }
  onDrop(client: Client, _code?: number) { /* seated: offline + allowReconnection(30) */ }
  onReconnect(client: Client) { /* seat online again */ }
  onLeave(client: Client, _code?: number) { /* delete seat; empty → disconnect() */ }
  onDispose() { /* room closed */ }
}
```

- `onAuth` is **static**; invalid JWT throws → client cannot connect.
- `auth` in `onJoin` is the userdata returned from `onAuth`.
- `refreshMetadata` feeds LobbyPage: `{ title, status, maxSeats, seats }` — `seats` = occupied seated count; `status` is `playing` only when `phase === 'playing'` (else `waiting`, including countdown).
- Consented client `leave()` goes straight to `onLeave` (no grace). Unexpected drop uses Colyseus `onDrop` → `allowReconnection`.
- Seating / start / reconnect details: `work-with-game` / `work-with-schema`.

## Registration And Live Lobby

```ts
// src/app.config.ts
import { defineRoom, LobbyRoom, /* … */ } from "colyseus";

rooms: {
  lobby: defineRoom(LobbyRoom),
  tourist: defineRoom(MyRoom).enableRealtimeListing(),
},
```

| Client action | Needs server |
|---------------|--------------|
| `joinOrCreate('lobby', { filter: { name: 'tourist' } })` | Key `lobby` → built-in `LobbyRoom` |
| Live `rooms` / `+` / `-` updates | `.enableRealtimeListing()` on `tourist` |
| `client.create('tourist', { maxSeats })` / `joinById` / `reconnect` | Key `tourist` in `rooms` + MyRoom grace hooks |
| `client.http.get('/rooms/tourist')` | Same (HTTP listing still available; UI uses LobbyRoom) |

Without `.enableRealtimeListing()`, lobby subscribers will not get create/dispose updates for `tourist`.

### LobbyRoom policy (D7 — no server lobby skill)

- Lobby is **listing only**. Do **not** call `allowReconnection` for LobbyRoom; do **not** add seat/connectivity grace for lobby subscribers.
- Dropped lobby clients are not held; client quietly resubscribes (see client `work-with-lobby`).
- Tests / loadtest use room name **`tourist`** (not `my_room`). Lobby listing tests cover SC-LOBBY-02 / SC-LOBBY-03.

## Intended Match Rules (Product)

Align with client Pinia expectations:

| Concern | Target |
|---------|--------|
| Capacity | No `maxClients = maxSeats`; seated ≤ `maxSeats` (2\|3\|4) via seats check; spectators may join |
| Seats | Unique `touristId` 1…4 + exactly four pieces (one per side N/E/S/W on free start cells; see `work-with-game`); seat while free slot in **any** phase |
| Connectivity | `connected` + `reconnectUntil` on Seat (D2); online at assign |
| Start | `phase` waiting → countdown → playing; full table or all-ready underfilled; legacy `started` mirrors `phase === 'playing'` |
| Status metadata | `waiting` until `phase === 'playing'`; always publish `maxSeats` + occupied `seats` |
| Unexpected drop | Hold seat 30 s + `allowReconnection`; sync offline + deadline (SC-PIECE-11…14, 16); does not cancel countdown |
| Consented leave | Immediate seat remove; free seat reopens while under maxSeats (SC-PIECE-07/08) |
| Empty seated | After permanent remove, `seats.size === 0` → `disconnect()` even with spectators (SC-PIECE-15 / D4) |
| Authority | Server mutates schema state; client only mirrors seats / presence / phase / renders |

Do not trust client-local board UI. Move validation belongs in the room (see `work-with-messages` / `work-with-game`).

## Client Protocol Expectations

From client rooms / lobby skills — server must provide **today**:

| Expectation | Notes |
|-------------|--------|
| Room type `tourist` | Registered key today; reconnect grace **only** here |
| Room type `lobby` | Built-in `LobbyRoom` for live list; **no** reconnect hold |
| Tourist board layout | Client-only tile geometry; server does **not** sync layout |
| Synced seats / phase / connectivity | `phase` / `maxSeats` / `countdownRemaining` + `seats` Map with `connected` / `reconnectUntil` / `ready` (+ legacy `started`) |
| Listing metadata | `title` / `status` / `maxSeats` / `seats` via `refreshMetadata` for LobbyPage rows |

Client: tourist token in `localStorage` + `reconnect` then `joinById`; lobby quiet resubscribe without token. Keep tourist rooms joinable by reconnection token within grace.

Do not treat legacy draughts `board` / `currentTurn` / cells `0`–`4` / `move` `{ from, to }` as current product requirements.

## Auth Gate

```ts
static async onAuth(token: string, _options: any, _context: any) {
  return await JWT.verify(token);
}
```

- Client sets `colyseus.sdk.auth.token` (JWT from `@colyseus/auth` login/anonymous).
- Tests: `JWT.sign({…})` then `colyseus.sdk.auth.token = token` before `createRoom` / `connectTo`.
- Secrets: `JWT_SECRET` (and related) in `.env.*` — not room-handler concerns beyond verify.
- LobbyRoom has no custom JWT `onAuth`; lobby screen is already behind client `requiresAuth`.

## Do / Don't

| Do | Don't |
|----|--------|
| Keep lifecycle in `MyRoom.ts` | Put match logic in `express(app)` routes |
| Register `lobby` + `tourist` with `.enableRealtimeListing()` | Reintroduce `my_room` or omit realtime listing |
| `allowReconnection` only on tourist seated `onDrop` | Add LobbyRoom grace / hold listing subscribers |
| Assign ≤`maxSeats` via seats check; allow spectator joins | Cap the room with `maxClients = maxSeats`; gate seating only on legacy `started` |
| Prefer `phase` for start/move gates | Treat fourth-seat auto-`started` as current product |
| Dispose when last seated leaves (even with spectators) | Let spectators keep an empty-seated match alive |
| Return userdata from `onAuth` for `onJoin` | Skip JWT verify for "dev convenience" in committed code |
| Coordinate schema / messages / rules with sibling skills + client | Change state field names unilaterally |
| Update `test/MyRoom.test.ts` + loadtest when room name or auth changes | Assume tests still pass after rename |

## Change Checklist

1. Belongs in `MyRoom` lifecycle or `app.config` `rooms` map — not a new HTTP BFF.
2. Registration: `lobby` + `tourist` (+ `.enableRealtimeListing()` on tourist); no LobbyRoom `allowReconnection`.
3. `onAuth` still `JWT.verify`; userdata reaches seat assignment.
4. Seating + start phases/ready/countdown + consented vs unexpected leave/reconnect + empty-seated dispose are explicit; no `maxClients = maxSeats`.
5. Synced fields / messages match client skills (schema/messages/game) including connectivity, `phase`, `maxSeats`, `ready`.
6. Tests + loadtest use `tourist`; SC-PIECE + SC-START + SC-MOVE + lobby live-list covered where applicable.
7. Run `npm test` / `npm run build` from the server package root; fix failures before claiming done.

## Related

- Schema state: `.agents/skills/server/work-with-schema/SKILL.md` (when present)
- Messages: `.agents/skills/server/work-with-messages/SKILL.md`
- Tourist rules: `.agents/skills/server/work-with-game/SKILL.md` (when present)
- Locate files only: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Server overview: `AGENTS.md` (Current vs client contract)
- Client lobby: `happy-tourist-meta/.agents/skills/client/work-with-lobby/SKILL.md`
- Client rooms (Pinia): `happy-tourist-meta/.agents/skills/client/work-with-rooms/SKILL.md`
