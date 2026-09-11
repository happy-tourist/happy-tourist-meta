---
name: server-work-with-auth
description: >-
  Use when adding, changing, reviewing, or debugging server authentication:
  @colyseus/auth HTTP routes, Google OAuth addProvider, AUTH_SALT / JWT_SECRET /
  SESSION_SECRET / GOOGLE_CLIENT_*, users schema defaults, MyRoom.onAuth
  JWT.verify, register/login/anonymous/Google → JWT → room join, or auth
  userdata in onJoin for this Colyseus checkers server.
---

# Work With Auth

Use this skill for **authentication** in the happy-tourist server
(`happy-tourist-server`).

Auth is **token-based Colyseus Auth** (`@colyseus/auth`) + **JWT room gate**.
There is **no** cookie session, Redis hash, XSRF, captcha, SMS challenge, or
express-session layer.

**Important:** The client never imports `@colyseus/auth`. Client auth goes
through `client.auth` (`@colyseus/sdk`) and token key `colyseus-auth-token`.
This skill is **server-only** — do not apply Vue / Pinia / router patterns
here. Client side: `client-work-with-auth` in `../happy-tourist.github.io`.

**Temp skills path:** this skill lives under `.agents/skills/server/` in this
package for now. Canonical skills are intended to live in
`happy-tourist-meta/.agents/skills/server/` once meta is available — prefer
that path when choosing skills if it exists. Runtime `src/…` paths are
relative to this server repo root.

Related skills (by name — load when that area is in scope):
`server-work-with-errors`, `server-work-with-test`, `server-work-with-structure`,
`work-with-config`, `work-with-routes`, `work-with-middleware`,
`server-locate-change-points`, `server-verify-code`.

## Core Rule

Auth is **JWT issued by `@colyseus/auth`**, verified in the Room with
`JWT.verify`. HTTP auth routes are auto-mounted when `database` is set on
`defineServer`. Rooms gate with `static onAuth`; userdata flows into `onJoin`.

Keep:

- User store + schema in `src/db/`
- Auth secrets in env (`.env.example` / `.env.${NODE_ENV}`)
- Room gate in `MyRoom.static onAuth` → `JWT.verify(token)`
- Seat / profile use of auth payload in `onJoin` (room logic)

