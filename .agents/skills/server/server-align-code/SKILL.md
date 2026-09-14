---
name: server-align-code
description: >-
  Use when aligning branch and working-tree changes against the active OpenSpec
  change (primary), optional pasted clarifications, repository analogues, test
  readiness, preservation of previous behavior, Colyseus room registration (lobby
  + tourist + enableRealtimeListing), @colyseus/schema board/currentTurn/status/players,
  JWT onAuth, room message move {from,to}, Express CORS/health endpoints,
  handler/lifecycle feedback loops (message/HTTP/schema/timer storms), first-sync
  races (seat assign after full-state encode), and GameDatabase/users schema
  consumed by sibling happy-tourist.github.io.
---

# Align Code

Perform a read-only code alignment audit for the Colyseus multiplayer tourist backend (`happy-tourist-server`) and report findings in Russian across three tiers: hard gaps, warnings, and recommendations.

Stack context: Colyseus 0.18 (`defineServer` / `defineRoom` via `@colyseus/tools`), `@colyseus/auth` + JWT, `@colyseus/database` + drizzle-orm + better-sqlite3, `@colyseus/schema`, Express 5, TypeScript ESM (`"type": "module"`, NodeNext), Node `>= 22`. Entry: `src/index.ts` → `listen(app)`; configure rooms/HTTP/DB in `src/app.config.ts`. Contract consumers: sibling SPA `../happy-tourist.github.io` (room type `tourist`, live `lobby` LobbyRoom, Game board + turn/`sendMove`, Colyseus Auth HTTP).

**Paths:** this skill currently lives in **this server repo** at `.agents/skills/server/` (temporary). Canonical skills/OpenSpec will move to **happy-tourist-meta** when present (`project-map.md` key `happy-tourist-meta`). Runtime `src/…` paths are relative to **this repository root**. Sibling Vue/Quasar client is **`../happy-tourist.github.io`**. Outside align-only mode, the agent runs `npm test` / `npm run build` / `npm run dev` from this repo root when verifying; fix failures before claiming done.

## Four Required Axes

Always complete all four:

1. **Requirements fit** — code vs **активный OpenSpec change** (primary); paste/диалог только как явные уточнения.
2. **Codebase fit** — changed behavior vs strong repository analogues.
3. **Test readiness** — deterministic defects in reachable states before deriving tests.
4. **Behavior preservation** — unjustified regressions vs base in refactors, shared helpers, and neighboring edits.

The primary subject is the current working tree and branch diff: staged, unstaged, relevant untracked files, and commits vs base. The **active OpenSpec change** is the primary requirements source; paste/dialog are secondary clarifications only.

## Hard Boundary

Do not edit, create, delete, format, or commit files. Do not run tests for preservation analysis. Deliver the complete result in chat.

## Input And Store

**Primary:** resolve and load one **active OpenSpec change**. Accept an optional change name from the user; otherwise auto-select the single active change. Paste/dialog wording may refine AC only when it does not contradict the change (or the user explicitly overrides).

Ask when the change cannot be resolved (0 or >1 active without a clear pick, and no name given). Do not treat paste alone as a substitute for an active change unless the user confirms `OpenSpec: none`.

OpenSpec lives under **happy-tourist-meta**. Resolve meta via `project-map.md` key `happy-tourist-meta` (from server: sibling `../happy-tourist-meta`). If OpenSpec/meta is unavailable and the user did not confirm fallback — ask; do not invent specs.

### Resolve OpenSpec change

Resolve **one** change name before Axes A–D (including edge-case / test-readiness checks):

1. Use the name the user passed (argument, slash-command, explicit mention).
2. Else infer from conversation context if unambiguous.
3. Else from meta root: `openspec list --json` — auto-select if exactly **one** active change.
4. Else if several active changes — ask the user to pick one.
5. Else if none — ask for a change name or explicit `OpenSpec: none` (then paste/dialog/repo only).

