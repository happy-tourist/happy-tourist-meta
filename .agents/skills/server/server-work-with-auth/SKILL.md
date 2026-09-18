---
name: server-work-with-auth
description: >-
  Use when adding, changing, reviewing, or debugging server authentication:
  @colyseus/auth HTTP routes, Google OAuth addProvider, email confirm/forgot
  callbacks (no auto onSendEmailConfirmation), smtp.bz mailer, emailVerified,
  POST /api/auth/send-email-confirmation + /api/auth/email, AUTH_SALT /
  JWT_SECRET / SESSION_SECRET / GOOGLE_CLIENT_*, users schema defaults,
  MyRoom.onAuth JWT.verify, register/login/anonymous/Google → JWT → room join,
  or auth userdata in onJoin for this Colyseus tourist server.
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
`work-with-database`, `work-with-env-deploy`, `server-locate-change-points`,
`server-verify-code`.

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
| Server def | `src/app.config.ts` | `database: db` enables `@colyseus/auth` HTTP routes + user store; import `src/config/auth.ts`; call `configureAuthEmailFlows()` after DB auth defaults; thin `POST /api/auth/*` endpoints |
| OAuth + email hooks | `src/config/auth.ts` | `getRuntimeAuth` / `getRuntimeJWT`; Google `addProvider`; **no** `onSendEmailConfirmation`; `onEmailConfirmed` / `onForgotPassword`; wrap built-in `onOAuthProviderCallback` for Google `emailVerified`; `auth.backend_url`; confirm HTML writer |
| Mailer | `src/lib/mailer.ts` | `sendEmail(to, subject, html)` via smtp.bz only (`SMTP_BZ_*`, `MAIL_FROM`); no Resend / `MAIL_PROVIDER`; `setSendEmailImpl` for tests |
| Auth HTML | `html/` | Confirm/reset templates under cwd (`address-confirmation.html` written at boot; email HTML assets) |
| DB init | `src/db/index.ts` | `GameDatabase` + `schemas: { users }` |
| Users schema | `src/db/schema.ts` | Extends `colyseus_users`: `displayName`, `rating`, `gamesPlayed`, `gamesWon`, nullable `theme`, `emailVerified` (default `false`) |
| Room gate | `src/rooms/MyRoom.ts` | `static onAuth(token)` → `JWT.verify(token)` → userdata to `onJoin` (**soft** verify — no `emailVerified` gate) |
| Secrets | `.env.example` / `.env.development` / `.env.production` | Auth + Google + mail (`SMTP_BZ_*`, `MAIL_FROM`) + `AUTH_BACKEND_URL` / `CLIENT_APP_URL` |
| HTTP auth | `/auth/*` (auto) | register / login / anonymous / OAuth + confirm-email / forgot / reset HTML from `@colyseus/auth` when `database` set |
| CORS | `app.config.ts` `express` hook | Credentials + `Authorization` allowed; must stay first |
| Tests | `test/` | Room JWT connect + `zz-authEmail.test.ts` (mock mailer); `keepLatestRequestListener` after multi-suite boot |

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
| `SMTP_BZ_HOST` / `PORT` / `USER` / `PASS` | smtp.bz transport for `src/lib/mailer.ts` |
| `MAIL_FROM` | From header (e.g. `Happy Tourist <noreply@happy-tourist.ru>`) |
| `AUTH_BACKEND_URL` | Public API origin → `auth.backend_url` (confirm/reset links) |
| `CLIENT_APP_URL` | Client origin; after successful confirm → `{CLIENT_APP_URL}/#/lobby` |

Authorized redirect URI pattern (set in Google Cloud Console, not in code):

- Dev: `http://localhost:2567/auth/provider/google/callback`
- Prod: `https://<api-host>/auth/provider/google/callback`

`@colyseus/tools` loads `.env.${NODE_ENV}` if present, else `.env`. Generate
secrets with `openssl rand -base64 32`. Never commit production secrets;
keep `.env.production` on the server only. Mail secrets stay on VPS — not in
GitHub Actions app secrets (see `work-with-env-deploy`).

## Runtime Auth Singleton (`getRuntimeAuth`)

Under `tsx`, `import '@colyseus/auth'` may resolve the package `@source`
entry while `GameDatabase.applyRouterDefaults` loads `build/index.mjs`.
Those are **different auth singletons**. Always wire email callbacks,
`backend_url`, and post-boot OAuth wrap on the **runtime (build)** copy via
`getRuntimeAuth()` / `getRuntimeJWT()` in `src/config/auth.ts`. Static
`addProvider` on the import may still run for tsx; `configureAuthEmailFlows()`
re-registers Google on the runtime copy.

Call `configureAuthEmailFlows()` once from the `express` hook **after**
database boot + `applyRouterDefaults` (not only as a top-level side-effect).

## Google OAuth Provider

File: `src/config/auth.ts` (`configureAuthEmailFlows` + static import).

```ts
auth.oauth.addProvider("google", {
  key: process.env.GOOGLE_CLIENT_ID,
  secret: process.env.GOOGLE_CLIENT_SECRET,
  scope: ["email", "profile"],
});
```

