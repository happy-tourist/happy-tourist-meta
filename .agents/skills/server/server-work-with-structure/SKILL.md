---
name: server-work-with-structure
description: >-
  Use when placing or moving code in the happy-tourist Colyseus server: rooms,
  schema state, db/user schema, app.config (defineServer), HTTP express /
  createEndpoint routes, tests, loadtest, or deciding where game logic vs sync
  fields vs auth belong (incl. content packs soft-unpublish pack+set/`in_catalog`
  + `previewPending` live cards + `authorDisplayName`). Prefer aligning room
  name/schema/messages with the client.
---

# Work With Structure

## Overview

Use this skill when creating or relocating code under `src` (and related
`test/` / `loadtest/` / deploy wiring). This is a **realtime Colyseus game
server**, not an Express BFF: authoritative seating, reconnect grace, turn order, and one-step move rules live in the Room;
the client mirrors seats/connectivity/`currentTurnSessionId` and renders pieces + presence + local move chrome + ephemeral say bubbles on a local board layout.

Stack: Colyseus 0.18 (`defineServer` / `defineRoom` via `@colyseus/tools`),
`@colyseus/auth` + JWT, `@colyseus/database` + Drizzle + better-sqlite3,
`@colyseus/schema`, Express 5 (thin HTTP inside `defineServer`), TypeScript
(`"type": "module"`, NodeNext), Node `>= 22`.

Entry: `src/index.ts` → `listen(app)`. Configure rooms / DB / HTTP in
`src/app.config.ts` — prefer not editing `index.ts` unless self-hosting
requires it.

Path style: relative imports with explicit `.js` suffix (NodeNext), e.g.
`import { MyRoom } from "./rooms/MyRoom.js"`.

**Temp skills path:** this skill lives under
`.agents/skills/server/` in this package for now. Canonical skills are
intended to live in `happy-tourist-meta/.agents/skills/server/` once meta
is available — prefer that path when choosing skills if it exists.

Sibling client: `../happy-tourist.github.io` (room type `tourist`, board + pieces from synced seats + turn/`sendMove`).

## Core Rules

1. Before adding folders, inspect a nearby peer in the **same layer** and match its file name, export style, and import path (`.ts` sources, `.js` in imports).
2. Create only the files the feature needs. Do not add empty stubs “for later”.
3. Place code by layer role (listen vs server wiring vs room logic vs sync schema vs DB vs thin HTTP).
4. Respect **allowed dependency direction** (see below). Never invert layers.
5. **Room** owns authoritative game logic and message handlers. **Schema** owns sync fields only.
6. Keep HTTP thin: `express` hook and/or `createEndpoint` — no fat REST BFF, no invented session/Redis auth.
7. Auth is `@colyseus/auth` (register/login/anonymous/Google + JWT + email confirm/forgot) and Room `onAuth` → `JWT.verify`. Auth config in `src/config/auth.ts` (`getRuntimeAuth` / `configureAuthEmailFlows`); mailer in `src/lib/`. Do not invent express-session + Redis.
8. Prefer aligning **room name**, **state schema**, and **messages** with the client rather than changing the client unilaterally.

## Folder Map