Announce: `Using OpenSpec change: <name>` (or `OpenSpec: none`). Override: user passes another name.

That change is **the primary requirement evidence** for atomic checklists, Axis A, Axis C edge cases / scenarios, and docs↔code contradictions — not an optional afterthought.

## Requirements Sources

Load **in this order**:

1. **Active OpenSpec change** (required when resolved): `proposal`, delta `specs/**/spec.md`, `design`, `tasks` — scenarios (SC-*), acceptance wording, design decisions, task scope.
2. Optional paste / dialog clarifications that explicitly update expected behavior without inventing a parallel spec.
3. Repository / `AGENTS.md` / strong analogues — conventions only, never as replacements for change evidence.

Treat explicit prohibitions and exceptions as atomic requirements: "не возвращать", "не добавлять", "не принимать", "только для…", "кроме…", "без JWT", "stub".

Auth (register / login / anonymous) and room JWT gate (`onAuth` → `JWT.verify`) are first-class. Authoritative russian-tourist rules and board truth live on the server; do not treat client local highlights as fulfilment of server AC.

A tester's guess/question is only a lead. Treat dialog wording as clarification only when it explicitly states or updates expected behavior relative to the active change.

## Atomic Requirement Checklist

### Schema / synced state fields

For every stated `@colyseus/schema` field, independently verify:

- present vs intentionally omitted on the synced state;
- source/path and transformation (room lifecycle → schema mutation → client `onStateChange`);
- requiredness for client contract;
- type and format (`number` cell `0|1|2|3|4`, color string/enum, status string, Map/collection of players);
- allowed range, sign, precision, and length;
- default / empty / initial board setup;
- who may mutate (server only — never trust client board);
- serialization / `@type` annotations match consumer expectations.

A synced field is not covered until all explicit properties are covered. Known client contract today: room `tourist` + lobby listing + `started`/`seats`/`currentTurnSessionId` + `move` `{ side, row, col }`. Scaffold `mySynchronizedProperty` must not remain once seating ships.

### Room messages

For every room message / handler, independently verify:

1. triggering client action (`room.send`) and when the room is ready (`status === 'playing'`, seat assigned);
2. exact message type string (product: `'move'`);
3. every payload field, nesting, requiredness, value, and source (`{ from, to }` with `{ row, col }` or documented equivalent);
4. handler registration (`this.onMessage(...)`) and argument order;
5. auth/seat/turn guards before applying the move;
6. authoritative rule validation (legal move, captures, promotion);
7. schema mutations after accept (board, turn, status, players);
8. reject / no-op / error feedback when illegal (what client observes);
9. disconnect / rejoin / `onLeave` interaction with in-progress games;
10. `maxClients` and seat color assignment when AC demands 1v1.

### Auth

For every auth-related requirement, independently verify:

1. HTTP `/auth/*` surface from `@colyseus/auth` when `database: db` is set;
2. secrets present in env contract: `AUTH_SALT`, `JWT_SECRET`, `SESSION_SECRET` (plus `DATABASE_URL`, `NODE_ENV`, `PORT`);
3. room gate: `MyRoom.onAuth` (or product room) calls `JWT.verify` and returns userdata to `onJoin`;
4. userdata fields used for seats/profile (`displayName`, rating, etc.) vs `src/db/schema.ts` defaults;
5. custom user columns have `.default(...)` so built-in register/login do not fail on NOT NULL;
6. anonymous vs email/password paths when AC distinguishes them;
7. missing/invalid token → join rejected (not silent open room).

### HTTP endpoints

For every HTTP endpoint / listing, independently verify:

1. triggering client action (lobby refresh, health probe, demo API);
2. HTTP method and exact path (`GET /health`, `GET /hi`, `GET /api/hello`, Colyseus `GET /rooms/:roomName`, `/auth/*`);
3. middleware order — CORS **must be first** in the Express hook;
4. CORS policy: production `https://happy-tourist.github.io` + credentials; development any origin;
5. every query/path parameter (room name for listing must match registered room — client expects `tourist`);
6. success body shape (`{ status, uptime }` for health, JSON for `/api/hello`);
7. non-prod only: `/monitor`, playground — must not leak in production;
8. room registration key in `defineServer({ rooms })` aligns with listing path the client calls.

Trace the concrete chain:

```text
HTTP /auth/* (register|login|anonymous) → JWT
  → Room.onAuth (JWT.verify) → onJoin / seat / color
  → schema state sync (board|currentTurn|status|players)
  → client onStateChange
  ↔ client room.send('move', { from, to }) → onMessage → rules → schema mutate
```

Example shape (auth join):

```ts
// client: auth token → joinOrCreate('tourist')
// server: MyRoom.onAuth → JWT.verify(token) → userdata
//         onJoin → assign color → state.players[sessionId].color
```

Example shape (move):

```ts
// client: room.send('move', { from, to })
// server: onMessage('move', …) → validate turn/rules → mutate board / currentTurn / status
```

Example shape (lobby):

```ts
// client: joinOrCreate('lobby', { filter: { name: 'tourist' } }) + rooms / + / -
// server: lobby: LobbyRoom; tourist: MyRoom.enableRealtimeListing()
```

Method presence or "looks compatible" alone is insufficient. Track each contract fact separately so one correct layer cannot hide another mismatch.

The client↔server contract is Colyseus Auth + room type `tourist` + live `lobby` + Game board + turn/`move` (`currentTurnSessionId`, `sendMove`). HTTP `/rooms/tourist` is optional fallback. When CR/docs/client and server disagree, report `code-only` / contradiction with both sides named (`src/rooms/*` / `src/rooms/schema/*` vs `../happy-tourist.github.io`).

Prefer aligning room name, schema, and messages with the client rather than changing the client unilaterally — unless AC explicitly says otherwise.

Use `AGENTS.md` and strong in-repo analogues for conventions; never as replacements for requirement evidence.

## Load Branch Work

Inspect read-only (from this server repo root):

```bash
git status --short
git diff
git diff --cached
git branch -vv
```

When the branch has commits beyond base, determine merge-base and inspect:

```bash
git log --oneline <base>..HEAD
git diff --stat <base>...HEAD
git diff <base>...HEAD
```

Read relevant untracked files and full current TS modules when surrounding behavior matters. Prefer `src/app.config.ts`, `src/rooms/`, `src/rooms/schema/`, `src/db/`, `test/`, deploy (`ecosystem.config.cjs`, `.github/workflows/`), and env templates. Do not ignore unstaged work.

## Load Planning Scope

After resolving the change name (see **Resolve OpenSpec change**), from meta:

```bash
openspec status --change "<name>" --json
```

Read concrete artifact paths from the result (`proposal`, `specs`, `design`, `tasks`). Use them as **primary** requirements for **all four axes**, including Axis C edge cases and scenario IDs from delta specs. Report clear docs↔code contradictions. If no change resolved — only after user confirms `OpenSpec: none`, audit from paste/dialog and repository evidence; otherwise stop and ask. Do not invent specs.

## Axis A — Requirements

Judge implementation first, then planning artifacts.

Report hard gaps only as:

- `missing` — explicitly required and absent;
- `docs-only` — promised by complete artifacts but absent from implemented feature;
- `code-only` — clear docs/code contradiction;
- `extra` — clearly forbidden or unjustified behavior/data/outbound side effect.

Ambiguous CR wording, unresolved Open Questions, or unproven leans → **Warnings**, not hard omissions.

Wrong schema field source/constraint, wrong room name registration, missing JWT `onAuth`, broken CORS order/origin, or trusting client board as authoritative are explicit omissions.

Score **Постановка: N/10** only from hard omissions (`missing` / `docs-only` / `code-only` / `extra`), not from Warnings or Recommendations.

