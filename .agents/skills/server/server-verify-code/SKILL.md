---
name: server-verify-code
description: >-
  Use when the user asks to verify code, check skill compliance, audit a branch
  diff vs master/main/merge-base, audit local diffs, or after implementing a
  change in the happy-tourist Colyseus tourist server. Also when checking
  defineServer wiring, room/schema/message contracts, auth/JWT, CORS, DB user
  defaults, or DRY / KISS / YAGNI balance on changed server files. Reports
  three tiers: Violations, Warnings, Recommendations.
---

# Verify Code

Use this skill to check **all production code changed on the current branch** in
the `happy-tourist-server` package against **all code-related** project skills
under `.agents/skills/server/`, plus the **built-in** checks in this file
(Server conventions, and DRY / KISS / YAGNI with conflict-aware judgment).

Default scope is the **full branch diff**: commits vs merge-base **and**
uncommitted working-tree changes. Do not stop at staged/unstaged/untracked.

This skill is primarily an **orchestrator**: domain rules live in the code
skills listed below — **read those skill files** when present and apply them; do
not restate or invent parallel domain rules here. Built-in sections below fill
gaps when a listed skill file is missing, and own Colyseus server conventions
(defineServer, rooms, auth, CORS, schema/messages) plus DRY / KISS / YAGNI
balance.

**Paths:** skills currently live in **this repo** at `.agents/skills/server/`
(temporary; later move to **happy-tourist-meta**). Runtime code and `src/…`
paths are relative to **this server repo root** (`happy-tourist-server`).
Sibling browser SPA is **`../happy-tourist.github.io`**, not a path under
skills.

**Stack assumptions:** Colyseus 0.18 (`defineServer` / `defineRoom` via
`@colyseus/tools`), `@colyseus/auth` + JWT, `@colyseus/schema`,
`@colyseus/database` + drizzle + better-sqlite3, Express 5 (middleware inside
`defineServer({ express })`), TypeScript (`"type": "module"`, NodeNext),
Node.js `>= 22`, mocha + `@colyseus/testing` under `test/`.

## When To Use

- User asks to verify code / skill compliance / convention check
- After implementing a feature or refactor, before commit
- When reviewing the current branch (or a named path) for convention drift

## Scope: code skills only

Skills live under `.agents/skills/server/<name>/SKILL.md` in this repo
(temporary; later `happy-tourist-meta/.agents/skills/server/`).

### Always include (read and apply each when the file exists)

Verify against **every** skill in this set for the checked code — not a subset
guessed from the path. Path routing below only helps prioritize deeper reading;
it does **not** allow skipping skills from this list.

| Skill | Concern |
|-------|---------|
| `server-work-with-errors` | error handling, empty `catch`, client-facing failure contracts |
| `server-work-with-auth` | `@colyseus/auth`, JWT, `onAuth` |
| `server-work-with-structure` | `src/` layout, `app.config.ts` vs `index.ts` |
| `work-with-config` | `defineServer` wiring, env-driven server config |
| `work-with-routes` | custom HTTP endpoints (`createEndpoint`, `/api/*`, `/health`) |
| `work-with-middleware` | Express middleware order (CORS first, monitor/playground) |
| `work-with-rooms` | room handlers (`onCreate` / `onJoin` / `onLeave` / `onDispose`) |
| `work-with-schema` | `@colyseus/schema` synced state |
| `work-with-messages` | room message handlers (none for seating; add with moves) |
| `work-with-game` | board game «Счастливый турист» rules (later); room `tourist` |
| `work-with-database` | `GameDatabase`, drizzle `users` schema defaults |
| `work-with-env-deploy` | `.env*`, secrets, PM2, GitHub Actions deploy |
| `work-with-loadtest` | `@colyseus/loadtest` scripts under `loadtest/` |

If a new code skill appears under `.agents/skills/server/` (same kind: how to
write app code — rooms, schema, messages, auth, routes, middleware, DB, env),
include it too. Prefer reading one extra skill over missing a rule.

If a listed skill file is **missing**, do not invent a parallel rulebook — apply
**Built-in: Server conventions** for that concern and continue.

### Always exclude

Do **not** use these for server-verify-code (unless the user explicitly asks):

