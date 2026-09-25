---
name: work-with-config
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus server config:
  env loading (.env.${NODE_ENV}), secrets (AUTH_SALT / JWT_SECRET /
  SESSION_SECRET / GOOGLE_CLIENT_* / SMTP_BZ_* / MAIL_FROM /
  AUTH_BACKEND_URL / CLIENT_APP_URL / BOOTSTRAP_ADMIN_IDS /
  DEFAULT_CONTENT_PACK_IDS), DATABASE_URL /
  PORT / NODE_ENV, src/app.config.ts defineServer wiring (incl. support/admin
  routes + bootstrap), src/config/auth.ts OAuth + email flows (getRuntimeAuth /
  configureAuthEmailFlows), CORS (ALLOWED_ORIGIN, credentials), /health /hi,
  or non-prod monitor / playground. Not for APP_NAME brand merge (not a BFF).
---

# Work With Config

Use this skill for **server-side** configuration in `happy-tourist-server`
(Colyseus multiplayer backend for board game «Счастливый турист»).

There is **no** `config.js` + `APP_NAME` brand merge (unlike Express BFF
packages). Runtime wiring lives in `src/app.config.ts`; contour values and
secrets live in env files loaded by `@colyseus/tools`.

Stack: Colyseus 0.18 (`defineServer` / `defineRoom` via `@colyseus/tools`),
`@colyseus/auth`, Express 5 (thin HTTP inside `defineServer`), TypeScript
(`"type": "module"`, NodeNext), Node `>= 22`.

Entry: `src/index.ts` → `listen(app)`. Prefer configuring rooms / DB / HTTP
in `src/app.config.ts`; avoid editing `index.ts` unless self-hosting requires
it.

**Temp skills path:** this skill lives under `.agents/skills/server/` in this
package for now. Canonical skills are intended to live in
`happy-tourist-meta/.agents/skills/server/` once meta is available — prefer
that path when choosing skills if it exists.

Sibling client: `../happy-tourist.github.io` (GitHub Pages origin
`https://happy-tourist.github.io`; local Colyseus/API on port `2567`).

## Quick Reference

| Case | Preferred pattern |
| --- | --- |
| Env load | `@colyseus/tools` loads `.env.${NODE_ENV}` if present, else `.env` |
| Secrets / contour | Env only — document in `.env.example`; never hardcode |
| Server wiring | `defineServer({ database, rooms, routes, express })` in `src/app.config.ts` |
| Room registration | `rooms: { lobby: defineRoom(LobbyRoom), tourist: defineRoom(MyRoom).enableRealtimeListing() }` |
| Thin HTTP API | `createRouter` + `createEndpoint` (e.g. `GET /api/hello`) |
| CORS | `ALLOWED_ORIGIN` prod string / else `true`; **first** middleware; credentials `true` |
| Health / smoke | `GET /health`, `GET /hi` in `express` hook |
| Monitor / Playground | Only when `NODE_ENV !== "production"` (`/monitor`, `playground()`) |
| Brand merge | **None** — no `APP_NAME` / `appData` / `config.js` |

## How Env Loading Works

```
NODE_ENV
   ↓
@colyseus/tools → .env.${NODE_ENV}  (if file exists)
                 else .env
   ↓
process.env → read in app.config.ts / listen (PORT)
```

Rules:

1. Locally prefer `.env.development` / `.env.production` copied from `.env.example`.
2. Production secrets stay on the server (or secret store) — do **not** commit real values; CI rsync excludes `.env*`.
3. Read `process.env` at wiring sites (`app.config.ts`, tools listen). Do not invent a second dotenv/`config.js` layer.
4. No brand blocks: one package, one tourist board-game product — contour differences are env-only (`NODE_ENV`, URLs/secrets, `DATABASE_URL`, `PORT`).

## Env Vars

From `.env.example`:

| Variable | Role |
| --- | --- |
| `AUTH_SALT` | Required secret for `@colyseus/auth` |
| `JWT_SECRET` | Required secret for JWT (room `onAuth` / auth routes) |
| `SESSION_SECRET` | Required secret for `@colyseus/auth` sessions |
| `GOOGLE_CLIENT_ID` | Google OAuth Web client ID (`auth.oauth.addProvider` in `src/config/auth.ts`) |
| `GOOGLE_CLIENT_SECRET` | Google OAuth Web client secret |
| `SMTP_BZ_HOST` / `SMTP_BZ_PORT` / `SMTP_BZ_USER` / `SMTP_BZ_PASS` | smtp.bz transport (`src/lib/mailer.ts`); host **`connect.smtp.bz`** (ports 2525/587 STARTTLS or 465/9465 SSL; mailer `secure` when port is 465 or 9465) |
| `MAIL_FROM` | From header for outbound mail |
| `AUTH_BACKEND_URL` | Public API origin → `auth.backend_url` (Google OAuth / API — **not** mail link base) |
| `CLIENT_APP_URL` | Client origin; mail links → `/#/confirm-email` and `/#/reset-password`; support create-ack/status mail → `/#/support/<id>`; SPA confirm → lobby |
| `BOOTSTRAP_ADMIN_IDS` | Comma-separated `colyseus_users.id`; on startup idempotent `ht_role='admin'` via `bootstrapAdminIds()` (env wins demotion) |
| `DEFAULT_CONTENT_PACK_IDS` | Comma-separated `content_packs.id`; one-shot grant into every user’s collection (boot backfill + create-user); only live + in-catalog + not blocked; empty = no-op; no re-grant after remove (`src/lib/defaultContentPacks.ts`) |
| `DATABASE_URL` | SQLite path (local `./game.db`; prod often under `/var/www/happy-tourist-server/game.db`) |
| `NODE_ENV` | `development` / `production` — picks env file, CORS origin, monitor/playground |
| `PORT` | Listen port (default `2567` via `@colyseus/tools` `listen`) |

Generate secrets locally with e.g. `openssl rand -base64 32`. Keep the same
`JWT_SECRET` across restarts so existing tokens stay valid during a contour.

### Env vs `app.config.ts`

| Belongs in **env** | Belongs in **`app.config.ts`** |
| --- | --- |
| Secrets (`AUTH_SALT`, `JWT_SECRET`, `SESSION_SECRET`, `GOOGLE_CLIENT_*`, `SMTP_BZ_*`, `MAIL_FROM`) | `defineServer` shape: `database`, `rooms`, `routes`, `express`; auth email configure + thin `/api/auth/*` + `/api/support/*` + `/api/admin/*` |
| Contour (`NODE_ENV`, `PORT`, `DATABASE_URL`, `AUTH_BACKEND_URL`, `CLIENT_APP_URL`, `BOOTSTRAP_ADMIN_IDS`, `DEFAULT_CONTENT_PACK_IDS`) | CORS middleware order and headers; support table ensure + bootstrap admins + default-pack backfill + auto-close interval |
| Anything that must change without a code change | `/health`, `/hi`, non-prod `monitor()` / `playground()` |
| | Room name → room class mapping; `createEndpoint` paths |

## `src/app.config.ts` Shape

```ts
const ALLOWED_ORIGIN =
  process.env.NODE_ENV === "production"
    ? "https://happy-tourist.github.io"
    : true; // any origin in development

const server = defineServer({
  database: db, // enables @colyseus/auth HTTP + user store
  rooms: {
    lobby: defineRoom(LobbyRoom),
    tourist: defineRoom(MyRoom).enableRealtimeListing(),
  },
  routes: createRouter({
    api_hello: createEndpoint("/api/hello", { method: "GET" }, async () => {
      return { message: "Hello World" };
    }),
  }),
  express: (app) => {
    // CORS first → /health → /hi → (non-prod) /monitor + playground
  },
});

export default server;
```

- `database: db` is enough for `@colyseus/auth` routes + user store.
- Prefer new rooms / endpoints / middleware here — not in `index.ts`.
- Path imports use explicit `.js` suffix (NodeNext).

## CORS

Client on GitHub Pages and this server are different origins; browsers need
CORS with credentials for auth cookies / credentialed fetches.

| Setting | Production | Development |
| --- | --- | --- |
| `ALLOWED_ORIGIN` | `"https://happy-tourist.github.io"` | `true` (reflect request origin or `*`) |
| Credentials | `Access-Control-Allow-Credentials: true` | same |
| Middleware order | **Must be first** in the `express` hook | same |

Implementation notes (match existing code):

- Resolve `Access-Control-Allow-Origin` from the string origin in prod, or from
  `req.headers.origin` / `"*"` when `ALLOWED_ORIGIN === true`.
- Allow methods: `GET, POST, OPTIONS`; handle `OPTIONS` with `204`.
- Allow headers include `Authorization` (JWT).
- Do not move CORS after other middleware; do not drop credentials when the
  client sends credentialed requests.

Sibling client env (for cross-checks only): `VITE_COLYSEUS_URL` /
`VITE_API_URL` point at this host — see client skill `work-with-env-deploy`.

## Health, Monitor, Playground

| Path | When | Role |
| --- | --- | --- |
| `GET /health` | Always | `{ status, uptime }` for deploy / monitoring |
| `GET /hi` | Always | Plain-text smoke check |
| `GET /monitor` | Non-production | Colyseus Monitor (`monitor()`) |
| playground at `/` | Non-production | `playground()` |

Gate with `process.env.NODE_ENV !== "production"`. Never enable monitor /
playground in production (exposes internal room tooling).

Auth HTTP (`/auth/*`) comes from `@colyseus/auth` when `database` is set — not
hand-rolled in the express hook. Colyseus also exposes room listing
`GET /rooms/:roomName` for the lobby.