Known scaffold fact: room name is `tourist` (+ live `lobby`); product sync includes seats + turn + `move`. Do not treat scaffold alone as fulfilment of AC that demands product rules — and do not invent draughts `move` / cells `0`–`4` as the current client contract.

## Axis B — Codebase

Find strong untouched analogues for the same domain/flow. Prefer same layer:

| Concern | Look in |
|---------|---------|
| Server definition | `src/app.config.ts` (`defineServer`: database, rooms, routes, express) |
| Room lifecycle | `src/rooms/MyRoom.ts` (`onCreate` / `onAuth` / `onJoin` / `onLeave` / `onDispose` / `onMessage`) |
| Synced state | `src/rooms/schema/MyRoomState.ts` |
| Database / users | `src/db/index.ts`, `src/db/schema.ts` |
| Entry / listen | `src/index.ts` (prefer not changing unless self-hosting) |
| Tests | `test/**/*.test.ts` (mocha + `@colyseus/testing`, JWT connect) |
| Loadtest | `loadtest/example.ts` |
| Env | `.env.example`, `.env.development`, `.env.production` (no prod secrets in git) |
| Deploy | `ecosystem.config.cjs`, `.github/workflows/deploy.yml` |
| Sibling client | `../happy-tourist.github.io` (stores / room protocol / live LobbyRoom) |

Compare JWT gate, CORS-first middleware, health body, room `maxClients`/seats, schema `@type` fields, message handler registration, user column defaults, and test room name / auth contract only when the analogue proves the behavior.

Prefer authoritative rules inside the Room handler + schema mutations — flag scattered trust of client board or duplicated rule sources as codebase-implied when analogues keep that boundary.

Report `gap` only with:

- concrete analogue path;
- changed file lacking the behavior;
- why parity is required rather than optional polish.

Label analogue-driven findings as codebase-implied. Score **Код проекта: N/10** only from hard `gap` items (not Warnings / Recommendations).

## Axis C — Test Readiness

Use evidence in this order:

1. explicit requirements — including the **resolved OpenSpec change** (delta specs scenarios/SC-*, design constraints, task acceptance) when present; do not skip change artifacts and invent generic edge cases instead;
2. room/schema/message contracts, JWT `onAuth`, HTTP path/body shapes, db column defaults;
3. strong analogues;
4. deterministic runtime semantics (join without token, full room, illegal move, wrong turn, disconnect).

For reachable behavior, examine permitted empty/null/zero/false inputs, constrained board indices, missing/invalid JWT, second join when `maxClients` reached, turn mismatch, illegal capture/move, promotion edge cases, `onLeave` mid-game, rejoin, CORS preflight `OPTIONS`, production vs development middleware branches, and register/login failure when user columns lack defaults — **and** every edge/negative path named or implied by the resolved change's specs/design (e.g. lobby snapshot/`+`/`-`, `enableRealtimeListing`, dispose removes listing).

Runtime facts that often create defects:

- room missing `lobby` / `.enableRealtimeListing()` while client uses LobbyRoom → empty live list;
- tests still create a non-registered room name → false green/red;
- scaffold state missing `board` / `currentTurn` / `status` / `players` → client sync breaks;
- `onMessage('move')` absent or payload shape drift vs `{ from, to }`;
- `onAuth` missing / not verifying JWT → open or always-rejected rooms;
- CORS not first or prod origin wrong → browser credentialed requests fail;
- custom user columns without `.default(...)` → `/auth/register` / `/auth/login` NOT NULL errors;

### Handler / lifecycle feedback loops (обязательно)

Когда diff/ветка трогает `onMessage`, room lifecycle (`onJoin` / `onLeave` / `onDispose`), schema mutations, `createEndpoint` / Express handlers, timers/`setInterval`, или код, который из handler снова шлёт message / HTTP / broadcast — **отдельно** проверь, нет ли закальцованности (шторм сообщений, рекурсивных HTTP, бесконечных timer ticks). Ожидание «один inbound event → конечное малое число outbound» — дефолт, пока AC явно не требует push/polling.