| Skill | Why excluded |
|-------|----------------|
| `server-work-with-test` | test-writing / test conventions (not used to audit prod code) |
| `server-locate-change-points` | planning where to edit |
| `server-verify-code` | this orchestrator |
| `server-align-code` | alignment / docs authorship |
| `openspec` / opsx skills | change workflow / specs |
| `commit` | commit messages |

Skip verifying files that are **only** tests (`test/**`, `**/*.test.ts`,
`**/*.spec.ts`, `**/__tests__/**`) unless the user explicitly asks to include
them. Focus on production/source code under `src/` (and deploy/runtime config
touched by the change: `ecosystem.config.cjs`, `.github/workflows/**`,
`.env.example` — not secret `.env*` contents). Include `loadtest/**` only when
those files are in the change set and `work-with-loadtest` applies.

## What To Check

**Always** inspect the full branch change set. Working tree alone is not enough.

Run git from the **repository root** (this server package is the git root).

1. Resolve merge-base: `git merge-base HEAD <base-ref>`. Prefer `origin/main`
   (this repo), else `origin/master`, `main`, `master`, `develop` — first that
   exists.
2. Committed on the branch: `git diff --name-only <merge-base>...HEAD` and
   `git diff <merge-base>...HEAD`
3. Staged: `git diff --cached --name-only` and `git diff --cached`
4. Unstaged: `git diff --name-only` and `git diff`
5. Untracked: `git ls-files --others --exclude-standard` (read those files fully)

Union of 2–5 is the file set (filter to server production sources unless the user
widened scope). Never skip step 2 because the working tree looks small.

Narrow only when the user **explicitly** asks for working-tree-only / only staged,
or names a **folder or path**.

Skip unrelated noise (lockfiles, coverage, binary assets, `game.db*`, build
output) unless the user asked to include them. Do not commit or demand review of
production secrets in `.env.production`.

## Commands (before done)

Run package scripts from the **server package root** (`happy-tourist-server`).
Fix failures before claiming done. Do not skip test / build after changes unless
the user explicitly asked for a report-only pass with no tooling.

When verification is part of an implementation you own, or when the user asked
to verify-and-fix / “make sure it passes”, run:

| Command | Purpose |
|---------|---------|
| `npm test` | mocha + `@colyseus/testing` (`test/**.test.ts`) |
| `npm run build` | `tsc` → `build/` (`tsconfig.build.json`) |

Prefer `npm test` for room/auth/schema behavior and `npm run build` for
type/compile confidence. Run `npm run loadtest` when loadtest changes are in
scope. For interactive smoke, you may start `npm run dev` when needed; prefer
finite gate commands (test/build). Do not treat passing test/build as a
substitute for skill checks.

Report failed test/build output under **Violations** (tooling) with the command
and a short failure summary. Do not invent eslint/tsc rules beyond the project
config and skill text.

## Workflow

1. Collect the file set: merge-base…HEAD **plus** staged, unstaged, untracked
   (or the path the user named). Exclude test-only files per Scope.
2. **Read all skills from the Always include table** that exist on disk (do not
   rely on memory; do not skip because the path “looks unrelated”).
3. For each source file, apply every code skill whose rules can touch that file.
   When unsure, apply the skill. If missing, use Built-in Server conventions.
4. Apply **Built-in: Server conventions** to all checked production files.
5. Apply **Built-in: DRY / KISS / YAGNI** to all checked production files.
   Resolve principle conflicts with the order in that section **before**
   classifying or fixing — never emit opposing principle fixes for the same hunk.
6. Classify every finding into Violations / Warnings / Recommendations
   (see Severity). Each item: file path, what is wrong, which skill/rule, and
   the expected pattern. Soft Prefer guidance → Warnings or Recommendations,
   not silence.
7. Fix **Violations** when the user asked to verify-and-fix, or when
   verification runs as part of an implementation task you own. Fix **Warnings**
   on verify-and-fix when the preferred pattern clearly fits. Fix
   **Recommendations** only if the user asked to tidy / apply soft order.
   On principle fixes, obey **Built-in: DRY / KISS / YAGNI** conflict order so
   DRY does not fight KISS/YAGNI (and vice versa). Otherwise list findings and
   wait.
8. When implementing / verify-and-fix: **run** `npm test` and `npm run build`
   (and loadtest if relevant) from the server package root; fix failures before
   claiming done.

Do not praise compliant code. Do not turn this into a product/bug review skill
or `server-align-code`. Do not report `server-work-with-test` findings unless
asked.

## Built-in: Server conventions