Do not invent Bearer axios middleware, Redis sessions, or cookie `checkSession`
guards. Do not put game rules in `/auth/*` handlers.

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Server def | `src/app.config.ts` | `database: db` enables `@colyseus/auth` HTTP routes + user store; side-effect import of `src/config/auth.ts` |
| OAuth providers | `src/config/auth.ts` | `auth.oauth.addProvider('google', …)`; do **not** override `onOAuthProviderCallback` (reuse built-in) |
| DB init | `src/db/index.ts` | `GameDatabase` + `schemas: { users }` |
| Users schema | `src/db/schema.ts` | Extends `colyseus_users`: `displayName`, `rating`, `gamesPlayed`, `gamesWon` |
| Room gate | `src/rooms/MyRoom.ts` | `static onAuth(token)` → `JWT.verify(token)` → userdata to `onJoin` |
| Secrets | `.env.example` / `.env.development` / `.env.production` | `AUTH_SALT`, `JWT_SECRET`, `SESSION_SECRET`, `DATABASE_URL`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` |
| HTTP auth | `/auth/*` (auto) | register / login / anonymous / OAuth provider callbacks from `@colyseus/auth` when `database` set |
| CORS | `app.config.ts` `express` hook | Credentials + `Authorization` allowed; must stay first |
| Tests | `test/MyRoom.test.ts` | `JWT.sign` → `sdk.auth.token` → create/connect room |

Prefer extending `users` with `.default(...)` on required custom columns so
built-in `/auth/register` and `/auth/login` do not fail on NOT NULL.

## End-To-End Auth Flow

```text
Client: register | login | signInAnonymously | signInWithProvider('google')
        │
        ▼
  POST /auth/*  (@colyseus/auth, enabled by defineServer { database: db })
  OAuth: /auth/provider/google/callback (built-in AuthService callback)
        │
        ├─ users table (SQLite / GameDatabase)
        └─ issues JWT (JWT_SECRET) → client stores as colyseus-auth-token
        │
        ▼
  Client joinOrCreate / join → SDK attaches token
        │
        ▼
  MyRoom.static onAuth(token, options, context)
        │
        ├─ JWT.verify(token)  → userdata
        └─ throw on invalid → client cannot connect
        │
        ▼
  onJoin(client, options, auth)  // auth === verified userdata
        │
        └─ seat colors / profile fields from auth (not from trusted client body)
```

Supported sign-in modes (Colyseus Auth): **email/password**, **anonymous**, and **Google OAuth** (`addProvider('google')`).
All yield a JWT the room gate treats the same way.

### Step-by-step (happy path)

1. `defineServer({ database: db, … })` wires GameDatabase → `@colyseus/auth` mounts `/auth/*`.
2. Client registers, logs in, signs in anonymously, or completes Google OAuth via SDK → server validates and returns JWT.
3. Client persists token as `colyseus-auth-token` and attaches it on room connect.
4. `MyRoom.onAuth` calls `JWT.verify(token)`; success returns userdata; failure throws → join rejected.
5. `onJoin` receives `auth` (userdata) — use it for seats / identity; do not trust client-supplied identity fields over `auth`.

## Secrets And Env

Required for `@colyseus/auth` (see `.env.example`):

| Variable | Role |
|----------|------|
| `AUTH_SALT` | Password hashing salt |
| `JWT_SECRET` | Sign / verify JWTs (`JWT.verify` / `JWT.sign`) |
| `SESSION_SECRET` | Auth package session secret |
| `DATABASE_URL` | SQLite path (`./game.db` local; prod often under `/var/www/…`) |
| `GOOGLE_CLIENT_ID` | Google OAuth Web client ID (`auth.oauth.addProvider`) |
| `GOOGLE_CLIENT_SECRET` | Google OAuth Web client secret |

Authorized redirect URI pattern (set in Google Cloud Console, not in code):

- Dev: `http://localhost:2567/auth/provider/google/callback`
- Prod: `https://<api-host>/auth/provider/google/callback`

`@colyseus/tools` loads `.env.${NODE_ENV}` if present, else `.env`. Generate
secrets with `openssl rand -base64 32`. Never commit production secrets;
keep `.env.production` on the server only.

## Google OAuth Provider

File: `src/config/auth.ts` (side-effect import from `app.config.ts` before listen).

```ts
import { auth } from "@colyseus/auth";

auth.oauth.addProvider("google", {
  key: process.env.GOOGLE_CLIENT_ID,
  secret: process.env.GOOGLE_CLIENT_SECRET,
  scope: ["email", "profile"],
});
```

- Reuse the built-in AuthService OAuth callback — do **not** set a custom
  `onOAuthProviderCallback` for MVP.
- Callback path is fixed by `@colyseus/auth`: `/auth/provider/google/callback`
  on the API host (same host as Colyseus HTTP).
- Client reaches Google via `client.auth.signInWithProvider('google')`
  (store `loginWithGoogle`); JWT room gate is unchanged.

## Users Schema Defaults

File: `src/db/schema.ts`.

Built-in `/auth/register` and `/auth/login` fill only standard Colyseus user
columns. Custom columns that are `NOT NULL` **must** have `.default(...)`:

```ts
export const users = tables.sqlite.users("colyseus_users", {
  displayName: text("display_name"),
  rating: integer("rating").notNull().default(1000),
  gamesPlayed: integer("games_played").notNull().default(0),
  gamesWon: integer("games_won").notNull().default(0),
});
```

| Field | Notes |
|-------|--------|
| `displayName` | Optional text; may map from register `options.name` / client display |
| `rating` | Default `1000` |
| `gamesPlayed` / `gamesWon` | Default `0` |

When adding new required profile columns, always add `.default(...)` or
register/login will break with NOT NULL.

## Room Gate (`onAuth` → `onJoin`)

File: `src/rooms/MyRoom.ts`.

```ts
import { JWT } from "@colyseus/auth";

static async onAuth(token: string, _options: any, _context: any) {
  const userdata = await JWT.verify(token);
  return userdata;
}

onJoin(client: Client, _options: any, auth: any) {
  // auth is verified userdata from onAuth — use for seats / identity
}
```

- Invalid / missing token → `JWT.verify` throws → Colyseus rejects the connection.
- Do not skip `onAuth` for “open” lobbies without an explicit product decision.
- Do not re-implement JWT parsing by hand; use `JWT` from `@colyseus/auth`.
- Tests: `JWT.sign({ … })` then `colyseus.sdk.auth.token = token` before `connectTo`
  (see `test/MyRoom.test.ts`).

## HTTP Surface (Auth)

| Surface | Notes |
|---------|--------|
| `/auth/*` | Provided by `@colyseus/auth` when `database` is set — do not duplicate |
| `/auth/provider/google/callback` | Built-in Google OAuth callback (API host); register URI in Google Console |
| Room join | Auth via JWT in `onAuth`, not Express middleware |
| CORS | Allow credentials + `Authorization`; keep CORS first in `express(app)` |

Game logic stays in the Room. Thin HTTP (`/health`, `/api/hello`) is unrelated
to challenge-login BFF patterns.

## Contract With Client

| Client (`happy-tourist.github.io`) | Server |
|------------------------------------|--------|
| `client.auth.registerWithEmailAndPassword` / `signInWithEmailAndPassword` / `signInAnonymously` / `signInWithProvider('google')` | `/auth/*` + Google provider via `@colyseus/auth` |
| Token key `colyseus-auth-token` | JWT verified in `onAuth` |
| Pinia `stores/auth`, LoginPage, router guards | **N/A on server** — do not port Vue patterns |
| Register options e.g. `{ name }` | Persist / map via users schema / Auth hooks if customized |

Coordinate userdata shape and register options with the client skill
`client-work-with-auth`. Change both sides together.

## Changing Auth Safely

1. Keep `database: db` on `defineServer` or `/auth/*` disappears.
2. Preserve `MyRoom.onAuth` → `JWT.verify` → return userdata to `onJoin`.
3. New `NOT NULL` user columns → `.default(...)` before deploy.
4. Keep `AUTH_SALT` / `JWT_SECRET` / `SESSION_SECRET` / `GOOGLE_CLIENT_*` in env templates and deploy notes.
5. CORS must continue to allow credentialed cross-origin auth from
   `https://happy-tourist.github.io` in production.
6. Update `test/MyRoom.test.ts` (and room name) when auth or room registration changes.
7. Sync register/login/anonymous/Google + userdata contract with the client — never only one side.
8. Prefer room `auth` payload for identity; do not trust client `options` over JWT userdata.
9. Keep Google provider registration in `src/config/auth.ts`; do not custom-override OAuth callback for MVP.

## Common Mistakes

| Mistake | Why it hurts |
|---------|----------------|
| Overriding `onOAuthProviderCallback` for MVP | Breaks built-in callback; keep `addProvider` only |
| Adding `NOT NULL` user column without `.default` | `/auth/register` / `/auth/login` fail |
| Removing `database` from `defineServer` | Auth HTTP routes gone |
| Skipping or weakening `onAuth` | Unauthenticated room joins |
| Inventing cookie / Redis / captcha / SMS auth | Wrong stack; client is JWT + `client.auth` |
| Porting Pinia / Vue router guards to the server | Server has no Vue; gate is `onAuth` |
| Trusting join `options` for user id over `auth` | Spoofable identity |
| Committing real `JWT_SECRET` / prod `.env` | Secret leak |
| Changing only server userdata fields | Client store / LoginPage break |

## Change Checklist

When touching auth:

1. Which piece? `app.config` / `config/auth` OAuth / `db/schema` / secrets / `MyRoom.onAuth` / `onJoin` / tests / client contract.
2. `/auth/*` still enabled via `database: db`; Google still registered via `addProvider` side-effect.
3. Custom user columns still have safe `.default(...)` where `NOT NULL`.
4. `onAuth` still `JWT.verify(token)` and returns userdata.
5. `onJoin` still uses `auth` for identity / seats.
6. Env secrets documented in `.env.example` (incl. `GOOGLE_CLIENT_*` + redirect URI comment); prod secrets not committed.
7. CORS still allows credentials + client origin in production.
8. Tests still sign JWT and connect with `sdk.auth.token`.
9. Contract synced with `../happy-tourist.github.io` (`client-work-with-auth`).
10. Built-in OAuth callback left untouched (no custom `onOAuthProviderCallback` for MVP).

## Do / Don't

| Do | Don't |
|----|--------|
| Use `@colyseus/auth` + JWT for all auth | Invent express-session + Redis / cookie challenge login |
| Gate rooms with `static onAuth` → `JWT.verify` | Trust client-only auth or skip room gate |
| Give custom `NOT NULL` columns `.default(...)` | Add required columns that break register/login |
| Keep secrets in env (`AUTH_SALT`, `JWT_SECRET`, `SESSION_SECRET`, `GOOGLE_CLIENT_*`) | Hardcode or commit production secrets |
| Pass verified userdata from `onAuth` into `onJoin` | Prefer join `options` over `auth` for identity |
| Coordinate with client `client.auth` / `colyseus-auth-token` | Import or mimic Vue / Pinia auth on the server |
| Support anonymous + email/password + Google via `addProvider` | Custom `onOAuthProviderCallback` or captcha / SMS unless product asks |
| Cover JWT connect in mocha + `@colyseus/testing` | Leave room auth untested after gate changes |
