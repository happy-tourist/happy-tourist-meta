---
name: server-work-with-test
description: >-
  Use when planning or writing mocha + @colyseus/testing tests for
  happy-tourist-server: room connect with JWT, onAuth failures, move messages,
  schema sync assertions, GET /rooms listing, or preference HTTP (GET/POST /api/theme).
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
- Schema sync: after join / move, client-visible state fields
  (`board`, `currentTurn`, `status`, `players[sessionId].color`) match
  authoritative room state (once implemented; align with client contract).
- Messages: `client.send("move", { from, to })` — legal move updates board;
  illegal move leaves state unchanged (and/or sends an error message if the
  room defines one).
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

Verify move (when implemented):
- Legal { from, to } on currentTurn: board cells update; turn flips
- Wrong turn / empty from / occupied to: state unchanged

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

### Future move validation

Once `onMessage("move", …)` exists:

- Two clients with JWT; assign colors via state; send `move` only on the
  current player’s turn.
- Assert schema `board` / `currentTurn` after a legal move.
- Assert no board change on illegal payloads (`from`/`to` out of range,
  wrong piece, mandatory capture ignored — when rules land).

Server is authoritative; never assert by trusting a client-only board copy.

### Schema sync assertions

- After `connectTo`, read synced state from the client SDK view or room state
  the harness exposes; assert fields the client SPA expects
  (`board`, `currentTurn`, `status`, `players`).
- Until schema is implemented, keep tests focused on join/auth; add schema
  assertions in the same change that introduces `MyRoomState` fields.

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