Grounded in `AGENTS.md` and current `src/` layout. Report as
`(skill: server-verify-code / Server conventions)`. Apply always; sibling skills
win when they exist and conflict on a detail.

### Entry and defineServer wiring

- Process entry: `src/index.ts` → `listen(app)` from `@colyseus/tools`. Prefer
  **not** editing `index.ts` unless self-hosting details require it.
- Configure rooms, database, routes, and Express middleware in
  `src/app.config.ts` via `defineServer({ … })`.
- `database: db` enables `@colyseus/auth` HTTP routes + user store.
- Rooms map should register the product room for the client contract (see
  Room name and client contract).

**Violations when:** gameplay/HTTP/auth wiring is stuffed into `index.ts` while
`app.config.ts` already owns that surface; `defineServer` database/rooms/routes
hooks are bypassed with a parallel ad-hoc server bootstrap without an explicit
request.

### CORS and Express order

- CORS middleware must stay **first** in the Express hook (GitHub Pages ↔ server
  cross-origin + credentials).
- Production: allow origin `https://happy-tourist.github.io` with credentials.
- Development: any origin is allowed.
- Non-production may mount `/monitor` (`monitor()`) and playground
  (`playground()`); do not expose those in production without an explicit
  request.
- Keep `/health` (and smoke checks like `/hi`) suitable for deploy monitors.

**Violations when:** CORS is moved after other middleware that breaks
preflight; production CORS drops credentials or the GitHub Pages origin;
monitor/playground enabled in production without request.

### Room name and client contract

Sibling client (`../happy-tourist.github.io`) assumes:

| Client expectation | Server should provide |
|--------------------|------------------------|
| Room type name `tourist` | Register room as `tourist` (+ `lobby` + `.enableRealtimeListing()`) |
| Tourist board layout on Game | Client-only tile geometry; no layout sync required |
| Synced seats / started | `MyRoomState`: `started` + `seats` Map; move messages later |
| Live lobby (`LobbyRoom`) | `lobby` registered; tourist has realtime listing |

Prefer aligning room name, schema, and messages with the client rather than
changing the client unilaterally.

**Violations when:** new gameplay ships under a room name / state shape / message
payload that breaks the client contract without an explicit coordinated client
change. **Warnings when:** scaffold leftovers (`mySynchronizedProperty`) remain
while tourist product seating is already shipped — remove the scaffold.

### Auth and rooms

- Room gate: `onAuth` verifies JWT (`JWT.verify`) and returns userdata to
  `onJoin`.
- Auth HTTP (`/auth/*`) comes from `@colyseus/auth` when `database` is set —
  do not invent a parallel register/login stack.
- Secrets: `AUTH_SALT`, `JWT_SECRET`, `SESSION_SECRET` (and `DATABASE_URL`,
  `NODE_ENV`, `PORT`) per `.env.example` — do not hardcode production secrets
  in source.

**Violations when:** rooms accept clients without JWT verification; auth secrets
committed; identity for game seats taken from untrusted client payloads instead
of verified auth userdata.

### Authoritative gameplay

- Once rules exist, board-game truth for «Счастливый турист» lives on the **server**.
- Today seating is authoritative (`seats`/`started`); there are **no** Game move messages; client mirrors seats and renders pieces.
- Do not trust client-supplied layout/state as source of truth.

**Violations when:** server applies client board snapshots as truth; or invents
legacy draughts `move`/`0`–`4` encoding as product without a coordinated change.

### Database / users schema

- `src/db/index.ts` — `GameDatabase` with `schemas: { users }`.
- `src/db/schema.ts` — extend `colyseus_users` with profile fields
  (`displayName`, `rating` default 1000, `gamesPlayed` / `gamesWon` defaults
  0). Custom columns need `.default(...)` so built-in `/auth/register` /
  `/auth/login` do not fail on NOT NULL.

**Violations when:** new NOT NULL user columns lack defaults and break register /
login. **Warnings when:** profile stats updated on only one of readers/writers
without a clear path.

### Layering (hard direction)

Typical paths:

```text
HTTP auth → @colyseus/auth + GameDatabase / users schema
Matchmaking → Colyseus room create/join + GET /rooms/:roomName
Gameplay (later) → Room handler + schema state ← client onStateChange / game messages
Express → CORS → /health (+ optional monitor/playground)
```