| Layer | Path | Role |
|-------|------|------|
| Entry | `src/index.ts` | `listen(app)` only |
| Server wiring | `src/app.config.ts` | `defineServer`: `database`, `rooms`, `routes`, `express`; import auth config; `configureAuthEmailFlows` after DB boot; thin `POST /api/auth/*` + `/api/support/*` + `/api/content/*` + `/api/admin/*`; boot `ensureSupportTables` / `ensureContentTables` / `bootstrapAdminIds` / soft-fail `backfillDefaultPacksOnBoot` / `startAutoCloseInterval` |
| Auth config | `src/config/` | `auth.ts` — `getRuntimeAuth` / Google `addProvider` / email hooks; wrap OAuth callback for `emailVerified` + one-shot default packs on new create; wrap anonymous/email register for default packs; map `htRole` → userdata `role` |
| Mailer / support / content / password policy | `src/lib/` | `mailer.ts` — smtp.bz `sendEmail` (+ test setter); `support.ts` — tickets/messages/roles/bootstrap/auto-close + `change_pack`/`pack_id` (**in-catalog** only) + create-ack mail + staff list filters + `setTicketStatus(..., { notify })` (HTTP stays thin); `content.ts` — working-copy draft CRUD + unified `submitPack`/approve + add-task-set + staff edit lock/save + soft-unpublish/republish pack + task-set (`in_catalog` on packs and sets; pack `unpublishPack` cascade-cancels open requests + one `notifyChangeAuthor` per author SC-PACK-137…141) + `previewPending` merges live `answerCards` for `task_set` (SC-PACK-134) + `authorDisplayName` on sets (SC-PACK-135) + needs-revision + `cascadeNormalizeTasks` (slot clear ≠ `answers_dirty`) + author delete + `listMyModerationPacks` / staff pending queue; migrate drop `content_user_drafts` + `ALTER` pack/set `in_catalog`; `defaultContentPacks.ts` — `DEFAULT_CONTENT_PACK_IDS` parse/eligible/grant + per-pack boot backfill (re-export from `content.ts`); `passwordPolicy.ts` — shared ≥8 + lower/upper/digit/symbol (register wrap / JSON reset / change-password) |
| Auth HTML | `html/` | Legacy Colyseus cwd templates; product confirm/reset UX is **SPA + JSON** (mail links via `CLIENT_APP_URL`) |
| Database | `src/db/` | `GameDatabase` (`index.ts`) + Drizzle user schema (`schema.ts`; `htRole` + support table decls) |
| Rooms | `src/rooms/` | Room handlers (`onCreate` / `onJoin` / `onDrop` / `onReconnect` / leave / dispose; `onMessage('move'|'ready'|'say')`) |
| Schema | `src/rooms/schema/` | `@colyseus/schema` synced state (`phase` / `maxSeats` / `countdownRemaining` + legacy `started` + `seats` + `currentTurnSessionId`) |
| Pure rules | `src/game/` | Authoritative move validate/apply (`touristMove.ts`) — no Colyseus I/O |
| Tests | `test/` | mocha + `@colyseus/testing` (`*.test.ts`; auth email + theme + room + `support.test.ts` + `zz-contentPacks.test.ts` + `zz-defaultContentPacks.test.ts`) |
| Loadtest | `loadtest/` | `@colyseus/loadtest` scripts |
| Deploy | `ecosystem.config.cjs`, `.github/workflows/` | PM2 + CI rsync |

Env templates: `.env.example`, `.env.development`, `.env.production` (do not commit prod secrets).

### Layer roles (detail)

| Layer | Owns | Does not own |
|-------|------|--------------|
| **`index.ts`** | Process listen | Room logic, HTTP handlers, schema |
| **`app.config.ts`** | Wire `database`, register rooms, thin `routes` / `express` (CORS first, health, auth email + support/content/admin endpoints, `configureAuthEmailFlows`, support + content table ensure); import `./config/auth.js` | Game rules, board mutation |
| **`config/`** | OAuth + email confirm/forgot hooks (`getRuntimeAuth`, wrap OAuth for verified); userdata `role` mapping | Room gate, user schema, from-scratch OAuth callback |
| **`lib/`** | Outbound mail (`mailer.ts` smtp.bz) + support helpers (`support.ts`) + content packs (`content.ts` + `defaultContentPacks.ts`) + password policy (`passwordPolicy.ts`) | Fat HTTP handlers, rooms |
| **`db/`** | SQLite GameDatabase; extend `colyseus_users` with defaults (`htRole`); declare support + `content_*` tables | Room messages; inventing a second auth store |
| **`rooms/`** | Auth gate (`onAuth`), seats, start phases/ready/countdown, reconnect grace, turn order (skip finished), `onMessage('move'|'ready'|'say')` + schema writes (move/ready/finish side-effects) / ephemeral broadcast (say) | Raw HTTP; client-trusted board; pure geometry tables (prefer `src/game/`) |
| **`rooms/schema/`** | Sync fields (`phase` / `maxSeats` / `countdownRemaining` + `seats` → `touristId` + `pieces` (+ `finished`) + connectivity/`ready`/`finishPlace` + `currentTurnSessionId` + `nextFinishPlace`; legacy `started`) | Validation / rules / side effects |
| **`game/`** | Pure tourist move rules (playable cells, Chebyshev, occupancy ignores finished) | Room lifecycle, schema `@type`, HTTP |
| **`test/` / `loadtest/`** | Boot server / joinOrCreate clients | Production deploy secrets |

## Dependency Direction

Typical path:

```text
index.ts → app.config.ts
app.config.ts → db / rooms (defineRoom) / config/auth / lib/mailer / lib/support / lib/passwordPolicy / thin HTTP
rooms → rooms/schema + game (pure rules) (+ JWT from @colyseus/auth)
config → @colyseus/auth (runtime singleton) + db + lib/mailer
lib → nodemailer / env / db (support helpers; no rooms)
db → @colyseus/database / drizzle schema only
game → no Colyseus / no express (pure functions only)
```

Allowed:

```text
app.config   →  db, rooms, config/auth, lib/mailer, lib/support, lib/passwordPolicy, colyseus tools (monitor, playground, createEndpoint)
config       →  @colyseus/auth (getRuntimeAuth), db, lib/mailer, lib/passwordPolicy (no rooms / no express game)
lib          →  nodemailer / env / db drizzle (support); no rooms / no fat express
rooms        →  rooms/schema, game/*, @colyseus/auth (JWT), colyseus Room/Client APIs
rooms/schema →  @colyseus/schema only
game         →  pure TS only (no rooms / schema / express)
db           →  @colyseus/database, drizzle (no rooms / no express)
test         →  app.config, game, lib/mailer mocks, lib/support test helpers, @colyseus/testing, JWT
loadtest     →  @colyseus/sdk / loadtest CLI
```

**Forbidden inversions:**

| Wrong | Why |
|-------|-----|
| `schema` → `rooms` / `app.config` / `db` | Sync defs must stay free of handlers |
| `db` → `rooms` / express | User store is not gameplay |
| `index.ts` growing with rooms/HTTP | Keep listen-only; wire in `app.config` |
| Fat REST “services” layer + Redis sessions | Use `@colyseus/auth` + Room `onAuth` JWT |
| Trusting client board state | Room validates and owns truth |
| HTTP handlers implementing game rules | Put pure rules in `src/game/`; Room `onMessage` only glues |

## Where New Code Belongs

Decide in this order:

1. **New room type / rename?** → `src/rooms/<Name>.ts` + register in `app.config.ts` `rooms` (playable key `tourist`; live list needs `lobby` + `.enableRealtimeListing()`).
2. **Synced state fields?** → `src/rooms/schema/<Name>State.ts` only; Room assigns/mutates them.
3. **Game messages / rules?** → pure module under `src/game/` + Room `onMessage(…)` glue that validates, applies, updates schema.
4. **User profile columns?** → `src/db/schema.ts`: **NOT NULL** columns need `.default(...)` so `/auth/register` / `/auth/login` stay compatible; nullable prefs (e.g. `theme`) do not; wire via `src/db/index.ts` if needed.
5. **Thin HTTP (health, demo API, preference, auth send-confirm / change-email / confirm-email / reset-password)?** → `express` hook or `createEndpoint` in `app.config.ts` (e.g. `GET|POST /api/theme`, `POST /api/auth/*`). CORS stays first.
6. **Auth HTTP?** → Built-in `/auth/*` from `@colyseus/auth` when `database` is set — do not reimplement register/login; confirm mail only via our send-confirm endpoint; product confirm/reset via JSON SPA endpoints (not API HTML).
7. **OAuth / email flows?** → `src/config/auth.ts` (`getRuntimeAuth`, `configureAuthEmailFlows`); wrap built-in OAuth callback for `emailVerified`; mailer in `src/lib/mailer.ts`.
8. **Room gate?** → static `onAuth` with `JWT.verify` on the Room class (no hard `emailVerified` gate).
9. **Test?** → `test/<Name>.test.ts` (boot `appConfig`, JWT, create/connect room; auth email → mock `setSendEmailImpl` + `keepLatestRequestListener`).
10. **Load script?** → `loadtest/` (update room name when registered name changes).
11. **PM2 / CI?** → `ecosystem.config.cjs` / `.github/workflows/` only for deploy process — not game logic.

### Rooms vs schema vs HTTP — what belongs where

**Put in rooms**

- `maxClients`, seats, disconnect / forfeit / reconnect policy (`onDrop` grace vs consented `onLeave`).
- `onMessage('move')` glue: seated + current turn → `validateTouristMove` → write piece `row`/`col` → advance turn.
- Authoritative turn order (`turnOrder` room-private) and seat/connectivity lifecycle.
- `onAuth` JWT verify; use returned userdata in `onJoin`.

**Put in `src/game/`**

- Pure playable-cell / adjacency / occupancy validate+apply (no Colyseus imports).

**Put in rooms/schema**

- Fields the client must sync once product decides (scaffold OK today).
- Encoding / shape: align with client (`{ side, row, col }` — not draughts encoding as product canon).
- Defaults via schema helpers — no rule engines here.

**Put in db**

- Extend `users` (`displayName`, `rating`, `gamesPlayed`, `gamesWon`, nullable `theme`, …); NOT NULL columns need defaults.
- `GameDatabase` connection string from `DATABASE_URL`.

**Put in app.config / index / config**

