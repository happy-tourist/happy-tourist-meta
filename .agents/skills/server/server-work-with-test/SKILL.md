---
name: server-work-with-test
description: >-
  Use when planning or writing mocha + @colyseus/testing tests for
  happy-tourist-server: room connect with JWT, onAuth failures, waiting-only
  seating / deferred pieces until playing / reconnect grace (SC-PIECE), move +
  steps/peeks / peek / endTurn / removed tiles (SC-MOVE-33…50 / SC-BOARD) +
  grille density / trap / rescue / push / returnFromFinish / all-jail /
  leave-clear (SC-LOBBY-14 / SC-BOARD-16/20 / SC-MOVE-51…64 /
  SC-MOVE-66…73 / SC-PIECE-24…28 / SC-FINISH-12/14) + catapult paced pipeline /
  deferred turn (SC-MOVE-78…93) + become-current grants
  (multi +1/+1; solo +1 step;
  already-current→solo no re-grant) + turn deadlines (setTurnBudgetsForTests) +
  trap presentation budgets (setTrapPresentationBudgetsForTests),
  center finish / finishPlace (SC-FINISH), preset say (SC-SAY), schema sync
  assertions, GET /rooms listing, preference HTTP (GET/POST /api/theme), auth
  email/password-policy (`zz-authEmail`) + profile displayName/change-password
  (`zz-authProfile` SC-PROFILE-*), support tickets + roles
  (test/support.test.ts — SC-SUP-* / SC-ROLE-*; mock mailer; BOOTSTRAP_ADMIN_IDS;
  incl. change_pack SC-SUP-27/28 + soft-unpublished reject D5/D9),
  or content packs (test/zz-contentPacks.test.ts — SC-PACK-* working copy +
  unified submit SC-PACK-100…105; add-task-set SC-PACK-108…110; staff lock/save
  SC-PACK-111…113; author delete SC-PACK-114; soft-unpublish/republish
  SC-PACK-120…124 (`in_catalog` pack) + task-set soft-hide SC-PACK-131 + preview live cards / authorDisplayName SC-PACK-134…135
  (`content_task_sets.in_catalog`; last-set 409); cascade; my-moderation +
  needs_revision; mock mailer; ensureContentTables / setEmailVerifiedForTests).
  Core workflow: test plan (mocks/verify) → write test/*.test.ts → run npm test
  from server package root and fix failures. Do not invent Jest/babel patterns.
trigger: slash
---

# Work With Test

Use this skill to plan and write integration-style tests for the Colyseus
server (`happy-tourist-server`). Start with a test plan, then implement tests
from that plan.

**Paths:** this skill lives in **happy-tourist-meta** at
`.agents/skills/server/server-work-with-test/`. Runtime `src/…` and `test/…`
paths are relative to the **happy-tourist-server** sibling root
(`../happy-tourist-server` from meta). Sibling client: `../happy-tourist.github.io`.

Stack: **mocha**, **tsx** (`-r tsx`), **`@colyseus/testing`**
(`ColyseusTestServer`, `boot`), **Node `assert`**, **TypeScript** ESM
(`"type": "module"`). Tests are `test/**.test.ts`.

Core principle: exercise the public contract of rooms / HTTP / auth as the
client will see them (JWT join, synced schema fields, `move`/`peek`/`endTurn`
accept/reject, private `budgets`/`peekOpen`, room listing), not private helpers.
Prefer extending `test/MyRoom.test.ts` patterns over inventing Jest, Vitest, or
babel setups.

## Mocha / Colyseus Testing Setup

| Piece | Role |
|-------|------|
| Script | `npm test` → `mocha -r tsx -r ./test/setupEnv.ts test/**.test.ts --exit --timeout 15000` |
| Runner | mocha + `@types/mocha`; load TS via tsx; `setupEnv.ts` sets `COLYSEUS_TESTING=1` |
| Harness | `@colyseus/testing`: `boot(appConfig)`, `cleanup()`, `shutdown()` |
| Auth | `JWT` from `@colyseus/auth` — `JWT.sign` then `colyseus.sdk.auth.token`; email flows use runtime JWT helpers |
| App under test | `import appConfig from "../src/app.config.js"` |
| Multi-suite HTTP | After each `boot`, call `keepLatestRequestListener(colyseus.server)` so stacked `request` listeners from reused `defineServer` do not double-fire (e.g. duplicate `sendEmail`) |

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
| Auth email flows (confirm / forgot / change-email / cooldown / password policy on register+reset) | `test/zz-authEmail.test.ts` — mock `setSendEmailImpl`; `clearConfirmSendCooldownForTests`; `keepLatestRequestListener` after boot; cover SC-AUTH-08 / SC-RESET-09 / SC-AUTH-10 where asserted |
| Auth profile (displayName + change-password) | `test/zz-authProfile.test.ts` — SC-PROFILE-01/02/04/05; bumpTokenVersion; reject no-password credential |
| Support tickets + roles (SC-SUP-* / SC-ROLE-*) | `test/support.test.ts` — mock mailer; `ensureSupportTables` / `bootstrapAdminIds` / `setUserRoleForTests` / `runAutoClose`; cover create-ack (SC-SUP-21), staff `topic`/`status` filters (SC-SUP-22), author self-reply **no** status mail, admin list excludes anonymous + `emailVerified`, role POST response includes `emailVerified` (SC-ROLE-09); `change_pack` in-catalog only + soft-unpublished → `pack_not_in_catalog` (D5/D9); `keepLatestRequestListener` after boot |
| Content packs (SC-PACK-*) | `test/zz-contentPacks.test.ts` — mock mailer; `ensureContentTables` / `setEmailVerifiedForTests` / `setUserRoleForTests`; cover catalog/collection, create+verify gate, working copy + unified submit SC-PACK-100…105, add-task-set SC-PACK-108…110, staff lock/save SC-PACK-111…113, author delete SC-PACK-114, soft-unpublish/republish SC-PACK-120…124 (`in_catalog` pack; keep live; ≠ block), task-set soft-hide SC-PACK-131 (`POST /api/content/task-set/unpublish|republish`; last published → `last_published_task_set`), cascade save, my-moderation + staff pending\|needs_revision, block endpoints + mail links; `keepLatestRequestListener` after boot |

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
  `phase` / `maxSeats` / `countdownRemaining` (+ legacy `started`), `seats` Map
  (`touristId` + `pieces` keyed by side `N|E|S|W` →
  `{ side, row, col, finished }` — **empty until playing** + `connected` /
  `reconnectUntil` / `ready` / `finishPlace` / `timeExpired`),
  `currentTurnSessionId`, `turnUntil`, `turnBudgetSeconds`,
  `nextFinishPlace`, and `removedTaskKeys`. Assert deferred pieces then
  materialize / free start cells; leave reopens seating **only in waiting**;
  join in countdown/playing is spectator (SC-PIECE-01…08 / SC-PIECE-19 in
  `test/MyRoom.test.ts`). Also cover reconnect grace SC-PIECE-11…16 and start
  capacity SC-START-* (maxSeats create, ready → countdown → playing; move gated
  on playing).
- Messages / turn / timer / budgets: cover SC-MOVE-* in `test/MyRoom.test.ts`
  (first seated holds turn; join-order rotation; legal orthogonal/diagonal;
  occupied/non-playable reject; out-of-turn / spectator / pre-playing / finished /
  time-expired / no-steps reject; **successful `move` does not advance turn**;
  `peek`/`peekAnswer`/`endTurn`; private `budgets`/`peekOpen`; grant multi +1/+1 /
  solo become-current +1 step (already-current→solo no re-grant — SC-MOVE-40/50);
  solo peeks∞ / finite steps; auto-end (keep turn when peeks∧live `*`); timeout force incorrect KEEP open peek; permanent leave
  advances; offline grace keeps turn for non-finished; deadline keeps ticking;
  multi 60s auto-pass; solo 300s → `timeExpired`; solo step-loss → `timeExpired`). Also SC-BOARD-07… (reward bag,
  Correct removes tile / Incorrect KEEP, holes **not landable**). Use `forcePlaying` (clears deadline) /
  `forcePlayingWithTimer` / `waitForPhase` helpers; accelerate clocks with
  `setTurnBudgetsForTests` + `resetTurnBudgets` in `beforeEach`/`afterEach`.
  Zero trap presentation budgets in `beforeEach` via
  `setTrapPresentationBudgetsForTests({ moveAnimMs: 0, catapultMs: 0, brokenMs: 0, grilleMs: 0 })`
  + `resetTrapPresentationBudgets` in `afterEach` so paced pipeline finishes inside short `waitMs`
  (override non-zero only for SC-MOVE-90…93 mid-hop / idle re-eval asserts).
  Pure rules: `test/touristMove.test.ts` (incl. finished occupancy /
  `finished` reject / `hasLegalMove` / `hasLegalPeek` / removed holes not landable).
- Finish: cover SC-FINISH-* (center land → `piece.finished`; 4th finish →
  `finishPlace = nextFinishPlace++`; turn skips finished / time-expired; all
  finished clears turn; join in playing is spectator (no mid-join seat); finished may still `say`).
- Preset say / ready: cover SC-SAY-* and ready-broadcast cases (known preset
  broadcast; `ready` message → say preset `ready` bypassing live cap; raw say
  `ready` rejected; non-whitelist / spectator / offline grace silent reject;
  max 3 live / `SAY_TTL_MS`). Assert via `waitForMessage('say')` — **not** schema.
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
- First join (waiting): seat has touristId 1…4 and **no pieces** until playing
- After enter playing / `forcePlaying`: exactly four pieces on N/E/S/W start cells
- Two seats: unique touristId; no shared (row,col) among any pieces after materialize
- Fill maxSeats → further joiner is spectator; leave while under maxSeats reopens seat **only in waiting**; join in countdown/playing is spectator (SC-PIECE-19)

Verify start / capacity (SC-START):
- createRoom("tourist", { maxSeats: 2|3|4 }) → state + metadata.maxSeats; invalid → 2
- metadata.seats = occupied seated count (not clients)
- Full table while waiting → countdown (COUNTDOWN_SECONDS) → playing (+ materialize + turn deadline)
- Underfilled ≥2: all ready → same countdown; solo ready rejected
- Leave/drop during countdown does not cancel; moves rejected until playing

Verify reconnect grace (SC-PIECE-11…16):
- Unexpected drop (client.reconnection.enabled=false; leave(false)): seat held;
  connected=false; reconnectUntil ≈ now+RECONNECT_GRACE_SECONDS*1000
- reconnect(token) within grace: same sessionId/touristId/pieces; connected=true; reconnectUntil=0
- Grace timeout: seat removed; kind/cells reusable; subsequent seat only while waiting
- Last seated permanent leave closes room even with spectators
- Observer sees connectivity fields sync (SC-PIECE-16)

Verify move / turn / timer / budgets (SC-MOVE + SC-BOARD):
- First seated holds `currentTurnSessionId`; successful `move` spends a step and
  **does not** advance the queue — advance via `endTurn` / auto-end / timeout /
  full-seat finish (skip finished / time-expired)
- Legal orthogonal/diagonal one-step: piece row/col update; illegal/out-of-turn/
  spectator/pre-playing/finished/time-expired/no-steps: unchanged
- `peek` → private `peekOpen`; `peekAnswer` Correct → +reward steps + `"r,c"` in
  `removedTaskKeys` (**not landable**); Incorrect KEEP tile + reward; multi peeks while peeks remain;
  solo peeks∞ / finite steps; become-current into solo → +1 step; already-current→solo carries steps; step-loss without live `*` → `timeExpired`
- Permanent leave of current advances turn; offline grace does not (non-finished);
  turnUntil keeps ticking; open peek → force incorrect KEEP on timeout/leave/endTurn
- ≥2 eligible: `turnBudgetSeconds===60`; timeout → advance without move; solo:
  `===300`; timeout → `timeExpired`
- `setTurnBudgetsForTests(multi, solo)` + `resetTurnBudgets()`; `forcePlaying` clears deadline; `forcePlayingWithTimer` uses `enterPlaying`

Verify finish (SC-FINISH):
- Legal move onto center → `piece.finished=true`; finished piece ignored for occupancy
- Seat’s 4th finished piece → `finishPlace = nextFinishPlace` then increment; seat stays seated
- Finished seat cannot `move`; may still `say`; turn never assigned to `finishPlace > 0`
- When every remaining seat is finished, `currentTurnSessionId === ""`; join in playing stays spectator (no mid-join seat)

Verify preset say (SC-SAY) + ready:
- Known `presetId` hello|luck → all clients get `say` `{ sessionId, presetId, at }`
- `ready` message → seat.ready + say preset `ready` (bypass live cap); raw say `ready` rejected
- Unknown / free text / spectator / offline grace → no broadcast
- Seated off-turn may say; spectators see seated say
- Fourth concurrent live say rejected; after `SAY_TTL_MS` may send again

Verify live lobby listing:
- joinOrCreate("lobby", { filter: { name: "tourist" } }); createRoom("tourist", { maxSeats }) → lobby receives +
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
    resetTurnBudgets();
    // Zero trap presentation so paced pipeline finishes within short waitMs.
    setTrapPresentationBudgetsForTests({
      moveAnimMs: 0,
      catapultMs: 0,
      brokenMs: 0,
      grilleMs: 0,
    });
    await colyseus.cleanup();
  });

  afterEach(() => {
    resetTurnBudgets();
    resetTrapPresentationBudgets();
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

Always use `createRoom("tourist", …)` matching `app.config.ts`. Include lobby live-list cases when changing listing / metadata. Import `setTrapPresentationBudgetsForTests` / `resetTrapPresentationBudgets` from `MyRoom` alongside turn-budget helpers.

### onAuth failure

- Omit `colyseus.sdk.auth.token`, or set a garbage string.
- Assert that `connectTo` / join fails (rejected promise or error). Prefer
  matching how neighboring tests assert rejections (`assert.rejects`).
- Do not weaken `MyRoom.onAuth` / `JWT.verify` to make the test pass.

### Seating / pieces sync (SC-PIECE)

Canonical coverage lives in `test/MyRoom.test.ts`:

- First join → exactly four pieces on sides N/E/S/W on that side’s start cells; `connected=true`, `reconnectUntil=0`, `ready=false`.
- Unique `touristId` among seats; no shared cell among any pieces in the room.
- Fill `maxSeats` → further joiners are spectators; leave while under maxSeats reopens a seat **only in waiting**; join in countdown/playing is spectator (SC-PIECE-19); leave/grace in playing does not reopen (SC-PIECE-08/14/21).
- Start: create `{ maxSeats }`; full table or all-ready underfilled → countdown → playing (SC-START-*).
- Unexpected drop → hold seat for `RECONNECT_GRACE_SECONDS` (SC-PIECE-11…14); `reconnect(token)` restores online; grace timeout removes; empty seated → dispose with spectators (SC-PIECE-15); connectivity sync (SC-PIECE-16).

Helpers in that file (`assertFourPiecesOnSides`, `listPieces`, `allRoomPieces`, `unexpectedDrop`, `assertOfflineGrace`, `forcePlaying`, `waitForPhase`)
are the preferred assertion style — extend them rather than inventing a parallel
seat-flat `side`/`row`/`col` model. Grace-timeout / countdown cases may need `this.timeout(…)` above the default 15s.

### Turn / move (SC-MOVE)

Canonical coverage: `test/MyRoom.test.ts` (SC-MOVE-*) + pure `test/touristMove.test.ts`.

- First seated → `currentTurnSessionId`; rotation only via `endTurn` / auto-end / timeout / finish-advance / leave-of-current (skip `finishPlace > 0` / `timeExpired`); solo non-finished stays current.
- Legal orthogonal/diagonal one-step updates piece `row`/`col` and spends 1 step — **does not** advance turn (SC-MOVE-35); only when `phase === 'playing'` + current + steps > 0 + seat eligible (use `forcePlaying` to skip countdown in isolation tests).
- Occupied / non-playable / out-of-turn / spectator / pre-playing / finished-seat / no-steps → no state change.
- Permanent leave of current → `applyTurnGrant` on next (multi +1/+1; solo become-current +1 step — SC-MOVE-50); already-current→solo carries steps (no re-grant — SC-MOVE-40); `onDrop` grace does not change turn (non-finished).
- Grilles (add-grille-traps): create `grilleDensity` few/medium/many → seed count 12/22/35% (SC-LOBBY-14 / SC-BOARD-16); land → trap + `holdingGrilleKeys` (SC-MOVE-51 / SC-PIECE-24/26); trapped rejects move/peek (SC-MOVE-52/53); `rescue` / `returnFromFinish` (SC-MOVE-54…59 / SC-FINISH-12/14); all-jail reset (SC-MOVE-60/61 / SC-PIECE-25); auto-end waits for rescue/return (SC-MOVE-62); solo same rules (SC-MOVE-64); spent grille leaves task peekable (SC-BOARD-20); permanent leave clears that seat’s holding (SC-PIECE-28; onDrop does not). Pure helpers cover density counts, ring cells, `hasLegalRescue` / `validateReturnFromFinish`.
- Push (add-tourist-push): `push` `{ pusherSide, targetSessionId, targetSide, row, col }` relocates target only (−1 step; no turn advance; land side-effects like move) — SC-MOVE-66…72; auto-end / solo step-loss count `hasLegalPush` (SC-MOVE-73); pure `farSideCell` / `validateTouristPush` / `hasLegalPush` in `test/touristMove.test.ts`.
- Catapults / paced trap pipeline (add-tourist-catapult): create `catapultDensity` same 12/22/35% ratios (SC-LOBBY-17); land → paced hop + presentation budget (SC-MOVE-78…89); **SC-MOVE-90** mid-overlay: piece still on catapult, dest grille not holding yet, `isTrapPipelineActiveForTests()`; **SC-MOVE-91/92** auto-end / deadline set `pendingTurnAdvance` until pipeline idle; **SC-MOVE-93** idle without pending **re-evals** auto-end after grille removes peek (land budget > 0 so post-move auto-end still sees free peek); reject further turn actions while pipeline active (board-lock parity). Helpers: `clearCatapultsForTests` / `plantHiddenCatapultForTests` / `setTrapRngForTests` / `setTrapPresentationBudgetsForTests`. With zeroed budgets, do **not** assert long-lived `revealingCatapultKeys` / `brokenCatapultKeys` after fire — assert pipeline idle instead.
- Pure module tests cover playable set, Chebyshev, occupancy (ignores finished; trapped still occupy) without room I/O.

Server is authoritative; never assert by trusting a client-only board copy.

### Finish (SC-FINISH)

Canonical coverage: `test/MyRoom.test.ts` (SC-FINISH-*) + finished cases in `test/touristMove.test.ts`.

- Center land → `piece.finished=true`; finished pieces do not occupy cells.
- 4th finished piece on a seat → `finishPlace = nextFinishPlace++`; seat remains seated.
- Finished seat rejects `move`; may still `say`; never holds `currentTurnSessionId`.
- All remaining seats finished → `currentTurnSessionId === ""`; join in playing stays spectator (SC-FINISH-07 / SC-MOVE-19).

### Preset say (SC-SAY) + ready

Canonical coverage: `test/MyRoom.test.ts` (SC-SAY-* / ready cases).

- Accept `hello` / `luck` → `broadcast('say', { sessionId, presetId, at })` to seated + spectators.
- `ready` message → `seat.ready` + say preset `ready` (bypass live cap); reject raw say `presetId: 'ready'`.
- Reject non-whitelist, spectator, offline-grace seat without broadcast (silent).
- Turn ownership not required; max `SAY_MAX_LIVE` (3) concurrent per session within `SAY_TTL_MS` (10s) — readiness broadcast bypasses the cap.
- Do **not** assert say via schema — use `client.waitForMessage('say')` / message listeners.
- TTL / countdown cases may need `this.timeout(…)` above the default.

### Schema sync assertions

- After `connectTo`, read synced state from the client SDK view or room state
  the harness exposes; assert fields the client SPA expects:
  `phase` / `maxSeats` / `countdownRemaining` (+ legacy `started`), `seats` →
  `touristId` + four `pieces` `{ side, row, col, finished, trapped }` + `connected` /
  `reconnectUntil` / `ready` / `finishPlace`, `currentTurnSessionId`,
  `removedTaskKeys`, `holdingGrilleKeys`, and `nextFinishPlace`.
- Do not assert legacy draughts fields (`board`, `currentTurn`,
  `players[].color`) — they are not in product schema.
- Do not assert hidden grille locations on schema — only revealed `holdingGrilleKeys`.

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
    resetTurnBudgets();
    setTrapPresentationBudgetsForTests({
      moveAnimMs: 0,
      catapultMs: 0,
      brokenMs: 0,
      grilleMs: 0,
    });
    await colyseus.cleanup();
  });

  afterEach(() => {
    resetTurnBudgets();
    resetTrapPresentationBudgets();
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