| Area | Location | Owns |
|------|----------|------|
| Entry | `src/index.ts` | `listen(app)` only |
| Server definition | `src/app.config.ts` | DB, rooms, routes, Express |
| DB | `src/db/` | GameDatabase + users schema |
| Rooms | `src/rooms/` | lifecycle, `onAuth`, messages, rules |
| Schema | `src/rooms/schema/` | synced state definitions |

**Do not:** invert layers (e.g. schema importing Express routes); put tourist
rules only on the client; scatter a second HTTP auth implementation beside
`@colyseus/auth`.

### TypeScript / files / soft style

- App code is **TypeScript** ESM (NodeNext); follow existing import style.
- Prefer extending the product room/schema over leaving conflicting scaffolds
  as the registered default.
- Soft tidy (Recommendations): group imports; keep room handlers readable;
  avoid drive-by reformat of untouched hunks.

### Agent commands

- Run `npm test` / `npm run build` / `npm run loadtest` from the server package
  root; fix failures before claiming done. Do not skip verification or assume
  pass without running.

### Severity cues for built-in conventions

| Tier | Examples |
|------|----------|
| **Violations** | Room name/state/move contract breaks client; missing JWT `onAuth`; trusting client board; CORS not first / wrong prod origin; NOT NULL user columns without defaults; gameplay wiring forced into `index.ts`; empty `catch` on auth/game paths; secrets in source |
| **Warnings** | Prefer existing schema fields but a one-off parallel property was added; env URL/path hardcoded when `.env` already covers it; HTTP listing used as primary path while live LobbyRoom is the product UI |
| **Recommendations** | Import grouping tidy; minor formatting; soft consistency with nearby room/schema analogues; mild DRY/KISS polish |

## Built-in: DRY / KISS / YAGNI

Report as `(skill: server-verify-code / DRY-KISS-YAGNI)`. Apply to all checked
production files. Principles are **judgment lenses**, not absolute mandates.

**Core:** Prefer the simplest correct code that meets the real requirement.
Project skills and Server conventions **win** over DRY, KISS, and YAGNI. When
principles pull opposite ways, choose **one** outcome using the conflict order
below — never “satisfy DRY” by violating KISS/YAGNI or a domain skill, and never
“simplify” by deleting a required shared contract.

### Verify lens

| Principle | Flag when | Do not flag when |
|-----------|-----------|------------------|
| **DRY** | Same *knowledge/rule* is duplicated in the change set and will drift if only one side changes | Similar-looking code with different domain meaning; intentional parallel handlers a skill requires; trivial short copies that stay clearer separate |
| **KISS** | New indirection, factory, or clever layer that obscures the change without payoff | Required layering from skills (`defineServer`, rooms, schema, auth, Express order) |
| **YAGNI** | Abstraction, extension point, or helper built for hypothetical future call sites (0–1 real uses of that shape) | Small helper with 2+ real call sites of the *same* shape already in the change |

### Conflict resolution (required before classify / fix)

Apply **in order**:

1. **Domain skills / Server conventions / contracts** — do not dedupe or simplify away a required pattern (e.g. keep authoritative rules on the server; do not merge distinct room/message concerns into one grab-bag).
2. **YAGNI** — remove or do not introduce unused / future-only abstractions.
3. **KISS** — prefer direct code over shared machinery when duplication is small or meanings differ.
4. **DRY** — extract only when identical knowledge would otherwise drift; the extraction must stay simple.

**Anti-conflict rule:** Do not report both “extract for DRY” and “inline for KISS/YAGNI” on the same hunk. Pick one using the order above. On verify-and-fix, apply that single outcome. Prefer **one finding per hunk** that names the winning principle (mention secondary principles only as supporting reason in the same bullet).

### Severity cues

| Tier | When |
|------|------|
| **Violations** | Rare for pure principles. Only if a new premature abstraction **breaks** a required skill/contract, or critical contract knowledge is duplicated **and already diverges** in the same change set |
| **Warnings** | Identical-knowledge duplication likely to drift; unused/future-only abstraction; complexity that obscures a preferred skill pattern |
| **Recommendations** | Mild duplication or slightly overcomplicated local code where a simpler safe shape is obvious |

### Red flags — STOP and re-resolve

- “Merge for DRY” when a skill or contract says keep the split
- Shared util with one call site “for later” (YAGNI)
- Inlining a module that multiple real callers already need (false KISS)
- Opposing principle findings on the same hunk without conflict resolution