Для каждой такой цепочки независимо проверь:

1. **Триггер** — inbound message, HTTP request, schema patch, timer, join/leave.
2. **Outbound side effect** — `clients.send` / `broadcast`, другой `onMessage` path, HTTP call-out, DB write that re-enters the same handler, `setInterval`/`setTimeout` без clear.
3. **Повторный вход** — может ли side effect снова попасть в тот же handler/route без нового внешнего события (self-send, webhook echo, middleware, вызывающий тот же endpoint).
4. **Идемпотентность / guard** — dedupe key, generation, «already applied», clearInterval on dispose; отсутствие guard при доказанном re-entry → defect.
5. **Room dispose / leave** — timers and subscriptions cleared in `onLeave`/`onDispose`; иначе накопление ticks после пустой комнаты.
6. **Кратность** — на один client game message / один preference POST ожидается конечный ответ/state patch, не каскад N сообщений без новых inputs.

Типичные петли для флага:

```text
onMessage / HTTP → mutate / broadcast / HTTP
  → same handler again → …   (or uncleared setInterval → storm)
```

- handler шлёт message того же type, который снова обрабатывает этот room;
- HTTP handler внутри вызывает тот же path / middleware loop;
- schema/`onChange`-подобный hook пишет поле, снова триггерящий тот же hook;
- `setInterval` в `onCreate` без clear в `onDispose` → tick storm после leave;
- retry без backoff/max на auth/DB failure внутри request path.

Report as hard `[defect]` when a reachable path deterministically storms messages/HTTP/DB writes or spins a timer/handler loop. If the chain looks risky but proof is incomplete → **Warning** with the suspected cycle edges.

### Async races — await gap before first sync / ack (обязательно)

Когда diff трогает `onJoin` seating/schema writes, `JOIN_ROOM` / full-state send path, или HTTP handler который отвечает до того, как side-effect завершён — проверь, что **первый** client-visible sync не зависит от гонки с необязательным await на **client**. Server обычно пишет seats в `onJoin` до `JOIN_ROOM`; дефект чаще на sibling client (`await` до `onStateChange`). Если server откладывает seat assign после первого encode full state без последующего patch — hard `[defect]` (client получит пустые seats).

Report hard `defect` only when a reachable state deterministically causes wrong HTTP/WS payload, runtime failure, invalid schema sync, stuck room state, unsafe side effect (including message/HTTP/timer storms **or first-sync races**), or contract violation.

Each hard defect includes condition, current behavior, expected invariant/evidence, file/symbol, and minimum scenario. Verdict:

- `READY` — no proven deterministic hard defect found;
- `BLOCKED` — at least one hard defect must be resolved or explicitly characterized before tests.

Unresolved Open Questions / ambiguous error-branch splits that do not yet prove wrong API/room behavior go to **Warnings**, not `BLOCKED`, unless a reachable wrong status/body/state is already deterministic.

Note existing tests under `test/` (mocha + `@colyseus/testing`). Cite missing coverage as **Recommendations** unless the missing test would be the only way to prove a hard defect already evidenced in code. Do not run `npm test` / `npm run build` here for preservation analysis; when verifying outside align-only mode, run the command from this repo root and fix failures before claiming done.

## Axis D — Behavior Preservation

Classify every changed hunk:

- `task` — directly justified by an atomic requirement;
- `incidental` — refactor/rename/cleanup/shared-helper change;
- `neighbor` — unrelated changed behavior in a task file.

Compare before vs after using diff/base, contracts, and callers. Report `regression` only for proven prior behavior changed without task justification: room registration name, schema public fields, message types/payloads, JWT `onAuth`, CORS/HTTP surface, user schema defaults, shared-helper results, or test auth/room contract.

Do not run tests. Do not double-count the same issue as both defect and regression unless the evidence and impact differ.

