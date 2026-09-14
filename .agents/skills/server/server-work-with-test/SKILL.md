---
name: server-work-with-test
description: >-
  Use when planning or writing mocha + @colyseus/testing tests for
  happy-tourist-server: room connect with JWT, onAuth failures, seating /
  reconnect grace (SC-PIECE), move messages, schema sync assertions,
  GET /rooms listing, or preference HTTP (GET/POST /api/theme).
  Core workflow: test plan (mocks/verify) → write test/*.test.ts → run npm test
  from server package root and fix failures. Do not invent Jest/babel patterns.
trigger: slash
---

# Work With Test

Use this skill to plan and write integration-style tests for the Colyseus
server (`happy-tourist-server`). Start with a test plan, then implement tests
from that plan.

**Paths:** this skill currently lives in **this server repo** at
`.agents/skills/server/` (temporary; later move to **happy-tourist-meta**).
Runtime `src/…` and `test/…` paths are relative to **this repository root**.
Sibling client: `../happy-tourist.github.io`.

Stack: **mocha**, **tsx** (`-r tsx`), **`@colyseus/testing`**
(`ColyseusTestServer`, `boot`), **Node `assert`**, **TypeScript** ESM
(`"type": "module"`). Tests are `test/**.test.ts`.

Core principle: exercise the public contract of rooms / HTTP / auth as the
client will see them (JWT join, synced schema fields, `move` accept/reject,
room listing), not private helpers. Prefer extending `test/MyRoom.test.ts`
patterns over inventing Jest, Vitest, or babel setups.

## Mocha / Colyseus Testing Setup

| Piece | Role |
|-------|------|
| Script | `npm test` → `mocha -r tsx test/**.test.ts --exit --timeout 15000` |
| Runner | mocha + `@types/mocha`; load TS via tsx |
| Harness | `@colyseus/testing`: `boot(appConfig)`, `cleanup()`, `shutdown()` |
| Auth | `JWT` from `@colyseus/auth` — `JWT.sign` then `colyseus.sdk.auth.token` |
| App under test | `import appConfig from "../src/app.config.js"` |

Do **not** add `jest.config`, `babel-jest`, `vitest`, or `.cjs` test files.

Agent **runs** `npm test` from the server package root after writing or changing
tests; fix failures before claiming done.

## File Layout

| Subject | Test path |
|---------|-----------|
| `src/rooms/MyRoom.ts` (or `TouristRoom` if renamed) | `test/MyRoom.test.ts` (match room module name) |
| Room schema / messages covered with room | Same room test file (or `test/<Room>.messages.test.ts` if large) |
| HTTP helpers (`/health`, `/rooms/:name`) | `test/http.test.ts` or next to the feature under test |
| Preference HTTP (`GET`/`POST /api/theme`) | `test/theme.test.ts` (auth reject + persist/login + GET after POST same JWT + cross-device older JWT) |

Mocha picks up `test/**.test.ts` via the npm script. Mirror room names under
`test/` as rooms grow; keep relative imports to `../src/...`.

```typescript
import assert from "assert";
import { ColyseusTestServer, boot } from "@colyseus/testing";
import { JWT } from "@colyseus/auth";
import appConfig from "../src/app.config.js";
```

## Workflow

1. Read the SUT (room, schema, `app.config` room registration, HTTP surface)
   and note registered room names (`lobby` + `tourist`).
2. Produce a test plan with two sections: **What needs to be mocked / stubbed**
   and **What to verify**.
3. Place or extend a file under `test/` as `*.test.ts`, following the harness
   below and the coverage topics.
4. Cover success and failure paths that exist in the SUT (valid JWT join,
   invalid/missing token, illegal `move`, schema fields after join/move,
   lobby live-list `+` / `-` when applicable, JWT-gated HTTP like `GET|POST /api/theme`).
5. Run `npm test` from the server package root (optionally a single file if mocha
   path filtering is used); fix failures before claiming done.

## Test Plan

### What Needs To Be Mocked / Stubbed

Colyseus tests usually boot the real `appConfig` — prefer that over mocking
the framework. Stub only when the SUT would hit something you must not run:

- External side effects beyond the in-process SQLite user store (if a test
  would call out over the network).
- Time-dependent rules only if the room uses clocks you need to control
  (document the stub; do not invent fake timers APIs unused in the repo).

Do **not** mock `JWT.verify` when testing `onAuth` — use real `JWT.sign` /
  bad tokens so the gate matches production. Do **not** mock `@colyseus/schema`
  when asserting synced state.

### What To Verify

Use these categories only when the SUT has relevant behavior:

- Room connect: `sessionId` matches; `onAuth` userdata reaches `onJoin`.
- Auth failure: missing / invalid token → connect rejects (client cannot join).
- Schema sync: after join, client-visible state matches room —
  `started`, `seats` Map (`touristId` + exactly four `pieces` keyed by side
  `N|E|S|W` → `{ side, row, col }` + `connected` / `reconnectUntil`), and
  `currentTurnSessionId`. Assert four pieces / free start cells / leave-pool
  reopen as in SC-PIECE-01…08 (`test/MyRoom.test.ts`). Also cover reconnect
  grace SC-PIECE-11…16 (unexpected drop holds seat; reconnect restores; grace
  timeout removes; empty-seated dispose; connectivity sync).
- Messages / turn: cover SC-MOVE-* in `test/MyRoom.test.ts` (first seated holds
  turn; join-order rotation; legal orthogonal/diagonal; occupied/non-playable
  reject; out-of-turn / spectator reject; permanent leave advances; offline grace
  keeps turn). Pure rules without room I/O: `test/touristMove.test.ts`. Do **not**
  assert draughts-era `board` / `players[sessionId].color`.
- Listing: live LobbyRoom — after `createRoom("tourist")`, lobby client
  receives `+`; after dispose, receives `-` (SC-LOBBY-02/03). HTTP
  `GET /rooms/tourist` remains optional fallback.
- Preference HTTP: `GET`/`POST /api/theme` — unauthenticated / anonymous rejected;
  registered JWT persists `theme`, returns it on next login, `GET` returns
  profile after POST with the same JWT without re-login, and an older session
  JWT also sees a theme saved from another session (`test/theme.test.ts`).

Formulation rules:

- Name the condition being tested.
- Assert outward contract (session, state fields, HTTP JSON), not private
  room fields with no sync.
- Cover both success and failure when the SUT branches on them.

Examples:

```text
Verify JWT room connect:
- Valid JWT.sign payload: connectTo succeeds; client.sessionId === room.clients[0].sessionId
- Missing/invalid token: connect rejects / join fails

Verify seating / pieces (SC-PIECE):
- First join: seat has touristId 1…4 and exactly four pieces on N/E/S/W start cells
- Two seats: unique touristId; no shared (row,col) among any pieces
- Fourth seat → started true; fifth → no seat; leave before start frees kind+cells

Verify reconnect grace (SC-PIECE-11…16):
- Unexpected drop (client.reconnection.enabled=false; leave(false)): seat held;
  connected=false; reconnectUntil ≈ now+RECONNECT_GRACE_SECONDS*1000
- reconnect(token) within grace: same sessionId/touristId/pieces; connected=true; reconnectUntil=0
- Grace timeout: seat removed; kind/cells reusable before start
- Last seated permanent leave closes room even with spectators
- Observer sees connectivity fields sync (SC-PIECE-16)

Verify move / turn (SC-MOVE):
- First seated holds `currentTurnSessionId`; successful move advances join-order queue
- Legal orthogonal/diagonal one-step: piece row/col update; illegal/out-of-turn/spectator: unchanged
- Permanent leave of current advances turn; offline grace does not

Verify live lobby listing:
- joinOrCreate("lobby", { filter: { name: "tourist" } }); createRoom("tourist") → lobby receives +
- room.disconnect() → lobby receives -

Verify theme preference HTTP:
- No token / anonymous: GET/POST /api/theme reject (401/403)
- Registered: POST { theme: "dark" } → login userdata.theme === "dark"
- Registered: POST then GET /api/theme with same JWT → theme without re-login
- Cross-device: older session JWT GET returns theme saved from another session POST
```

## Coverage Topics

### Room connect with JWT

Canonical pattern (`test/MyRoom.test.ts`):

```typescript
describe("testing your Colyseus app", () => {
  let colyseus: ColyseusTestServer<typeof appConfig>;

  before(async () => (colyseus = await boot(appConfig)));
  after(async () => colyseus.shutdown());

  beforeEach(async () => {
    await colyseus.cleanup();
  });

  it("connecting into a room with JWT", async () => {
    const token = await JWT.sign({ id: 1, username: "test" });
    colyseus.sdk.auth.token = token;

    const room = await colyseus.createRoom("tourist", {});
    const client1 = await colyseus.connectTo(room);

    assert.strictEqual(client1.sessionId, room.clients[0].sessionId);
  });
});
```

Always use `createRoom("tourist", …)` matching `app.config.ts`. Include lobby live-list cases when changing listing / metadata.

### onAuth failure

- Omit `colyseus.sdk.auth.token`, or set a garbage string.
- Assert that `connectTo` / join fails (rejected promise or error). Prefer
  matching how neighboring tests assert rejections (`assert.rejects`).
- Do not weaken `MyRoom.onAuth` / `JWT.verify` to make the test pass.

### Seating / pieces sync (SC-PIECE)

Canonical coverage lives in `test/MyRoom.test.ts`:

- First join → exactly four pieces on sides N/E/S/W on that side’s start cells; `connected=true`, `reconnectUntil=0`.
- Unique `touristId` among seats; no shared cell among any pieces in the room.
- Fourth seated → `started === true`; fifth → no seat / no extra pieces.
- Consented leave before start → kind + cells reusable; after start → no reseat.
- Unexpected drop → hold seat for `RECONNECT_GRACE_SECONDS` (SC-PIECE-11…14); `reconnect(token)` restores online; grace timeout removes; empty seated → dispose with spectators (SC-PIECE-15); connectivity sync (SC-PIECE-16).

Helpers in that file (`assertFourPiecesOnSides`, `listPieces`, `allRoomPieces`, `unexpectedDrop`, `assertOfflineGrace`)
are the preferred assertion style — extend them rather than inventing a parallel
seat-flat `side`/`row`/`col` model. Grace-timeout cases may need `this.timeout(…)` above the default 15s.

### Turn / move (SC-MOVE)

Canonical coverage: `test/MyRoom.test.ts` (SC-MOVE-*) + pure `test/touristMove.test.ts`.

- First seated → `currentTurnSessionId`; join-order rotation after legal move; solo wraps to self.
- Legal orthogonal/diagonal one-step updates piece `row`/`col` and advances turn.
- Occupied / non-playable / out-of-turn / spectator → no state change.
- Permanent leave of current advances turn; `onDrop` grace does not change turn.
- Pure module tests cover playable set, Chebyshev, occupancy without room I/O.

Server is authoritative; never assert by trusting a client-only board copy.

### Schema sync assertions

- After `connectTo`, read synced state from the client SDK view or room state
  the harness exposes; assert fields the client SPA expects:
  `started`, `seats` → `touristId` + four `pieces` `{ side, row, col }` +
  `connected` / `reconnectUntil`, and `currentTurnSessionId`.
- Do not assert legacy draughts fields (`board`, `currentTurn`,
  `players[].color`) — they are not in product schema.

### Live lobby listing (SC-LOBBY-02 / SC-LOBBY-03)

- Client lobby uses Colyseus built-in `LobbyRoom` with filter `name: tourist`.
- Tests: `joinOrCreate("lobby", { filter })`, wait for `+` on create and `-` on dispose.
- `createRoom("tourist", …)` and loadtest `--room tourist` must match `app.config.ts`.
- HTTP `GET /rooms/tourist` is optional fallback coverage, not the primary UI path.

## Test File Structure

```typescript
import assert from "assert";
import { ColyseusTestServer, boot } from "@colyseus/testing";
import { JWT } from "@colyseus/auth";
import appConfig from "../src/app.config.js";

describe("MyRoom", () => {
  let colyseus: ColyseusTestServer<typeof appConfig>;

  before(async () => {
    colyseus = await boot(appConfig);
  });

  after(async () => {
    await colyseus.shutdown();
  });

  beforeEach(async () => {
    await colyseus.cleanup();
  });

  it("should connect with a valid JWT", async () => {
    const token = await JWT.sign({ id: 1, username: "test" });
    colyseus.sdk.auth.token = token;

    const room = await colyseus.createRoom("tourist", {});
    const client = await colyseus.connectTo(room);

    assert.strictEqual(client.sessionId, room.clients[0].sessionId);
  });
});
```

Conventions:

- Top-level `describe` named after the room / area under test.
- Prefer `it("should …")` or the existing connecting wording; stay consistent
  within a file.
- Always `boot` once per suite, `cleanup` in `beforeEach`, `shutdown` in
  `after`.
- Use `assert` / `assert.strictEqual` / `assert.rejects` — not Jest `expect`.
- Room name in tests must match `app.config.ts` (`tourist`).

## Helpers (optional, keep in-file)

Prefer small local helpers in the test file over new packages:

```typescript
async function connectWithUser(
  colyseus: ColyseusTestServer<typeof appConfig>,
  roomName: string,
  user: { id: number; username: string },
) {
  const token = await JWT.sign(user);
  colyseus.sdk.auth.token = token;
  const room = await colyseus.createRoom(roomName, {});
  const client = await colyseus.connectTo(room);
  return { room, client, token };
}
```

Share helpers only when a second test file needs them (`test/helpers/…`);
default is a single `SKILL.md` and in-file helpers — no separate md required.

## Checklist Before Finishing

- Test plan covered; success and failure for auth/join (and move/schema when
  present).
- File is `test/**/*.test.ts` with relative imports to `src/**/*.js`.
- Harness: `boot` → `cleanup` → `shutdown`; JWT via `@colyseus/auth`.
- Room name matches `app.config.ts` registration (`tourist`; also cover `lobby` live-list when changing listing).
- No Jest / babel / Vitest APIs or config files.
- Ran `npm test` from the server package root; failures fixed before claiming done.

## Anti-Patterns

```typescript
// Jest / Vitest — wrong stack
import { describe, it, expect, jest } from "@jest/globals";
jest.mock("../src/rooms/MyRoom.js");
expect(x).toBe(y);

// babel-jest / .cjs tests — not used here
// test/MyRoom.test.cjs

// Skipping JWT when room uses onAuth
// colyseus.connectTo(room) without colyseus.sdk.auth.token

// Hard-coding a room name that does not match app.config.ts (`tourist` / `lobby`)
// without updating registration + tests + loadtest together

// Skipping npm test after adding or changing tests
```