## Path hints (optional prioritization only)

Use only to decide **where to look harder**, never to drop a skill from Always include.

| Change / path signals | Look harder at |
|----------------------|----------------|
| `src/app.config.ts`, `src/index.ts` | Server conventions (`defineServer`, CORS order, room registration) |
| `src/rooms/**` | `work-with-rooms`, `work-with-messages`, `work-with-game`, auth `onAuth` |
| `src/rooms/schema/**` | `work-with-schema`, client sync contract (`started`/`seats`) |
| `src/db/**` | `work-with-database`, users defaults |
| auth / JWT / `@colyseus/auth` | `server-work-with-auth` |
| custom `/api/**`, `createEndpoint` | `work-with-routes` |
| Express CORS / monitor / playground | `work-with-middleware` |
| `.env*`, `ecosystem.config.cjs`, `.github/workflows/**` | `work-with-env-deploy` |
| `loadtest/**` | `work-with-loadtest` |
| error handling / empty `catch` | `server-work-with-errors` |
| `src/` layout ownership | `server-work-with-structure`, `work-with-config` |

## High-signal checks (examples, not a full rulebook)

Reminders to **open the skill** (or Built-in) — skill text wins.

- **defineServer**: rooms/DB/routes/Express in `app.config.ts`; prefer not editing `index.ts`.
- **CORS**: first middleware; prod `https://happy-tourist.github.io` + credentials.
- **Room contract**: name `tourist`; live `lobby`; static client board; synced rules/messages later.
- **Auth**: JWT in `onAuth`; `@colyseus/auth` + DB user store.
- **Rules**: authoritative on server when they land; never trust client board.
- **DB**: user column `.default(...)` for register/login.
- **Tooling**: run `npm test` / `npm run build` from the server package root.
- **DRY/KISS/YAGNI**: conflict order — skills → YAGNI → KISS → DRY; one fix per hunk.

## Severity

Map each skill finding by how the source skill phrases the rule. Skill text wins;
when wording mixes levels, pick the strongest that still applies.

| Tier | When | Skill wording cues (examples) |
|------|------|-------------------------------|
| **Violations** | Hard break of a required pattern / contract | Do / Don't, Must, Never, Always, Hard Rules, missing `onAuth`, client board trust, CORS order, breaking `tourist` contract |
| **Warnings** | Preferred pattern clearly fits; allowed exception does **not** apply | Prefer, Should, “use X instead of Y” when X fits |
| **Recommendations** | Soft tidy / style / optional polish | usually, Soft, import grouping, optional analogue consistency, mild DRY/KISS |

Do not invent findings outside the code skills and the built-in sections above.
Do not inflate Prefer into Violations.

## Output

Report language: **Russian** for prose; keep paths, symbols, and code as-is.
Section headings stay **Violations** / **Warnings** / **Recommendations** (and
`## Verify Code`) as in the template below.

Always include all three finding sections. If a section has no items, put a
single line `- none` (do not mix items and `none` in the same section).

Format:

```text
## Verify Code

Scope: branch vs <merge-base> (<base-ref>) + staged + unstaged + untracked [+ path if used]
Skills: all code skills (excl. test/locate/align/openspec/commit) + Server conventions + DRY/KISS/YAGNI
Tests/build: <commands run and pass/fail summary, or skipped with reason>

### Violations
- `path` — <что не так> (skill: `<name>`)
  Expected: <краткий правильный паттерн или секция skill>

### Warnings
- `path` — <что не так> (skill: `<name>`)
  Expected: <краткий правильный паттерн или секция skill>

### Recommendations
- none
```

Empty-section example: only `- none` under that heading. Item shape is the same
in every tier. Do not collapse Warnings/Recommendations into Violations or omit
soft Prefer findings.

## Red Flags — collect the branch first

These mean STOP and re-collect files vs merge-base before reporting:

- Ran only `git diff` / `--cached` / untracked
- “User didn’t ask for the whole branch”
- “Working tree has the real work”
- “I’ll check working tree first, branch later”
- Report Scope without `branch vs <merge-base>`

## Related Skills

All skills in the Always include table. For tests use `server-work-with-test`
when present (and when the user asks to include tests). For requirements /
analogue alignment use `server-align-code`. Server conventions and DRY / KISS /
YAGNI balance are owned by this skill when sibling files are absent.
