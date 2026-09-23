---
name: work-with-routes
description: >-
  Use when adding, changing, or reviewing HTTP routes on the happy-tourist
  Colyseus server: createRouter / createEndpoint in app.config.ts, Express
  hook handlers (/health, /hi), auth /auth/*, theme, support tickets, content
  packs (/api/content/* dual submit answers|tasks + draft statuses/tasksDirty/needsModeration + D5′ lock + staff nested hub), admin roles, or Colyseus room listing /rooms/:roomName.
  Keep HTTP thin — game logic belongs in rooms.
---

# Work With Routes

Use this skill when adding or changing **HTTP** handlers in `happy-tourist-server`.

This is a **realtime game server**, not a REST BFF. Prefer WebSocket room messages
for gameplay. HTTP is for health, demo/smoke, auth (provided by Colyseus), thin
profile preferences (e.g. UI theme), support tickets / content packs / admin roles,
and lobby room listing.

Stack: Express **5** (via `defineServer({ express })`), `createRouter` /
`createEndpoint` from `colyseus` / `@colyseus/tools`, TypeScript ESM (NodeNext).

**Temp skills path:** this skill lives under `.agents/skills/server/` in this
package for now. Canonical skills are intended to live in
`happy-tourist-meta/.agents/skills/server/` once meta is available — prefer that
path when choosing skills if it exists.

Configure HTTP in `src/app.config.ts`. Prefer not editing `src/index.ts`
(`listen` only) unless self-hosting requires it.

## Quick Reference

| Topic | Pattern |
|-------|---------|
| Where | `src/app.config.ts` — `defineServer({ routes, express })` |
| Typed/demo API | `routes: createRouter({ name: createEndpoint(path, opts, handler) })` |
| Health / smoke | `express(app)` → `app.get(...)` |
| Auth HTTP | `/auth/*` auto from `@colyseus/auth` when `database` is set — do not reimplement |
| Lobby listing | Live UI: `LobbyRoom` + `.enableRealtimeListing()`; HTTP `GET /rooms/:roomName` remains as fallback |
| CORS / monitor | Express hook — see `work-with-middleware` / `work-with-config` |
| Gameplay | **Not** HTTP — Room handlers + schema + messages |

## Rules

| Do | Don't |
|----|--------|
| Keep handlers thin (JSON / plain text, no board rules) | Put tourist move validation or board mutation in HTTP |
| Add demo/API-style endpoints via `createEndpoint` in `routes` | Invent a second Express router tree or BFF layer |
| Put ops smoke checks (`/health`, `/hi`) in the `express` hook | Duplicate `/auth/*` or reinvent JWT login as custom Express routes |
| Align registered room name (`tourist`) with client / HTTP path | Assume listing works for a room key the client does not use |
| Keep success bodies simple (`{ ... }` or plain text) | Invent `{ errorCode, errorMessage }` BFF envelopes (not used here) |
| Leave CORS first in `express` (credentials + GitHub Pages origin) | Mount game state behind REST “for convenience” |

## Two HTTP surfaces in `app.config.ts`

### 1. `routes` — `createRouter` + `createEndpoint`

Used for small named API endpoints wired by Colyseus tools:

```ts
routes: createRouter({
  api_hello: createEndpoint("/api/hello", { method: "GET" }, async () => {
    return { message: "Hello World" };
  }),
  // Example: JWT-gated preference (see api_theme / api_theme_get)
}),
```

- Key (`api_hello` / `api_theme` / `api_theme_get`) is an internal id; path/method come from `createEndpoint`.
- Handler returns a value → JSON response. Keep it side-effect light.
- Prefer this for new **public demo / thin JSON** endpoints that are not healthchecks.
- Authenticated thin reads/writes (profile prefs): `use: [auth.middleware()]`; writes validate body (e.g. zod) and update Drizzle by `auth.id`; reads SELECT by `auth.id`; reject missing JWT / `anonymous === true`.

### 2. `express(app)` — raw Express 5

Used for middleware and simple ops routes already in the scaffold:

```ts
app.get("/health", (_req, res) => {
  res.json({ status: "ok", uptime: process.uptime() });
});

app.get("/hi", (_req, res) => {
  res.send("It's time to kick ass and chew bubblegum!");
});
```

- CORS must stay **first** middleware in this hook.
- Non-prod: `monitor()` at `/monitor`, `playground()` at `/` — do not enable in production.

For CORS / monitor details use `work-with-middleware` and `work-with-config`.

## Route catalog (current + built-in)

| Method | Path | Source | Notes |
|--------|------|--------|-------|
| GET | `/api/hello` | `createEndpoint` (`api_hello`) | Demo JSON `{ message }` |
| GET | `/api/theme` | `createEndpoint` (`api_theme_get`) | `{ theme: 'light' \| 'dark' \| null }`; `auth.middleware()`; registered JWT only; SELECT `users.theme`; reject unauth / anonymous |
| POST | `/api/theme` | `createEndpoint` (`api_theme`) | Body `{ theme: 'light' \| 'dark' }`; `auth.middleware()`; registered JWT only; updates `users.theme`; reject unauth / anonymous |
| POST | `/api/auth/send-email-confirmation` | `createEndpoint` | `auth.middleware()`; registered non-anonymous; **60s cooldown only after successful send**; confirm JWT 30m → smtp.bz; already verified → no-op/reject; **no** auto-send on register |
| POST | `/api/auth/email` | `createEndpoint` | Body `{ email }`; change email + `emailVerified = false`; unique check; **no** auto-send; returns user/token |
| POST | `/api/auth/confirm-email` | `createEndpoint` | Unauthenticated JSON `{ token }` → JWT → `emailVerified`; product SPA confirm path |
| POST | `/api/auth/reset-password` | `createEndpoint` | Unauthenticated JSON `{ token, password }` → password policy → reset + bumpTokenVersion + one-time token; product SPA reset path |
| POST | `/api/auth/display-name` | `createEndpoint` | JWT; `{ displayName }` trim min 1 → `users.displayName` |
| POST | `/api/auth/change-password` | `createEndpoint` | JWT; `{ currentPassword, newPassword }` → Hash.verify + policy → set hash + bumpTokenVersion; reject if no password credential |
| POST | `/api/support/tickets` | `createEndpoint` | JWT any; `{ topic, body }` create + first message; create-ack mail (non-anonymous; distinct from status/auto-close); helpers in `src/lib/support.ts` |
| GET | `/api/support/tickets` | `createEndpoint` | JWT; own list |
| GET | `/api/support/tickets/:id` | `createEndpoint` | JWT; detail + messages (owner or staff) |
| POST | `/api/support/tickets/:id/messages` | `createEndpoint` | JWT; author/staff reply; closed rejects; author from `awaiting_response` → `in_progress` with `notify: false` (D4 — no status mail on self-reply bump) |
| POST | `/api/support/tickets/:id/close` | `createEndpoint` | JWT; author **or** staff close |
| GET | `/api/support/staff/tickets` | `createEndpoint` | JWT moderator\|admin; query `topic` (optional), `status`=`open`\|`closed`\|`all` (default `open`); cap ~50 **after** filter |
| POST | `/api/support/tickets/:id/take` | `createEndpoint` | JWT staff; take into `in_progress` |
| POST | `/api/support/tickets/:id/status` | `createEndpoint` | JWT staff; `{ status }` |
| GET | `/api/content/packs` | `createEndpoint` | JWT; approved live catalog (blocked still listed); helpers in `src/lib/content.ts` |
| POST | `/api/content/packs` | `createEndpoint` | JWT + non-anonymous + `emailVerified` (DB); create pack + draft into author collection |
| GET | `/api/content/packs/:id` | `createEndpoint` | JWT; live approved snapshot |
| GET\|POST | `/api/content/packs/:id/draft` | `createEndpoint` | JWT + verified editor; get/put draft (statuses + `tasksDirty` / `needsModeration` + requestId; dirty-answers / D5′ pending lock) |
| POST | `/api/content/packs/:id/submit/answers` | `createEndpoint` | JWT + verified; answers submit (≥2 cards; one pending per `(pack, type)`; only pending author may resubmit) |
| POST | `/api/content/packs/:id/submit/tasks` | `createEndpoint` | JWT + verified; tasks submit (≥2 tasks, filled slots; deny if answers dirty; non-authors denied while answers pending) |
| GET\|POST | `/api/content/packs/:id/moderation` (+ `/messages`) | `createEndpoint` | JWT; type-scoped change-author ↔ staff thread (reject comment on reject) |
| POST | `/api/content/packs/:id/block` \| `/unblock` | `createEndpoint` | JWT moderator\|admin |
| GET | `/api/content/collection` | `createEndpoint` | JWT (incl. anonymous); own collection |
| POST | `/api/content/collection` \| `/remove` | `createEndpoint` | JWT; add/remove pack |
| GET | `/api/content/staff/pending` | `createEndpoint` | JWT moderator\|admin; **answers-pending** packs only |
| GET\|POST | `/api/content/staff/requests/:id` (+ `/approve` `/reject` `/cancel` `/messages`) | `createEndpoint` | JWT staff; answers hub (task-set list marks) → nested tasks; approve answers needs live tasks |
| GET | `/api/admin/users` | `createEndpoint` | JWT admin; **exclude** anonymous; include `emailVerified` (+ id/email/role/displayName) |

**Content moderation mail (D20):** links from `src/lib/content.ts` (`editorThreadLink`) go to embedded-thread pages — answers → `#/content/packs/:id/edit`, tasks → `#/content/packs/:id/tasks/:taskSetId` (fallback `#/…/edit`) — **not** bare `#/content/packs/:id/moderation`.
| POST | `/api/admin/users/:id/role` | `createEndpoint` | JWT admin; `{ role }`; **POST** (not PATCH); response user includes `emailVerified` (same as list) |
| GET | `/health` | `express` hook | `{ status, uptime }` — deploy / monitor |
| GET | `/hi` | `express` hook | Plain text smoke |
| * | `/auth/*` | `@colyseus/auth` | Present when `database: db` is set (register/login/forgot; built-in HTML not product UX) |
| GET | `/rooms/:roomName` | Colyseus | Available-rooms HTTP listing (fallback; live UI uses LobbyRoom) |
| GET | `/monitor` | `express` (non-prod) | Colyseus Monitor |
| * | `/` playground | `express` (non-prod) | Dev playground |

### Auth routes

Do **not** hand-roll register/login/anonymous in Express. With `database` on
`defineServer`, `@colyseus/auth` mounts `/auth/*`. Room access still uses JWT in
`Room.onAuth` (`server-work-with-auth`).

### Lobby listing vs room registration

Live client lobby uses `lobby` + `tourist` with `.enableRealtimeListing()` (`work-with-rooms`). HTTP `GET /rooms/tourist` remains available as fallback for the same registered name — do not reintroduce `my_room`.

## Layering

```
HTTP (thin)
  createEndpoint / express handlers
    → optional read-only status / demo data
    → NEVER authoritative board or move application

Gameplay
  Room + schema + messages (move, etc.)
```

Canonical product path for moves: client `send('move', { from, to })` → Room
handler → mutate synced state. Not `POST /api/move`.

## Response shapes

| Kind | Pattern | Examples |
|------|---------|----------|
| JSON object | return value from `createEndpoint` or `res.json(...)` | `/api/hello`, `/api/theme`, `/health` |
| Plain text | `res.send(string)` | `/hi` |
| Auth | shapes from `@colyseus/auth` | `/auth/*` |
| Errors | Keep simple; no ServiceError / BFF envelope | see `server-work-with-errors` |

## Adding a new HTTP endpoint

1. Ask: does this belong in a **Room message** instead? If gameplay → rooms, stop.
2. Choose surface:
   - Thin JSON API / demo → `createEndpoint` inside `createRouter` in `app.config.ts`.
   - Ops / health / middleware-adjacent → `express(app)` hook.
3. Do **not** add captcha/session Redis guards. Auth is Colyseus JWT + `/auth/*`.
4. Keep the handler short; no Drizzle board writes, no tourist rules.
5. Match existing response style (simple JSON or plain text).
6. If the client will call it, align path/method/body with `../happy-tourist.github.io` (or document that it is server-only smoke).
7. Run `npm run build` from the server package root when useful; smoke `/health` locally when a dev server is running; fix failures before claiming done.

Do **not** create `src/app/routes/routes.js`-style BFF trees, mappers, or Soap/axios layers.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Implementing `move` over REST | Use Room `onMessage('move', …)` |
| Reimplementing `/auth/register` in Express | Rely on `@colyseus/auth` + `database` |
| Copying cookie `checkSession` / `createError` BFF envelopes | Not this stack — JWT + thin JSON |
| Putting CORS after routes | Keep CORS first in `express` |
| Expecting listing for a room key the client does not use | Keep registration as `tourist` (+ live `lobby`) |
| Fat handlers with DB game stats “because HTTP is easy” | Prefer room lifecycle / dedicated thin endpoint only if product asks |
| Content moderation mail → bare `#/content/packs/:id/moderation` | D20: `editorThreadLink` → `#/…/edit` (answers) or `#/…/tasks/:taskSetId` (tasks) |

## Checklist for a new or changed route

1. Path lives in `src/app.config.ts` (`routes` and/or `express`) — not a new router package.
2. Handler is thin; no authoritative tourist logic.
3. Correct surface chosen (`createEndpoint` vs `express`).
4. No duplicate of `/auth/*` or room listing.
5. Response shape matches nearby endpoints.
6. Client contract updated only if the SPA will call the path.
7. Room name for listing matches `rooms` registration when touching lobby.

## Related skills

- `work-with-middleware` — CORS order, monitor/playground in the Express hook
- `work-with-config` — env, `defineServer` wiring, CORS origin
- `server-work-with-auth` — `/auth/*`, JWT, Room `onAuth`
- `work-with-rooms` — gameplay and messages (not HTTP)
- `server-work-with-errors` — simple HTTP / room failure patterns
- `server-work-with-structure` — where HTTP vs rooms vs db belong
