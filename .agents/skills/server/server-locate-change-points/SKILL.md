---
name: server-locate-change-points
description: Finds files and exact places that need to be changed or where new files should be added in the happy-tourist Colyseus checkers server based on a task description. Use when the user asks to analyze a task, locate implementation points, find affected files, or identify where changes should be made without editing code.
---

# Locate Change Points

Use this skill when the user gives a task statement and wants only analysis of where changes are needed in this server package (`happy-tourist-server`).

Stack: Colyseus 0.18 (`defineServer` / `defineRoom` via `@colyseus/tools`), `@colyseus/auth` + JWT, `@colyseus/database` + drizzle-orm + better-sqlite3, `@colyseus/schema`, Express 5, TypeScript (`"type": "module"`, NodeNext), Node `>= 22`. Skills path for now: `.agents/skills/server/` in this repo (canonical copy may later live under `happy-tourist-meta`). Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

## Hard Rule

Do NOT edit, create, delete, format, or commit files.

Only inspect the codebase and report:

- existing files that likely need changes;
- exact rooms, schema fields, HTTP routes, auth/DB touch points, tests, loadtest, deploy/env involved;
- places where new files should be added, if the task requires new code;
- open questions when the implementation location cannot be determined confidently.

## Server Layer Map

Search and assign ownership top-down along the call path.

| Layer | Path | Role |
|-------|------|------|
| Entry | `src/index.ts` | `listen(app)` from `@colyseus/tools` |
| Server def | `src/app.config.ts` | `defineServer`: database, rooms, routes, express (CORS, `/health`, `/hi`, monitor/playground); side-effect import `./config/auth.js` |
| OAuth config | `src/config/auth.ts` | `auth.oauth.addProvider('google', …)`; leave built-in `onOAuthProviderCallback` alone |
| DB | `src/db/index.ts`, `src/db/schema.ts` | `GameDatabase`, `users` extension |
| Rooms | `src/rooms/MyRoom.ts` | `onAuth` / `onCreate` / `onJoin` / `onLeave` / `onDispose` |
| Schema | `src/rooms/schema/MyRoomState.ts` | `@colyseus/schema` sync state |
| Tests | `test/` | mocha + `@colyseus/testing` |
| Loadtest | `loadtest/example.ts` | `@colyseus/loadtest` |
| Deploy | `ecosystem.config.cjs`, `.github/workflows/deploy.yml`, `.env.*` | PM2, GH Actions |

Allowed dependency direction:

`app.config` (rooms / routes / express / config/auth) → room handler → schema state; auth userdata from JWT → `onJoin`; DB schema only for persisted profile fields used by `@colyseus/auth` / GameDatabase.

Prefer not editing `src/index.ts` unless self-hosting / listen details require it — configure rooms and HTTP in `src/app.config.ts`.

### Client contract gaps (current scaffold)

The sibling client already assumes a checkers contract; server is still scaffold. When locating change points for gameplay/lobby, prefer aligning server to client rather than inventing a parallel protocol.

| Client expectation | Server today |
|--------------------|--------------|
| Room type name `checkers` | Registered as `checkers` in `app.config.ts` with `.enableRealtimeListing()` |
| Live lobby (`LobbyRoom`) | `lobby: defineRoom(LobbyRoom)` — client filters `name: checkers` |
| State: `board`, `currentTurn`, `status`, `players[sessionId].color` | Scaffold `MyRoomState` (`mySynchronizedProperty`) |
| Message `move` `{ from, to }` | Not implemented yet |
| Cell values `0`–`4` (empty / white / black / kings) | Not implemented yet |
| Lobby `GET /rooms/checkers` | Available (HTTP fallback; UI uses live LobbyRoom) |

### HTTP surface (today)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/health` | `{ status, uptime }` — deploy/monitor |
| GET | `/hi` | Plain text smoke check |
| GET | `/api/hello` | Demo JSON via `createEndpoint` |
| POST | `/api/theme` | Body `{ theme: 'light' \| 'dark' }`; `auth.middleware()`; registered only; updates `users.theme` |
| * | `/auth/*` | Provided by `@colyseus/auth` when `database` is set (incl. `/auth/provider/google/callback`) |
| GET | `/rooms/:roomName` | Colyseus available-rooms listing (HTTP fallback; live UI uses LobbyRoom) |
| GET | `/monitor` | Dev only (`monitor()`) |
| * | `/` playground | Dev only (`playground()`) |