### Changed rooms, schema, and message contract

Refactors of room/schema surfaces often break `../happy-tourist.github.io` quietly. When a changed file touches `src/app.config.ts` rooms map, `src/rooms/*`, `src/rooms/schema/*`, or message handlers — audit the realtime contract separately from internal helpers.

For every removed, renamed, or reshaped room/field/message, independently verify:

1. **Old public surface** — room type name, schema fields, message type/payload, success/reject behavior previously observed by clients/tests.
2. **New resolution path** — which room class, schema type, or handler now owns the value.
3. **Caller migration** — grep sibling client (`stores/game`, boot Colyseus, pages) and `test/` for old room names / field names / message types; untouched client call sites are strong regression evidence.
4. **Effective sync** — trace what `@colyseus/schema` actually patches to clients; a renamed field with old client mapping is a break.
5. **Indirect paths** — shared rule helpers, seat assignment used by join and future action paths.

Typical regression patterns to flag:

- room key or listing registration (`lobby` / `tourist` / `.enableRealtimeListing()`) changed without client/test update;
- inventing or requiring draughts `board` / `currentTurn` / `move` / cells `0`–`4` as current product without AC;
- `onAuth` removed or no longer calls `JWT.verify`;
- seating / `maxClients` changed without AC.

Report as `regression` or `defect` when old call sites deterministically stop receiving the intended state, accept/reject moves incorrectly, or fail join/list. Mention approximate client/test hit count when grep proves it.

### Auth, HTTP, DB, and deploy wiring

JWT, Express hook, and user store are easy to break in "cleanup" hunks. Treat them as public cross-layer contracts.

When a changed hunk touches `onAuth`, `src/db/schema.ts`, CORS middleware, `/health` / `/api/*`, env secret names, or deploy/PM2 config, independently verify:

1. **Auth invariants** — room join still requires verified JWT; userdata still reaches `onJoin`.
2. **HTTP side effects** — CORS still first; prod origin + credentials preserved; monitor/playground still non-prod only.
3. **DB defaults** — custom columns still have defaults compatible with built-in `/auth/register` / `/auth/login`.
4. **Listing alignment** — `GET /rooms/<name>` still matches registered room key.
5. **Tests vs runtime** — a schema only asserted in isolation does not prove room wiring; if `app.config` stops registering the room, report even if unit-like stubs pass.
6. **Deploy** — rsync excludes, `pm2 reload`, and env-on-server-only assumptions not silently broken when AC touches ops.
7. **No handler storms** — thin preference/auth HTTP and room messages stay one-shot per request/event; no self-reentry or uncleared timers (see **Handler / lifecycle feedback loops**).

Typical regression patterns to flag:

- unused-variable cleanup deletes JWT verify or seat assignment;
- CORS moved after other middleware or prod origin dropped;
- user column default removed → register breaks;
- health body shape changed while probes expect `{ status, uptime }`;
- test still boots old room name after registration rename;
- new `/api/*` or `onMessage` path that re-invokes itself or leaves a room timer running after dispose.

Report as `regression` or `defect` when consumers deterministically lose auth, CORS, listing, or schema sync after the change, or when a reachable path storms messages/HTTP. Name the field/room/message/endpoint, before/after call sites, and consumers (client and/or `test/`).

## Severity

Every finding goes into exactly one tier. Do not silence softer items; do not inflate them into hard gaps.

| Tier | When | Align cues |
|------|------|------------|
| **Hard gaps** | Explicit requirement / forbidden behavior / required analogue parity / deterministic wrong state / proven unjustified regression | `missing`, `docs-only`, `code-only`, `extra`, required `gap`, `defect`, `regression`; scores and `BLOCKED` use **only** this tier |
| **Warnings** | Strong lean without full proof; ambiguous AC with a clear risk; desirable analogue parity not proven mandatory; Open Question that can change expected behavior; likely server↔client mismatch pending API/аналитика | `[warning]`; does **not** lower scores or force `BLOCKED` alone |
| **Recommendations** | Optional polish vs analogue; docs/OpenSpec sync; unchecked planning tasks; test-coverage ideas that are not defects; soft parity with client or neighbouring room | `[recommendation]`; informational only |