- Callback path is fixed by `@colyseus/auth`: `/auth/provider/google/callback`
  on the API host (same host as Colyseus HTTP).
- Client reaches Google via `client.auth.signInWithProvider('google')`
  (store `loginWithGoogle`); JWT room gate is unchanged.
- Google path marks `emailVerified = true` (trusted provider) by **wrapping**
  the built-in `onOAuthProviderCallback` after DB boot — do **not** replace it
  with a from-scratch callback; call the previous handler then set
  `emailVerified`.

## Email Confirm And Forgot (No Auto-Send On Register)

File: `src/config/auth.ts` + `src/lib/mailer.ts`.

**Do not** set `auth.settings.onSendEmailConfirmation`. In Colyseus 0.18 that
hook auto-sends mail on register; product sends confirm **only** from the
cabinet button via our HTTP endpoint.

| Hook / piece | Behavior |
|--------------|----------|
| `onSendEmailConfirmation` | **Unset** — register must not send mail |
| `onEmailConfirmed` | Set `emailVerified = true` for that email |
| `onForgotPassword` | `sendEmail` with reset HTML from AuthService |
| Confirm / reset UI | Built-in **server HTML** (`/auth/confirm-email`, `/auth/reset-password`) — no SPA pages |
| After confirm | Redirect to `CLIENT_APP_URL/#/lobby` (confirm success HTML written under `html/`) |
| `auth.backend_url` | From `AUTH_BACKEND_URL` (link host for tokens) |
| Mailer | `sendEmail` → smtp.bz only; mockable in tests via `setSendEmailImpl` |
| Legacy backfill | One-shot: existing non-anonymous users → `emailVerified = true` (config flag) |