CORS middleware must stay first in the Express hook (GitHub Pages ↔ server cross-origin + credentials).

## Decision Guidance

Use these rules to pick the layer before naming files.

### HTTP endpoint vs room message vs schema field

| If the change is… | Prefer |
|-------------------|--------|
| New or changed HTTP path (health, demo API, custom Express) | `src/app.config.ts` — `routes` (`createEndpoint`) and/or `express(app)` middleware |
| Lobby room listing by name | Register `lobby` + `checkers` with `.enableRealtimeListing()`; HTTP `GET /rooms/:roomName` is fallback only |
| Realtime gameplay intent (move, resign, rematch, chat) | Room message handler in `src/rooms/MyRoom.ts` (`this.onMessage(...)`), not a new HTTP route |
| Synced board / turn / status / player seats visible to clients | `@colyseus/schema` in `src/rooms/schema/MyRoomState.ts` (+ room code that mutates state) |
| Authoritative rules / validation of moves | Room handler (`MyRoom.ts`); do not trust client board state |
| Room lifecycle (maxClients, seats, disconnect, dispose) | `MyRoom.ts` lifecycle hooks |

**Do not** put board truth or move validation in Express routes. Keep HTTP for auth (built-in), health, and non-realtime helpers; keep match state in schema + room messages.

### Auth vs DB column

| If the change is… | Prefer |
|-------------------|--------|
| Room join gate / JWT verify | `MyRoom.onAuth` (`JWT.verify`) — userdata flows to `onJoin` |
| Register / login / anonymous / Google OAuth HTTP | Built-in `@colyseus/auth` (`/auth/*`) via `database: db` in `app.config.ts`; Google via `src/config/auth.ts` `addProvider` — avoid reinventing unless extending |
| Persisted profile fields (`displayName`, `rating`, `gamesPlayed`, `gamesWon`, nullable `theme`, …) | `src/db/schema.ts` users extension — **NOT NULL** custom columns need `.default(...)` so `/auth/register` / `/auth/login` do not fail; nullable prefs like `theme` do not |
| GameDatabase wiring / schemas map | `src/db/index.ts` |
| Secrets for auth (salt, JWT, session, Google client) | `.env.example` / `.env.development` / `.env.production` (`AUTH_SALT`, `JWT_SECRET`, `SESSION_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`) |

### Deploy / env

| If the change is… | Prefer |
|-------------------|--------|
| Listen port / `NODE_ENV` / `DATABASE_URL` | `.env.*` (+ document in `.env.example`) |
| PM2 process / memory / cwd | `ecosystem.config.cjs` |
| CI build, rsync, remote `npm ci` / `build` / `pm2 reload` | `.github/workflows/deploy.yml` |
| CORS allowed origin (prod GitHub Pages) | Express CORS block in `src/app.config.ts` |
| Monitor / playground exposure | `app.config.ts` gated by non-production |

### Tests and loadtest

| If the change is… | Prefer |
|-------------------|--------|
| Room create/join, auth JWT connect | `test/MyRoom.test.ts` (update room name / contract when registration changes) |
| Theme preference HTTP | `test/theme.test.ts` (`POST /api/theme` auth + persist) |
| Multi-client join pressure | `loadtest/example.ts` (`joinOrCreate`; `--room` / `--numClients`) |

## Domain Hotspots

| Domain | Start here |
|--------|------------|
| Room registration / lobby name | `src/app.config.ts` `rooms` (`lobby` + `checkers` + `.enableRealtimeListing()`) |
| Auth to rooms | `MyRoom.onAuth` + `@colyseus/auth` JWT; secrets in `.env.*` |
| Google OAuth provider | `src/config/auth.ts` + side-effect import from `app.config.ts`; `GOOGLE_CLIENT_*` in `.env.*` |
| User profile columns | `src/db/schema.ts` + `src/db/index.ts` |
| Game state sync | `src/rooms/schema/MyRoomState.ts` |
| Moves / match flow | `src/rooms/MyRoom.ts` messages + lifecycle; align with client `move` / board cells `0`–`4` |
| HTTP health / CORS / demo API | `src/app.config.ts` express + routes |
| Tests | `test/MyRoom.test.ts`, `test/theme.test.ts`, … |
| Loadtest | `loadtest/example.ts` |
| Deploy / PM2 / CI | `ecosystem.config.cjs`, `.github/workflows/deploy.yml`, `.env.production` (on server only) |