## Patterns for Changing Config

### 1. New env variable

1. Add to `.env.example` with a short comment.
2. Set in local `.env.development` / server `.env.production` (not committed secrets).
3. Read `process.env.MY_VAR` at the wiring site (usually `app.config.ts` or DB init).
4. Document in `AGENTS.md` Config table if it is a long-lived contour knob.

### 2. New room or rename for client contract

1. Register in `rooms` inside `defineServer`.
2. Prefer client-aligned name `tourist` when implementing the real game.
3. Update tests / loadtest `--room` flags to match.

### 3. New thin HTTP endpoint

1. Prefer `createEndpoint` inside `createRouter` for JSON demo/API-style routes.
2. Use the `express` hook for middleware, health, or third-party mounts
   (`monitor`, `playground`).
3. Keep handlers thin — game truth stays in the Room.

### 4. CORS / origin change

1. Update `ALLOWED_ORIGIN` production string if the client host changes.
2. Keep credentials + first-middleware order.
3. Smoke from the real client origin (Pages ↔ server), not only same-origin curl.

## Examples

### Read contour in `app.config.ts`

```ts
const ALLOWED_ORIGIN =
  process.env.NODE_ENV === "production"
    ? "https://happy-tourist.github.io"
    : true;
```

### Keep CORS first

```ts
express: (app) => {
  app.use((req, res, next) => {
    /* Access-Control-* headers; OPTIONS → 204 */
    next();
  });
  app.get("/health", /* … */);
  // only then: other routes / monitor / playground
};
```

### Non-prod tooling only

```ts
if (process.env.NODE_ENV !== "production") {
  app.use("/monitor", monitor());
  app.use("/", playground());
}
```

### Add a secret

1. Document `MY_SECRET` in `.env.example`.
2. Set value on the deploy contour only.
3. Read `process.env.MY_SECRET` where needed — never commit the real value.

## Common Mistakes

| Mistake | Why it hurts | Fix |
| --- | --- | --- |
| Editing `index.ts` for rooms/HTTP | Fights Colyseus Cloud / tools entry convention | Wire in `app.config.ts` |
| CORS not first | Preflight / credentialed calls fail behind later middleware | CORS first in `express` hook |
| Prod monitor/playground | Leaks room internals | Gate on `NODE_ENV !== "production"` |
| Hardcoded secrets in source | Leak via git | Env + `.env.example` placeholders only |
| Inventing `APP_NAME` / `config.js` brand merge | Wrong model for this package | Env + `app.config.ts` only |
| Changing CORS origin without client host update | Browser blocks SPA ↔ server | Keep prod origin = Pages URL |
| Committing `.env.production` secrets | Credential leak | Keep prod env on VPS; rsync excludes `.env*` |
| Second dotenv in feature modules | Split sources of truth | Rely on `@colyseus/tools` load + `process.env` |

## Checklist

When changing or reviewing config-related work:

- [ ] New secrets/contour knobs documented in `.env.example`
- [ ] No real secrets committed; prod values only on server / secret store
- [ ] Rooms / routes / express changes land in `src/app.config.ts`
- [ ] `index.ts` left as `listen(app)` unless self-hosting truly requires more
- [ ] CORS remains **first** middleware; credentials enabled; prod origin correct
- [ ] `/health` (and smoke `/hi`) still work after express edits
- [ ] Monitor / playground still non-production only
- [ ] Room rename / new endpoints reflected in tests or loadtest if applicable
- [ ] No `APP_NAME` brand merge introduced

## Agent Workflow

1. Confirm whether the task is env, `defineServer` wiring, CORS, or monitor/playground.
2. Edit only the files the change requires (see map below).
3. Run npm commands from the server package root; fix failures before claiming done.

Typical commands (agent runs):

```bash
npm run build
npm test
npm run dev
```

Optional prod-shaped smoke (after build, with `.env.production` present locally or on server):

```bash
npm run start:prod
```

## Where Things Live

| Concern | Location |
| --- | --- |
| Env templates | `.env.example`, `.env.development`, `.env.production` |
| Env load | `@colyseus/tools` (before `listen`) |
| Server wiring | `src/app.config.ts` |
| Process entry | `src/index.ts` (`listen` only) |
| DB URL consumer | `src/db/` + `DATABASE_URL` |
| Auth secrets | `@colyseus/auth` via env |
| CORS / health / monitor | `express` hook in `src/app.config.ts` |
| High-level notes | `AGENTS.md` → Config And Env / HTTP Surface |
| Deploy cwd / PM2 | `ecosystem.config.cjs`, `.github/workflows/deploy.yml` |

Related skills (by name only): `work-with-routes`, `work-with-middleware`,
`work-with-database`, `server-work-with-auth`, `server-work-with-structure`,
`server-work-with-test`.
