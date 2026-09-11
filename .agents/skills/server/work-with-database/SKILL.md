---
name: work-with-database
description: >-
  Use when adding, changing, reviewing, or debugging GameDatabase, Drizzle user
  schema, colyseus_users extensions, DATABASE_URL / game.db paths, or profile
  fields (displayName, rating, gamesPlayed, gamesWon, theme) in
  happy-tourist-server so built-in /auth/register and /auth/login keep working.
---

# Work With Database

Use this skill for the **SQLite user store** in `happy-tourist-server`: `GameDatabase` (`@colyseus/database`), Drizzle schema extension, and how it hooks into `@colyseus/auth`.

**Temp path:** this skill lives under `.agents/skills/server/` in this repo for now; canonical copy may later move to `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root.

Driver: **better-sqlite3**. ORM surface: **drizzle-orm** via Colyseus `tables.sqlite.users`.

## Map Of Pieces

| Piece | Path | Role |
|-------|------|------|
| Connection | `src/db/index.ts` | `GameDatabase`: `connectionString` = `DATABASE_URL` ?? `./game.db`; `schemas: { users }` |
| Schema | `src/db/schema.ts` | Extend built-in `colyseus_users` with profile columns |
| Wire-up | `src/app.config.ts` | `database: db` → enables `/auth/*` + user store |
| Env | `.env.example` / `.env.*` | `DATABASE_URL` (local `./game.db`; prod often `/var/www/happy-tourist-server/game.db`) |
| Deploy | `.github/workflows/deploy.yml` | rsync **excludes** `game.db*` — DB is not shipped by deploy |

## Current Schema Facts

`src/db/schema.ts` — `tables.sqlite.users("colyseus_users", { … })`:

| Field | Column | Notes |
|-------|--------|-------|
| `displayName` | `display_name` | `text` (optional / nullable unless you add constraints) |
| `rating` | `rating` | `integer().notNull().default(1000)` |
| `gamesPlayed` | `games_played` | `integer().notNull().default(0)` |
| `gamesWon` | `games_won` | `integer().notNull().default(0)` |
| `theme` | `theme` | nullable `text` — `light` \| `dark` \| unset (`null`); **no** NOT NULL / no default required |

These are **profile** fields (display name, rating, games played/won, UI theme) for auth users — not room/board state. `theme` is written by thin `POST /api/theme` and read by `GET /api/theme` for registered users only; it also lands in JWT userdata on subsequent login (client restore must not rely on JWT alone after reload).

## Relation To Auth

```text
defineServer({ database: db })
        │
        ▼
  @colyseus/auth HTTP: /auth/register | /auth/login | anonymous
        │
        ▼
  writes/reads colyseus_users (built-in cols + our extensions)
        │
        ▼
  JWT → MyRoom.onAuth → userdata in onJoin
```

- Built-in auth routes fill **only standard Colyseus user columns**.
- Custom columns that are `NOT NULL` **without** `.default(...)` break `/auth/register` and `/auth/login`.
- Nullable preference columns (like `theme`) are fine **without** `.default(...)` — auth insert leaves them unset.
- Do **not** reinvent register/login HTTP; extend `users` in `schema.ts` and keep `schemas: { users }` in `index.ts`.
- Room gate stays JWT in `MyRoom.onAuth`; DB schema is for **persisted** profile data, not realtime board truth.
- Existing prod `game.db` must gain new columns (GameDatabase schema sync / ALTER-compatible path) — deploy does not ship a fresh DB.

## How To Add Columns Safely

1. Edit `src/db/schema.ts` only for new persisted user fields.
2. Prefer `integer(...).notNull().default(...)` or `text(...).default(...)` for any column auth insert paths will touch without supplying a value.
3. **Must** use `.default(...)` on custom columns that would otherwise be NOT NULL — required for built-in `/auth/register` / `/auth/login`.
4. Keep exporting `users` and leave `src/db/index.ts` as `schemas: { users }` unless you add another named schema Colyseus expects.
5. Document `DATABASE_URL` in `.env.example` if path/env behavior changes.
6. Remember prod DB file is **server-local** and excluded from rsync — schema changes apply on the running host’s existing `game.db` (plan migrations / recreate carefully; do not assume deploy copies a fresh DB).

Example pattern (same style as existing fields):

```ts
export const users = tables.sqlite.users("colyseus_users", {
  displayName: text("display_name"),
  rating: integer("rating").notNull().default(1000),
  gamesPlayed: integer("games_played").notNull().default(0),
  gamesWon: integer("games_won").notNull().default(0),
  theme: text("theme"), // nullable light | dark | unset
  // newField: integer("new_field").notNull().default(0),
});
```

## Do

- Extend users via `tables.sqlite.users("colyseus_users", { … })` in `src/db/schema.ts`.
- Give every custom NOT NULL column a `.default(...)`.
- Keep `GameDatabase` wiring in `src/db/index.ts` (`DATABASE_URL` ?? `./game.db`).
- Treat profile stats (rating, games played/won) as DB fields updated from authoritative server code (e.g. room end), not from trusted client payloads alone.
- Preserve prod path awareness: often `/var/www/happy-tourist-server/game.db`.

## Don't

- Add custom NOT NULL columns **without** `.default(...)` — breaks built-in auth.
- Put board / match state in the users table — that belongs in `@colyseus/schema` room state.
- Commit `game.db`, secrets, or production `.env` with real credentials.
- Expect rsync deploy to create or replace `game.db` (it is excluded as `game.db*`).
- Bypass `@colyseus/auth` with a parallel user table unless there is a strong, explicit reason.
- Edit `src/index.ts` for DB — wire `database: db` in `app.config.ts`.

## Checklist

When changing DB-related code:

- [ ] Custom columns have `.default(...)` where auth insert omits them
- [ ] `schemas: { users }` still matches `schema.ts` export
- [ ] `.env.example` still documents `DATABASE_URL`
- [ ] No reliance on deploying `game.db` via rsync
- [ ] Profile vs room-state boundary clear