- `index.ts`: `listen` only.
- `app.config.ts`: register rooms, attach `database`, thin routes, CORS + `/health` + dev monitor/playground; side-effect import `./config/auth.js`.
- `src/config/auth.ts`: OAuth + email flows (`getRuntimeAuth`, `configureAuthEmailFlows`); wrap built-in OAuth callback only for `emailVerified`.
- `src/lib/mailer.ts`: smtp.bz outbound mail only.

### What NOT to put

| Avoid | Prefer |
|-------|--------|
| Game rules inside `express` / `createEndpoint` | Room message handlers |
| Sync field definitions inside the Room file when peers use `rooms/schema/` | `rooms/schema/*State.ts` |
| express-session + Redis (B2B-style) | `@colyseus/auth` + JWT `onAuth` |
| Editing `index.ts` for every feature | `app.config.ts` |
| Changing client unilaterally for room name/state/messages | Align server to client contract |
| Empty `services/` / BFF layer folders | Not used in this package |
| Custom columns without `.default(...)` | Breaks built-in register/login |
| Committing `.env.production` secrets | Keep secrets on the VPS only |

## Naming

| Kind | Convention | Examples |
|------|------------|----------|
| Source files | PascalCase for Room/State classes; camelCase modules as peers | `MyRoom.ts`, `MyRoomState.ts`, `schema.ts` |
| Room registration key | product name matching client | `lobby`, `tourist` (do not reintroduce `my_room`) |
| Schema export | `schema({ ... })` + `SchemaType<typeof …>` | `MyRoomState` |
| Imports | relative + `.js` (NodeNext) | `from "./rooms/MyRoom.js"` |
| Tests | `*.test.ts` under `test/` | `test/MyRoom.test.ts` |
| Loadtest | `loadtest/*.ts` | `loadtest/example.ts` |
| PM2 | `.cjs` at repo root | `ecosystem.config.cjs` |

Match existing peers; room keys `lobby` + `tourist` must stay aligned across `app.config.ts`, tests, and loadtest.

## Typical Shapes

### Entry + config

```text
src/index.ts          # listen(app) only
src/app.config.ts     # defineServer({ database, rooms, routes, express })
```

### Database + mail

```text
src/db/
├── index.ts    # export const db = new GameDatabase(...)
└── schema.ts   # users = tables.sqlite.users("colyseus_users", { ... })

src/lib/
├── mailer.ts          # smtp.bz sendEmail (+ setSendEmailImpl for tests)
├── support.ts         # tickets / roles / bootstrap helpers
├── content.ts         # content packs (ensureContentTables + working copy + submitPack/add-task-set + staff lock/save + soft-unpublish pack+set in_catalog + pack unpublish cascade-cancel open req + mail SC-PACK-137…141 + previewPending live cards + authorDisplayName + needs-revision + cascadeNormalize ≠ answers_dirty + author delete + listMyModeration/listPending)
├── defaultContentPacks.ts  # DEFAULT_CONTENT_PACK_IDS parse/eligible/grant + per-pack boot backfill (SC-PACK-142…147; re-export from content.ts)
└── passwordPolicy.ts  # shared product password policy (register / reset / change)

html/           # legacy Colyseus cwd templates (not product SPA UX)
```

### Room + schema

```text
src/rooms/
├── MyRoom.ts              # Room class + onAuth + lifecycle + messages
└── schema/
    └── MyRoomState.ts     # schema({ ... }) sync fields only
```

### Outside src

```text
test/MyRoom.test.ts
test/theme.test.ts
test/zz-authEmail.test.ts
test/zz-authProfile.test.ts
test/zz-contentPacks.test.ts
test/zz-defaultContentPacks.test.ts  # SC-PACK-142…147 DEFAULT_CONTENT_PACK_IDS
test/setupEnv.ts                     # COLYSEUS_TESTING + isolated game.test.db wipe
test/keepLatestRequestListener.ts
loadtest/example.ts
ecosystem.config.cjs
.github/workflows/deploy.yml
```

## Real Composition Examples

**Listen** — `index.ts` imports `./app.config.js` and calls `listen(app)`.

**Wire room** — `app.config.ts` `rooms: { lobby: defineRoom(LobbyRoom), tourist: defineRoom(MyRoom).enableRealtimeListing() }`.

**Auth gate** — `MyRoom.onAuth` → `JWT.verify(token)` → userdata to `onJoin`.

**HTTP** — CORS middleware first in `express`; `/health` JSON; `createEndpoint("/api/hello", …)` demo; `createEndpoint` `GET|POST /api/theme` JWT + registered theme preference; `POST /api/auth/send-email-confirmation` + `/api/auth/email` + `/api/auth/confirm-email` + `/api/auth/reset-password`; `/auth/*` from `@colyseus/auth` via `database: db` (built-in HTML not product UX).

