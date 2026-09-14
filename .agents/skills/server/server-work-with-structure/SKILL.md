---
name: server-work-with-structure
description: >-
  Use when placing or moving code in the happy-tourist Colyseus server: rooms,
  schema state, db/user schema, app.config (defineServer), HTTP express /
  createEndpoint routes, tests, loadtest, or deciding where game logic vs sync
  fields vs auth belong. Prefer aligning room name/schema/messages with the
  client.
---

# Work With Structure

## Overview

Use this skill when creating or relocating code under `src` (and related
`test/` / `loadtest/` / deploy wiring). This is a **realtime Colyseus game
server**, not an Express BFF: authoritative seating, reconnect grace, and later move rules live in the Room;
the client mirrors seats/connectivity and renders pieces + presence on a local board layout.

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

Sibling client: `../happy-tourist.github.io` (room type `tourist`, board + pieces from synced seats;
move messages deferred until product rules land).

## Core Rules

1. Before adding folders, inspect a nearby peer in the **same layer** and match its file name, export style, and import path (`.ts` sources, `.js` in imports).
2. Create only the files the feature needs. Do not add empty stubs “for later”.
3. Place code by layer role (listen vs server wiring vs room logic vs sync schema vs DB vs thin HTTP).
4. Respect **allowed dependency direction** (see below). Never invert layers.
5. **Room** owns authoritative game logic and message handlers. **Schema** owns sync fields only.
6. Keep HTTP thin: `express` hook and/or `createEndpoint` — no fat REST BFF, no invented session/Redis auth.
7. Auth is `@colyseus/auth` (register/login/anonymous/Google + JWT) and Room `onAuth` → `JWT.verify`. Register OAuth providers in `src/config/auth.ts`. Do not invent express-session + Redis.
8. Prefer aligning **room name**, **state schema**, and **messages** with the client rather than changing the client unilaterally.

## Folder Map

| Layer | Path | Role |
|-------|------|------|
| Entry | `src/index.ts` | `listen(app)` only |
| Server wiring | `src/app.config.ts` | `defineServer`: `database`, `rooms`, `routes`, `express`; side-effect import of OAuth config |
| OAuth config | `src/config/` | `auth.ts` — `auth.oauth.addProvider('google', …)`; no custom `onOAuthProviderCallback` in MVP |
| Database | `src/db/` | `GameDatabase` (`index.ts`) + Drizzle user schema (`schema.ts`) |
| Rooms | `src/rooms/` | Room handlers (`onCreate` / `onJoin` / `onDrop` / `onReconnect` / leave / dispose; messages later) |
| Schema | `src/rooms/schema/` | `@colyseus/schema` synced state definitions |
| Tests | `test/` | mocha + `@colyseus/testing` (`*.test.ts`) |
| Loadtest | `loadtest/` | `@colyseus/loadtest` scripts |
| Deploy | `ecosystem.config.cjs`, `.github/workflows/` | PM2 + CI rsync |

Env templates: `.env.example`, `.env.development`, `.env.production` (do not commit prod secrets).

### Layer roles (detail)

| Layer | Owns | Does not own |
|-------|------|--------------|
| **`index.ts`** | Process listen | Room logic, HTTP handlers, schema |
| **`app.config.ts`** | Wire `database`, register rooms, thin `routes` / `express` (CORS first, health, dev monitor/playground); import `./config/auth.js` | Game rules, board mutation |
| **`config/`** | OAuth provider registration (`addProvider`) | Room gate, user schema, custom OAuth callback (leave built-in) |
| **`db/`** | SQLite GameDatabase; extend `colyseus_users` with defaults | Room messages; inventing a second auth store |
| **`rooms/`** | Auth gate (`onAuth`), seats, reconnect grace (`onDrop`/`onReconnect`), lifecycle; future game `onMessage` + state | Raw HTTP; client-trusted board |
| **`rooms/schema/`** | Sync fields (`started` + `seats` → `touristId` + `pieces` + `connected` / `reconnectUntil`; moves later) | Validation / rules / side effects |
| **`test/` / `loadtest/`** | Boot server / joinOrCreate clients | Production deploy secrets |

## Dependency Direction

Typical path:

```text
index.ts → app.config.ts
app.config.ts → db / rooms (defineRoom) / config/auth / thin HTTP
rooms → rooms/schema (+ JWT from @colyseus/auth)
config → @colyseus/auth (oauth.addProvider only)
db → @colyseus/database / drizzle schema only
```

Allowed:

```text
app.config   →  db, rooms, config/auth, colyseus tools (monitor, playground, createEndpoint)
config       →  @colyseus/auth (addProvider; no rooms / no express)
rooms        →  rooms/schema, @colyseus/auth (JWT), colyseus Room/Client APIs
rooms/schema →  @colyseus/schema only
db           →  @colyseus/database, drizzle (no rooms / no express)
test         →  app.config, @colyseus/testing, JWT
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
| HTTP handlers implementing game rules | Put rules in the Room message handler |

## Where New Code Belongs

Decide in this order:

1. **New room type / rename?** → `src/rooms/<Name>.ts` + register in `app.config.ts` `rooms` (playable key `tourist`; live list needs `lobby` + `.enableRealtimeListing()`).
2. **Synced state fields?** → `src/rooms/schema/<Name>State.ts` only; Room assigns/mutates them.
3. **Game messages?** → Room `onMessage(…)` when rules land — validate, apply, update schema.
4. **User profile columns?** → `src/db/schema.ts`: **NOT NULL** columns need `.default(...)` so `/auth/register` / `/auth/login` stay compatible; nullable prefs (e.g. `theme`) do not; wire via `src/db/index.ts` if needed.
5. **Thin HTTP (health, demo API, preference read/save)?** → `express` hook or `createEndpoint` in `app.config.ts` (e.g. `GET|POST /api/theme`). CORS stays first.
6. **Auth HTTP?** → Already from `@colyseus/auth` when `database` is set — do not reimplement `/auth/*`.
7. **OAuth provider (Google)?** → `src/config/auth.ts` via `auth.oauth.addProvider`; side-effect import from `app.config.ts`; do not override built-in `onOAuthProviderCallback` unless product asks.
8. **Room gate?** → static `onAuth` with `JWT.verify` on the Room class.
9. **Test?** → `test/<Name>.test.ts` (boot `appConfig`, JWT, create/connect room).
10. **Load script?** → `loadtest/` (update room name when registered name changes).
11. **PM2 / CI?** → `ecosystem.config.cjs` / `.github/workflows/` only for deploy process — not game logic.

### Rooms vs schema vs HTTP — what belongs where

**Put in rooms**

- `maxClients`, seats, disconnect / forfeit / reconnect policy (`onDrop` grace vs consented `onLeave`).
- Future game `onMessage` handlers; reject illegal actions once rules exist.
- Authoritative state updates and status transitions when rules land.
- `onAuth` JWT verify; use returned userdata in `onJoin`.

**Put in rooms/schema**

- Fields the client must sync once product decides (scaffold OK today).
- Encoding / shape: align with client when rules land (not draughts encoding as product canon).
- Defaults via schema helpers — no rule engines here.

**Put in db**

- Extend `users` (`displayName`, `rating`, `gamesPlayed`, `gamesWon`, nullable `theme`, …); NOT NULL columns need defaults.
- `GameDatabase` connection string from `DATABASE_URL`.

**Put in app.config / index / config**

- `index.ts`: `listen` only.
- `app.config.ts`: register rooms, attach `database`, thin routes, CORS + `/health` + dev monitor/playground; side-effect import `./config/auth.js`.
- `src/config/auth.ts`: OAuth `addProvider('google', …)` only — leave built-in OAuth callback alone.

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

### Database

```text
src/db/
├── index.ts    # export const db = new GameDatabase(...)
└── schema.ts   # users = tables.sqlite.users("colyseus_users", { ... })
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
loadtest/example.ts
ecosystem.config.cjs
.github/workflows/deploy.yml
```

## Real Composition Examples

**Listen** — `index.ts` imports `./app.config.js` and calls `listen(app)`.

**Wire room** — `app.config.ts` `rooms: { lobby: defineRoom(LobbyRoom), tourist: defineRoom(MyRoom).enableRealtimeListing() }`.

**Auth gate** — `MyRoom.onAuth` → `JWT.verify(token)` → userdata to `onJoin`.

**HTTP** — CORS middleware first in `express`; `/health` JSON; `createEndpoint("/api/hello", …)` demo; `createEndpoint` `GET|POST /api/theme` JWT + registered theme preference; `/auth/*` from `@colyseus/auth` via `database: db`.

**Test** — `boot(appConfig)`, `JWT.sign(...)`, `createRoom("tourist")`, `connectTo`, assert `sessionId`; lobby `+`/`-` cases when listing changes.

**Client contract** — room `tourist` + live `lobby`; static Game board on client; synced rules / messages later; HTTP `/rooms/tourist` is fallback only.

## Creating New Pieces — Checklist

**New / product tourist room**

1. Implement Room handler under `src/rooms/` (lifecycle today; `onMessage` when rules land).
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