## Workflow

1. Read the task statement carefully and extract:
   - target feature or behavior;
   - entities (auth, lobby rooms, board/state, move messages, profile/DB, HTTP health, deploy/env);
   - whether the task changes existing behavior or adds a new flow;
   - whether the client contract (room name `checkers`, state shape, `move`, `/rooms/checkers`) is involved.

2. Search the codebase by domain terms from the task:
   - room name / registration (`lobby`, `checkers`, `LobbyRoom`, `enableRealtimeListing`, `defineRoom`, `rooms:`);
   - room hooks (`onAuth`, `onCreate`, `onJoin`, `onLeave`, `onDispose`, `onMessage`);
   - schema symbols (`MyRoomState`, `Schema`, `type`, `MapSchema`, board/turn/status/players);
   - auth (`JWT.verify`, `@colyseus/auth`, `AUTH_SALT`, `JWT_SECRET`);
   - DB (`GameDatabase`, `users`, `displayName`, `rating`, `gamesPlayed`, `gamesWon`, `theme`);
   - HTTP (`/health`, `/hi`, `/api/hello`, `POST /api/theme`, `createEndpoint`, CORS, `monitor`, `playground`);
   - tests / loadtest (`@colyseus/testing`, `joinOrCreate`);
   - deploy (`ecosystem.config.cjs`, `pm2`, `rsync`, `DATABASE_URL`).

3. Inspect nearby files to understand ownership along the layering path:
   - `app.config.ts` room key → `MyRoom` → `MyRoomState`;
   - JWT `onAuth` → userdata in `onJoin`;
   - schema fields clients will observe via `onStateChange`;
   - matching `test/` and `loadtest/` when room name or auth contract changes;
   - sibling `../happy-tourist.github.io` when protocol/schema/room listing must stay aligned — do not invent client files here.

4. Apply decision guidance to classify each hit as edit vs add, and note related schema/test/env/client touch points.

5. Stop after identifying the change points. Do not continue into implementation.

## Search Tips

| Look for… | Where / how |
|-----------|-------------|
| Room registration | `src/app.config.ts` — `rooms` map keys |
| Room lifecycle / messages | `src/rooms/MyRoom.ts` — hooks and `onMessage` |
| Sync state fields | `src/rooms/schema/MyRoomState.ts` |
| Auth gate | `onAuth` + `JWT.verify`; auth HTTP is framework-provided under `/auth/*` |
| OAuth providers (Google) | `src/config/auth.ts` (`addProvider`); import from `app.config.ts` |
| User columns / defaults | `src/db/schema.ts` |
| Express / CORS / health | `express(app)` in `src/app.config.ts` |
| Custom HTTP routes (`/api/hello`, `POST /api/theme`, …) | `routes` / `createEndpoint` in `src/app.config.ts` |
| Env secrets / DB path | `.env.example`, `.env.development`, `.env.production` |
| Tests | `test/**.test.ts` — boots `appConfig`, JWT, room name |
| Loadtest | `loadtest/example.ts` |
| Deploy | `ecosystem.config.cjs`, `.github/workflows/deploy.yml` |

## Output Format

Respond in this structure (Russian headings):

```markdown
## Где править
- `path/to/file.ts` — why this file is relevant and what kind of change is expected.

## Где добавить
- `path/to/NewFile.ts` — why a new file belongs here.

## Связанные места
- `path/to/related-file.ts` — why it should be checked during implementation.

## Вопросы
- Clarifying question, only if needed.
```

Omit empty sections. Keep each bullet specific and actionable. Prefer paths under `src/`, `test/`, `loadtest/`, plus deploy/env files when relevant. For client contract work, cite `../happy-tourist.github.io` under Связанные места (do not edit it from this skill).

## Confidence

Prefer saying "likely" when the location is inferred from naming or nearby patterns. Say "confirmed" only when the inspected code directly proves the relationship.