**Test** — `boot(appConfig)`, `JWT.sign(...)`, `createRoom("tourist")`, `connectTo`, assert `sessionId`; lobby `+`/`-` cases when listing changes; auth email suite mocks mailer (`setSendEmailImpl`) and uses `keepLatestRequestListener` after multi-suite boot.

**Client contract** — room `tourist` + live `lobby`; synced `phase` / `maxSeats` / `countdownRemaining` / `seats` (+ `ready` / `finishPlace` / piece `finished`) / `currentTurnSessionId` / `nextFinishPlace` (+ legacy `started`) + `move` `{ side, row, col }` (center → finish side-effect) + `ready` + ephemeral `say` `{ presetId }` → broadcast `{ sessionId, presetId, at }`; HTTP `/rooms/tourist` is fallback only.

## Creating New Pieces — Checklist

**New / product tourist room**

1. Implement Room handler under `src/rooms/` (lifecycle + `onMessage('move')`; pure rules in `src/game/`).
2. Define sync state under `src/rooms/schema/` (scaffold OK; product fields with client lockstep).
3. Register room name in `app.config.ts` (`tourist` + live `lobby` when listed).
4. Keep `onAuth` JWT; set seating policy when product decides.
5. Update `test/` and `loadtest/` room name / expectations.
6. Align future message/state shape with `../happy-tourist.github.io`.

**New sync field**

1. Add to schema file only.
2. Mutate from Room after validation — never from HTTP or client trust.

**New user column**

1. Extend `src/db/schema.ts` — NOT NULL columns get `.default(...)`; nullable prefs (e.g. `theme`) may omit default.
2. Do not break `@colyseus/auth` register/login column set.

**New HTTP endpoint**

1. Prefer `createEndpoint` in `routes` or a thin `app.get/post` in `express`.
2. Do not put tourist rules there. Keep CORS first.

**New test / loadtest**

1. `test/<Name>.test.ts` via `@colyseus/testing` + JWT.
2. `loadtest/` for scripted joins; pass `--room` / `--numClients` via npm script.

## Domain Anchors

| Domain | Primary paths | Notes |
|--------|---------------|-------|
| Process entry | `src/index.ts` | `listen` only |
| Server wiring | `src/app.config.ts` | DB, rooms, thin HTTP, CORS |
| Auth HTTP | `@colyseus/auth` via `database` | `/auth/*` — do not reimplement |
| Room auth | `rooms/*.ts` `onAuth` | `JWT.verify` |
| Users / rating / theme | `src/db/schema.ts` | NOT NULL → defaults; nullable prefs OK |
| Gameplay | `src/rooms/*` + `rooms/schema/*` | authoritative rules + sync |
| Lobby list | `lobby` LobbyRoom + `tourist` `.enableRealtimeListing()` | HTTP `/rooms/:roomName` is fallback |
| Health / smoke | `express` `/health`, `/hi` | deploy checks |
| Dev tools | `monitor`, `playground` | non-production only |
| Client SPA | `../happy-tourist.github.io` | coordinate room/state/messages |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Growing `index.ts` with rooms or routes | Wire in `app.config.ts` |
| Putting game validation in HTTP | Room `onMessage` |
| Schema file with game-rule functions | Keep sync fields only |
| Inventing Redis/session auth like a BFF | `@colyseus/auth` + JWT `onAuth` |
| Reintroducing `my_room` or omitting `.enableRealtimeListing()` | Keep `lobby` + `tourist` registration |
| Custom DB columns without defaults | Add `.default(...)` |
| Imports without `.js` suffix | NodeNext: `from "./X.js"` |
| Leaving tests on old room name after rename | Update `test/` + `loadtest/` |
| Trusting client-sent board snapshots | Recompute/validate on server |

## Related Skills

- Sibling client placement → `../happy-tourist.github.io/.agents/skills/client/client-work-with-structure`
- Canonical copies (when meta exists) → `happy-tourist-meta/.agents/skills/server/`

## Verification

For structure-only placement tasks, confirm:

- [ ] Correct layer (`index` / `app.config` / `db` / `rooms` / `rooms/schema` / `test` / `loadtest`)
- [ ] Dependency direction respected (no schema → room, no db → room)
- [ ] Room owns rules + messages; schema owns sync fields only
- [ ] HTTP stays thin; no Redis session invention
- [ ] Auth via `@colyseus/auth` + `onAuth` JWT
- [ ] Relative `.js` imports (NodeNext)
- [ ] Room name / state / messages aligned with client when implementing gameplay
- [ ] Tests/loadtest updated if registration name or auth contract changes
