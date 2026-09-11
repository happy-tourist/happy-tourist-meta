---
name: work-with-routes
description: >-
  Use when adding, changing, or reviewing HTTP routes on the happy-tourist
  Colyseus server: createRouter / createEndpoint in app.config.ts, Express
  hook handlers (/health, /hi), auth /auth/* surface, or Colyseus room listing
  /rooms/:roomName. Keep HTTP thin — game logic belongs in rooms.
---

# Work With Routes

Use this skill when adding or changing **HTTP** handlers in `happy-tourist-server`.

This is a **realtime game server**, not a REST BFF. Prefer WebSocket room messages
for gameplay. HTTP is for health, demo/smoke, auth (provided by Colyseus), and
lobby room listing.

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
| Lobby listing | Colyseus built-in `GET /rooms/:roomName` (client: `/rooms/checkers`) |
| CORS / monitor | Express hook — see `work-with-middleware` / `work-with-config` |
| Gameplay | **Not** HTTP — Room handlers + schema + messages |

## Rules

| Do | Don't |
|----|--------|
| Keep handlers thin (JSON / plain text, no board rules) | Put checkers move validation or board mutation in HTTP |
| Add demo/API-style endpoints via `createEndpoint` in `routes` | Invent a second Express router tree or BFF layer |
| Put ops smoke checks (`/health`, `/hi`) in the `express` hook | Duplicate `/auth/*` or reinvent JWT login as custom Express routes |
| Align lobby listing room name with registered room type | Assume `/rooms/checkers` works while room is still registered as `my_room` |
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
}),
```

- Key (`api_hello`) is an internal id; path/method come from `createEndpoint`.
- Handler returns a value → JSON response. Keep it side-effect light.
- Prefer this for new **public demo / thin JSON** endpoints that are not healthchecks.

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
| GET | `/health` | `express` hook | `{ status, uptime }` — deploy / monitor |
| GET | `/hi` | `express` hook | Plain text smoke |
| * | `/auth/*` | `@colyseus/auth` | Present when `database: db` is set |
| GET | `/rooms/:roomName` | Colyseus | Available-rooms listing (client lobby) |
| GET | `/monitor` | `express` (non-prod) | Colyseus Monitor |
| * | `/` playground | `express` (non-prod) | Dev playground |

### Auth routes

Do **not** hand-roll register/login/anonymous in Express. With `database` on
`defineServer`, `@colyseus/auth` mounts `/auth/*`. Room access still uses JWT in
`Room.onAuth` (`server-work-with-auth`).

### Lobby listing vs room registration

Client lobby expects `GET /rooms/checkers`. Listing works for the **registered
room type name**. Today the room is `my_room` — rename to `checkers` when
implementing the product room (`work-with-rooms`).

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
| JSON object | return value from `createEndpoint` or `res.json(...)` | `/api/hello`, `/health` |
| Plain text | `res.send(string)` | `/hi` |
| Auth | shapes from `@colyseus/auth` | `/auth/*` |
| Errors | Keep simple; no ServiceError / BFF envelope | see `server-work-with-errors` |

## Adding a new HTTP endpoint

1. Ask: does this belong in a **Room message** instead? If gameplay → rooms, stop.
2. Choose surface:
   - Thin JSON API / demo → `createEndpoint` inside `createRouter` in `app.config.ts`.
   - Ops / health / middleware-adjacent → `express(app)` hook.
3. Do **not** add captcha/session Redis guards. Auth is Colyseus JWT + `/auth/*`.
4. Keep the handler short; no Drizzle board writes, no checkers rules.
5. Match existing response style (simple JSON or plain text).
6. If the client will call it, align path/method/body with `../happy-tourist.github.io` (or document that it is server-only smoke).
7. Propose `npm run build` / smoke against `/health` when useful; wait for user «готово».

Do **not** create `src/app/routes/routes.js`-style BFF trees, mappers, or Soap/axios layers.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Implementing `move` over REST | Use Room `onMessage('move', …)` |
| Reimplementing `/auth/register` in Express | Rely on `@colyseus/auth` + `database` |
| Copying cookie `checkSession` / `createError` BFF envelopes | Not this stack — JWT + thin JSON |
| Putting CORS after routes | Keep CORS first in `express` |
| Expecting `/rooms/checkers` while room is `my_room` | Rename registration when aligning client |
| Fat handlers with DB game stats “because HTTP is easy” | Prefer room lifecycle / dedicated thin endpoint only if product asks |

## Checklist for a new or changed route

1. Path lives in `src/app.config.ts` (`routes` and/or `express`) — not a new router package.
2. Handler is thin; no authoritative checkers logic.
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