Confirm-link helper (`buildConfirmEmailContent`: JWT `expiresIn: '30m'`, HTML
with link) is used by `POST /api/auth/send-email-confirmation`, not by register.

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
  theme: text("theme"), // nullable light | dark | unset — no NOT NULL / default required
  emailVerified: integer("email_verified", { mode: "boolean" })
    .notNull()
    .default(false),
});
```

| Field | Notes |
|-------|--------|
| `displayName` | Optional text; may map from register `options.name` / client display |
| `rating` | Default `1000` |
| `gamesPlayed` / `gamesWon` | Default `0` |
| `theme` | Nullable UI preference (`light` \| `dark`); written by `POST /api/theme`, read by `GET /api/theme` for registered users; also appears in userdata on next login (JWT alone is not enough for reload sync) |
| `emailVerified` | Default `false` for new email registrations; Google + legacy non-anonymous backfill → `true`; change-email resets to `false`; exposed in userdata |

When adding new **NOT NULL** profile columns, always add `.default(...)` or
register/login will break. Nullable prefs (like `theme`) do not need a default.

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
- Do **not** gate join on `emailVerified` (soft verify — play allowed unverified).
- Tests: `JWT.sign({ … })` then `colyseus.sdk.auth.token = token` before `connectTo`
  (see `test/MyRoom.test.ts`).

## HTTP Surface (Auth)

| Surface | Notes |
|---------|--------|
| `/auth/*` | Provided by `@colyseus/auth` when `database` is set — do not duplicate |
| `/auth/provider/google/callback` | Built-in Google OAuth callback (API host); register URI in Google Console |
| `/auth/confirm-email`, `/auth/reset-password`, `/auth/forgot-password` | Built-in HTML / API; confirm success redirects to client lobby |
| `POST /api/auth/send-email-confirmation` | `createEndpoint` + `auth.middleware()`; registered non-anonymous; **60s cooldown only after a successful send** (not on failure); in-flight guard; confirm JWT 30m; already verified → no-op/reject; **only** path that sends confirm mail |
| `POST /api/auth/email` | Change email; unique check; `emailVerified = false`; **no** auto-send; returns updated user/token for client |
| Room join | Auth via JWT in `onAuth`, not Express middleware |
| CORS | Allow credentials + `Authorization`; keep CORS first in `express(app)` |

Game logic stays in the Room. Thin HTTP (`/health`, `/api/hello`, `GET|POST /api/theme`,
auth send-confirm / change-email) uses `createEndpoint` + `auth.middleware()` like
theme (see `work-with-routes` / `work-with-database`).

## Contract With Client

| Client (`happy-tourist.github.io`) | Server |
|------------------------------------|--------|
| `client.auth.registerWithEmailAndPassword` / `signInWithEmailAndPassword` / `signInAnonymously` / `signInWithProvider('google')` | `/auth/*` + Google provider via `@colyseus/auth` (**no** confirm mail on register) |
| `sendPasswordResetEmail` / cabinet `http.post` send-confirm + change-email | `onForgotPassword` + `POST /api/auth/send-email-confirmation` + `POST /api/auth/email` |
| Token key `colyseus-auth-token` | JWT verified in `onAuth` |
| Pinia `stores/auth`, LoginPage, AccountPage, ForgotPasswordPage, router guards | **N/A on server** — do not port Vue patterns |
| Userdata `email` + `emailVerified` | Persisted on `users`; returned in userdata / JWT payload |
| Register options e.g. `{ name }` | Persist / map via users schema / Auth hooks if customized |

Coordinate userdata shape and register options with the client skill
`client-work-with-auth`. Change both sides together.

## Changing Auth Safely

1. Keep `database: db` on `defineServer` or `/auth/*` disappears.
2. Preserve `MyRoom.onAuth` → `JWT.verify` → return userdata to `onJoin`.
3. New `NOT NULL` user columns → `.default(...)` before deploy.
4. Keep auth / Google / mail / URL env keys in `.env.example` and deploy notes; prod secrets not committed.
5. CORS must continue to allow credentialed cross-origin auth from
   `https://happy-tourist.github.io` in production.
6. Update mocha tests (room JWT + email flows with mock mailer) when auth changes.
7. Sync register/login/anonymous/Google + email verify/forgot + userdata with the client — never only one side.
8. Prefer room `auth` payload for identity; do not trust client `options` over JWT userdata.
9. Keep Google `addProvider` in `src/config/auth.ts`; wrap (do not replace) built-in OAuth callback only for `emailVerified`.
10. Never add `onSendEmailConfirmation` for auto-send; confirm mail only via send-confirm endpoint; cooldown starts only after successful send.
11. Wire callbacks via `getRuntimeAuth()` / `configureAuthEmailFlows()` so tsx vs build auth singletons stay consistent.

## Common Mistakes

| Mistake | Why it hurts |
|---------|----------------|
| Replacing `onOAuthProviderCallback` from scratch | Breaks built-in Google flow; **wrap** previous handler for `emailVerified` only |
| Wiring email hooks on static `@colyseus/auth` import under tsx | Wrong singleton vs HTTP runtime — use `getRuntimeAuth()` |
| Starting confirm cooldown before send succeeds | Blocks retries after SMTP/transient failure |
| Setting `onSendEmailConfirmation` | Auto-mail on register — product forbids this |
| Adding `NOT NULL` user column without `.default` | `/auth/register` / `/auth/login` fail |
| Removing `database` from `defineServer` | Auth HTTP routes gone |
| Skipping or weakening `onAuth` | Unauthenticated room joins |
| Gating `onAuth` / rooms on `emailVerified` | Soft verify — play must stay open |
| Inventing cookie / Redis / captcha / SMS auth | Wrong stack; client is JWT + `client.auth` |
| Porting Pinia / Vue router guards to the server | Server has no Vue; gate is `onAuth` |
| Trusting join `options` for user id over `auth` | Spoofable identity |
| Committing real `JWT_SECRET` / SMTP / prod `.env` | Secret leak |
| Adding Resend / `MAIL_PROVIDER` | Only smtp.bz is in scope |
| Changing only server userdata fields | Client store / LoginPage / cabinet break |

## Change Checklist

When touching auth:

1. Which piece? `app.config` / `config/auth` (OAuth + email hooks) / `lib/mailer` / `db/schema` / secrets / HTTP send-confirm|change-email / `MyRoom.onAuth` / `onJoin` / tests / client contract.
2. `/auth/*` still enabled via `database: db`; Google still registered via `addProvider` side-effect.
3. `onSendEmailConfirmation` still **unset**; confirm send only via `POST /api/auth/send-email-confirmation`.
4. Custom user columns still have safe `.default(...)` where `NOT NULL` (incl. `emailVerified`).
5. `onAuth` still `JWT.verify(token)` and returns userdata; no hard emailVerified gate.
6. `onJoin` still uses `auth` for identity / seats.
7. Env secrets documented in `.env.example` (Google + `SMTP_BZ_*` / `MAIL_FROM` / `AUTH_BACKEND_URL` / `CLIENT_APP_URL`); prod secrets not committed.
8. CORS still allows credentials + client origin in production.
9. Tests still cover JWT connect + mailer-mocked email flows.
10. Contract synced with `../happy-tourist.github.io` (`client-work-with-auth`).
11. Built-in OAuth callback still invoked (wrap only for `emailVerified`); `getRuntimeAuth` + `configureAuthEmailFlows` still used.

## Do / Don't

| Do | Don't |
|----|--------|
| Use `@colyseus/auth` + JWT for all auth | Invent express-session + Redis / cookie challenge login |
| Gate rooms with `static onAuth` → `JWT.verify` | Trust client-only auth or skip room gate |
| Give custom `NOT NULL` columns `.default(...)` | Add required columns that break register/login |
| Keep secrets in env (auth + Google + smtp.bz + URL bases) | Hardcode or commit production secrets |
| Pass verified userdata from `onAuth` into `onJoin` | Prefer join `options` over `auth` for identity |
| Coordinate with client `client.auth` / `colyseus-auth-token` | Import or mimic Vue / Pinia auth on the server |
| Support anonymous + email/password + Google via `addProvider` + wrap for verified | Replace OAuth callback entirely or captcha / SMS unless product asks |
| Send confirm mail only from button HTTP endpoint | Set `onSendEmailConfirmation` or auto-send on change-email |
| Use smtp.bz `sendEmail` for forgot + confirm | Add Resend / second mail provider |
| Cover JWT connect + email flows in mocha | Leave room auth / mail paths untested after changes |
