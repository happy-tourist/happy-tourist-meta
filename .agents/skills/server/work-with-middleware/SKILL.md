---
name: work-with-middleware
description: >-
  Use when adding, changing, reviewing, or debugging Express middleware in the
  happy-tourist Colyseus server defineServer express(app) hook: CORS (first),
  /health, /hi, non-production monitor()/playground(), Access-Control headers,
  OPTIONS 204, or credentials cross-origin for GitHub Pages. Not room JWT
  onAuth and not BFF session/captcha guards.
---

# Work With Middleware

Use this skill when working with HTTP middleware inside `defineServer({ express })`
in `src/app.config.ts`.

Stack: Colyseus 0.18 (`defineServer` via `@colyseus/tools` / `colyseus`), Express 5
mounted in the `express(app)` hook, TypeScript (`"type": "module"`, NodeNext),
Node `>= 22`.

**Temp skills path:** this skill lives under `.agents/skills/server/` in this
package for now. Canonical skills are intended to live in
`happy-tourist-meta/.agents/skills/server/` once meta is available — prefer that
path when choosing skills if it exists.

Related skills (by name only — load when that area is in scope): `work-with-config`
(env / `ALLOWED_ORIGIN` / NODE_ENV), `work-with-routes` (`createEndpoint` /
`createRouter`), `server-work-with-auth` (JWT / `@colyseus/auth`),
`server-work-with-structure`.

## Out Of Scope

This is **not** a BFF guard layer. Do **not** invent or port:

- `checkSession` / `checkCaptcha` / `checkUser`
- `express-session` + Redis
- CAPTCHA verify middleware
- Auth that `res.status(401|403).send(...)` from Express for room play

Room access uses **JWT in `Room.onAuth`** (`JWT.verify`), not Express middleware.
HTTP auth routes (`/auth/*`) come from `@colyseus/auth` when `database` is set —
do not reimplement them as custom middleware here.

## Core Model

All custom Express middleware for this package lives in the **`express: (app) => { ... }`**
callback in `src/app.config.ts`. Order matters.

```
HTTP request
  → CORS middleware (MUST be first)
  → /health, /hi (and any other app.get/app.use you add)
  → non-prod: /monitor (monitor()), / (playground())
  → Colyseus / @colyseus/auth / routes from defineServer
```

| Piece | Where | Role |
|-------|--------|------|
| CORS | first `app.use` in `express` | Cross-origin + credentials for SPA ↔ server |
| `GET /health` | `express` | Deploy / uptime JSON |
| `GET /hi` | `express` | Plain-text smoke check |
| `monitor()` | `app.use("/monitor", …)` only if not production | Colyseus Monitor UI |
| `playground()` | `app.use("/", …)` only if not production | Colyseus Playground |

`ALLOWED_ORIGIN`:

- production → `"https://happy-tourist.github.io"`
- otherwise → `true` (any origin; resolved per-request from `req.headers.origin` or `"*"`)

## Patterns

### CORS first

```ts
const ALLOWED_ORIGIN =
  process.env.NODE_ENV === "production"
    ? "https://happy-tourist.github.io"
    : true;

express: (app) => {
  app.use((req, res, next) => {
    const origin =
      typeof ALLOWED_ORIGIN === "string"
        ? ALLOWED_ORIGIN
        : (req.headers.origin ?? "*");

    res.header("Access-Control-Allow-Origin", origin);
    res.header("Access-Control-Allow-Credentials", "true");
    res.header("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
    res.header(
      "Access-Control-Allow-Headers",
      "Origin, X-Requested-With, Content-Type, Accept, Authorization"
    );

    if (req.method === "OPTIONS") return res.sendStatus(204);
    next();
  });

  // …health, hi, monitor/playground after CORS
};
```

Rules:

- CORS middleware **must stay first** in the Express hook (GitHub Pages ↔ server,
  credentials).
- Reflect a concrete `Access-Control-Allow-Origin` when using credentials — do not
  pair `Allow-Credentials: true` with a blind `*` in production.
- Keep `Authorization` in `Allow-Headers` (client sends JWT for rooms / auth).
- Preflight: `OPTIONS` → **204** then stop; otherwise `next()`.
- Do not move CORS into a separate Redis/session package; keep it inline (or a
  small local helper) next to this hook unless the project already extracts it.

### Health and smoke routes

```ts
app.get("/health", (_req, res) => {
  res.json({ status: "ok", uptime: process.uptime() });
});

app.get("/hi", (_req, res) => {
  res.send("It's time to kick ass and chew bubblegum!");
});
```

Rules:

- `/health` returns JSON `{ status, uptime }` for deploy/monitor.
- `/hi` is a plain-text smoke check — keep it lightweight.
- Prefer registering these **after** CORS, **before** heavy/dev-only mounts.

### Non-production monitor and playground

```ts
if (process.env.NODE_ENV !== "production") {
  app.use("/monitor", monitor());
  app.use("/", playground());
}
```

Rules:

- Gate with `NODE_ENV !== "production"` — never expose Monitor/Playground in prod.
- Import `monitor` / `playground` from `colyseus` (same as `app.config.ts`).
- Mount Monitor under `/monitor`; Playground at `/`.

## Wiring Changes

1. Edit only the `express(app)` hook in `src/app.config.ts` (or a helper it clearly owns). Prefer not editing `src/index.ts` for middleware.
2. Keep CORS as the **first** `app.use`.
3. Add thin routes with `app.get` / `app.use` here; use `createEndpoint` / `createRouter` under `routes:` for typed Colyseus HTTP endpoints (`work-with-routes`).
4. Do **not** add session/captcha/Redis guards. Room security → `onAuth` + JWT (`server-work-with-auth`).
5. If changing allowed origin or env, align with `work-with-config` / `.env*`.
6. Run `npm test` / `npm run build` from the server package root when relevant; fix failures before claiming done.

## Checklist

- [ ] CORS is the first middleware in `express(app)`.
- [ ] Production origin is `https://happy-tourist.github.io`; credentials enabled.
- [ ] `Allow-Methods` / `Allow-Headers` include what the SPA needs (`Authorization`).
- [ ] `OPTIONS` returns 204 without falling through incorrectly.
- [ ] `/health` and `/hi` still registered after CORS.
- [ ] `monitor` / `playground` only when `NODE_ENV !== "production"`.
- [ ] No Redis / express-session / checkCaptcha-style guards invented.
- [ ] Room auth left on JWT `onAuth`, not Express middleware.

## Common Mistakes

- Registering CORS after `/health` or Monitor — breaks preflight / credentialed SPA calls.
- Using `Access-Control-Allow-Origin: *` with `Allow-Credentials: true` in production.
- Enabling `monitor()` / `playground()` in production.
- Porting cookie `checkSession` / `checkCaptcha` / Redis session middleware into this server.
- Putting room JWT verification in Express middleware instead of `MyRoom.onAuth`.
- Editing `src/index.ts` for CORS/health instead of `app.config.ts` `express` hook.
- Duplicating `@colyseus/auth` HTTP under custom middleware when `database: db` already enables `/auth/*`.