Examples:

- AC requires a specific game message/state shape and the handler/fields are absent → hard `[missing]`.
- AC unclear whether disconnect forfeits or allows rejoin; code picks one and no wrong state is proven → **Warning** until discriminant is settled.
- Analogue room always verifies JWT in `onAuth` and CR says «как MyRoom» → hard `[gap]` if that file is the cited evidence; if CR is silent → **Recommendation**.
- OpenSpec `tasks.md` still unchecked while code exists → **Recommendation** (docs sync), not hard requirements gap.
- Ambiguous CR wording that was misread once already → **Warning** with both readings, ask for confirmation rather than hard `missing`.
- AC demands live LobbyRoom listing and server omits `lobby` / `.enableRealtimeListing()` → hard `[missing]` / `[code-only]` vs client contract as evidenced.
- Scaffold-only state while AC demands product sync fields → hard `[missing]` / `[docs-only]`; do not treat scaffold property as fulfilment (and do not invent draughts board/move as the AC).
- `onMessage` / HTTP handler re-enters itself (or uncleared `setInterval` after dispose) and storms outbound work → hard `[defect]`.
- Suspected re-entry without a proven storm path → **Warning** with cycle edges.

Hard sections below still omit when empty. **Warnings** and **Recommendations** always appear; if empty, a single line `- нет`.

## Report

Write in Russian.

```markdown
## Вердикт
- Постановка: N/10 — <основание; только hard>
- Код проекта: N/10 — <основание; только hard>
- Готовность к тестам: READY | BLOCKED — <основание; только hard defect>
- Регрессии: нет | N — <основание; только hard>
- Режим: requirements-only | post-propose | in-progress
- Источник: активный OpenSpec change <name> (+ уточнения | none + fallback)
- OpenSpec change: <name | none; passed | active | inferred>
- Работа в ветке: <staged / unstaged / untracked / commits>

## Просмотренные файлы
- `path` — <state>

## Пробелы относительно постановки
- [missing|docs-only|code-only|extra] <gap and evidence>

## Пробелы относительно кода проекта
- [gap] <gap> — аналог: `path` — нет в: `changed-file`

## Расхождения docs и кода
- <proven contradiction>

## Явные дефекты перед написанием тестов
- [defect] <condition/current/expected/evidence/file/scenario>

## Регрессии прежнего поведения (без прогона тестов)
- [regression] <hunk class/before/after/justification/file/caller>

## Изменившийся HTTP / room listing / CORS
- [regression|defect] <old method/path/origin/body → new; client/test call sites>

## Room / schema / JWT / move
- [regression|defect] <room name|schema field|onAuth|message type/payload|turn/status; before/after; consumers; why chain breaks>

## Handler / lifecycle-петли
- [defect|warning] <trigger → outbound → re-entry / uncleared timer; expected 1 effect vs storm; file/symbol>

## Async-гонки (first sync / seat assign)
- [defect|warning] <onJoin mutate after first full-state encode without patch | client gap note; file/symbol>

## Предупреждения
- [warning] <risk / ambiguity / likely gap; evidence; what would promote it to hard>
- нет

## Рекомендации
- [recommendation] <optional polish / docs sync / soft parity / test coverage>
- нет

## Карта аналогов
- `path` — <finding supported>

## Требуется решение перед тестами
- <only for BLOCKED from hard defects>

## Что делать дальше
- <hard first; then warnings; recommendations last>
```

Omit empty hard detail sections. Always keep **Предупреждения** and **Рекомендации** (use `- нет` when empty; do not mix items and `нет`).

Never apply fixes from align mode.
