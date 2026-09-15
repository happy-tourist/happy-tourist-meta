---
name: client-align-code
description: >-
  Use when aligning branch and working-tree changes against the active OpenSpec
  change (primary), optional pasted clarifications, repository analogues, test
  readiness, preservation of previous behavior, Vue 3 props/emits/slots, Pinia
  auth/game/theme store public surface, reactive/async feedback loops (watch →
  HTTP/SDK → mutate watched state), async races (await gap before onStateChange /
  listener attach misses first ROOM_STATE), Colyseus room protocol (tourist room /
  lobby / status; turn + `sendMove` / `move`), hash-router requiresAuth/guest guards, and
  Quasar error UX (store error + q-banner).
---

# Align Code

Perform a read-only code alignment audit for the Vue 3 Quasar tourist SPA (`happy-tourist.github.io`) and report findings in Russian across three tiers: hard gaps, warnings, and recommendations.

Stack context: Vue 3 `<script setup>`, Quasar 2, Pinia 4 (`auth` setup store, `game` options store), `@colyseus/sdk` 0.18, vue-router 5 hash mode, vue-i18n 11, TypeScript. Contract surface: Colyseus Auth + room messages via `client` from `src/boot/colyseus.ts`; live lobby via LobbyRoom (`subscribeLobby`); sibling server `../happy-tourist-server`.

**Paths:** this skill currently lives in **this client repo** at `.agents/skills/client/` (temporary). Canonical skills/OpenSpec will move to **happy-tourist-meta** when present (`project-map.md` key `happy-tourist-meta`). Runtime `src/…` paths are relative to **this repository root**. Sibling Colyseus server is **`../happy-tourist-server`**.

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

OpenSpec lives under **happy-tourist-meta**. Resolve meta via `project-map.md` key `happy-tourist-meta` (from client: sibling `../happy-tourist-meta`). If OpenSpec/meta is unavailable and the user did not confirm fallback — ask; do not invent specs.

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

Treat explicit prohibitions and exceptions as atomic requirements: "не отображать", "не добавлять", "скрыть", "только для…", "кроме…".

Auth/guest vs registered flows and route meta (`requiresAuth` / `guest`) are first-class. Game rules live on the server; Game mirrors seats/`currentTurnSessionId` and shows local select/hints — do not treat local layout as authoritative rules.

A tester's guess/question is only a lead. Treat dialog wording as clarification only when it explicitly states or updates expected behavior relative to the active change.

## Atomic Requirement Checklist

For every stated field, independently verify:

- displayed vs intentionally hidden;
- source/path and transformation (e.g. room `status` labels, displayName, theme preference);
- requiredness;
- type and format;
- allowed range, sign, precision, and length;
- default/placeholder;
- editable/read-only/disabled state;
- validation and conditional visibility.

A rendered field is not covered until all explicit properties are covered.

For every request / realtime action, independently verify:

1. triggering UI action and timing;
2. required warning or blocking confirmation when AC demands it (this app has no global dialog registry — confirm via Quasar dialog/`q-dialog` if specified);
3. transport: Colyseus Auth method, LobbyRoom messages (`rooms` / `+` / `-`), or `room.send` message type;
4. every path/query parameter (e.g. lobby filter `name: tourist`, `joinById(roomId)`);
5. every body / options / message payload field, nesting, requiredness, value, and source;
6. actual argument order into store actions / SDK calls;
7. event wiring (`@click` / `emit` / store action / router `push`);
8. response / state-sync handling (`onStateChange`, auth `onChange`, success navigation);
9. required refresh, leave, emit, and post-action notice (store `error` + page `q-banner`, not toasts-as-confirm).

Trace the concrete chain:

```text
UI event → page handler
  → Pinia action (auth | game) → client.auth.* | client.http.* | client.create/join* | room.send
  → onChange / onStateChange / catch → store.error → q-banner | router
```

Example shape (login):

```ts
// LoginPage → auth.login → client.auth.signInWithEmailAndPassword
await auth.login(email, password);
await router.push({ name: 'lobby' });
// failures: auth.error → q-banner
```

Example shape (game enter / board + seats):

```ts
// LobbyPage → game.createGame / joinGame → navigate Game
// GamePage → LAYOUT tiles + pieces from game.seats (no room.send for board UX today)
// room.onStateChange → seats / phase / maxSeats / countdownRemaining / sessionId → status
```

Example shape (lobby list):

```ts
// LobbyPage → game.subscribeLobby → LobbyRoom rooms / + / -
await game.subscribeLobby();
```

Method presence or "looks compatible" alone is insufficient. Track each contract fact separately so one correct component cannot hide another mismatch.

The client↔server contract is Colyseus Auth + room type `tourist` + live `lobby` (LobbyRoom + `.enableRealtimeListing()`) + Game board mirroring seats/`phase`/`maxSeats`/`countdownRemaining`/`currentTurnSessionId` + `sendMove` → `move` `{ side, row, col }` + `sendReady` → `ready`. When CR/docs/server and client disagree, report `code-only` / contradiction with both sides named (`src/stores/*` vs `../happy-tourist-server`).

A toast / silent catch is not blocking confirmation. When confirmation is required, wait for explicit approval; cancel/close must not perform the mutating action. With analogue-only evidence, require only what the analogue proves.

Use `AGENTS.md` and strong in-repo analogues for conventions; never as replacements for requirement evidence.

## Load Branch Work

Inspect read-only (from this client repo root):

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

Read relevant untracked files and full current Vue/TS files when surrounding behavior matters. Do not ignore unstaged work.

## Load Planning Scope

After resolving the change name (see **Resolve OpenSpec change**), from meta:

```bash
openspec status --change "<name>" --json
```

Read concrete artifact paths from the result (`proposal`, `specs`, `design`, `tasks`). Use them as **primary** requirements for **all four axes**, including Axis C edge cases and scenario IDs from delta specs. Report clear docs↔code contradictions. If no change resolved — only after user confirms `OpenSpec: none`, audit from paste/dialog and repository evidence; otherwise stop and ask.

## Axis A — Requirements

Judge implementation first, then planning artifacts.

Report hard gaps only as:

- `missing` — explicitly required and absent;
- `docs-only` — promised by complete artifacts but absent from implemented feature;
- `code-only` — clear docs/code contradiction;
- `extra` — clearly forbidden or unjustified behavior/data.

Ambiguous CR wording, unresolved Open Questions, or unproven leans → **Warnings**, not hard omissions.

Wrong field source/constraint, wrong Auth/HTTP/room API, absent confirm, incorrect event wiring, broken `requiresAuth`/`guest` gating, or client inventing rules the server owns are explicit omissions.

Score **Постановка: N/10** only from hard omissions (`missing` / `docs-only` / `code-only` / `extra`), not from Warnings or Recommendations.

## Axis B — Codebase

Find strong untouched analogues for the same domain/flow. Prefer same layer:

| Concern | Look in |
|---------|---------|
| Pages | `src/pages/*Page.vue` (Login / Lobby / Game) |
| Shared widgets | `src/components/*` |
| Pinia | `src/stores/auth.ts`, `src/stores/theme.ts`, `src/stores/game.ts` |
| Colyseus client | `src/boot/colyseus.ts` |
| Router / guards | `src/router/routes.ts`, `src/router/index.ts` |
| i18n | `src/boot/i18n.ts`, `src/i18n/*` |
| Env | `.env.development`, `.env.production`, `env.d.ts` |
| Sibling server | `../happy-tourist-server` (room schema / auth / HTTP) |

Compare notices, loading (`loading` / `listing` / `status === 'connecting'`), errors (store `error` + `q-banner`), auth readiness (`whenReady`), route meta, room enter/leave, empty lobby lists, and rejoin-on-refresh only when the analogue proves the behavior.

Prefer Colyseus I/O inside Pinia (`auth`, `game`), not scattered `client.*` in many components — flag deviations as codebase-implied when analogues keep that boundary.

Report `gap` only with:

- concrete analogue path;
- changed file lacking the behavior;
- why parity is required rather than optional polish.

Label analogue-driven findings as codebase-implied. Score **Код проекта: N/10** only from hard `gap` items (not Warnings / Recommendations).

## Axis C — Test Readiness

Use evidence in this order:

1. explicit requirements — including the **resolved OpenSpec change** (delta specs scenarios/SC-*, design constraints, task acceptance) when present; do not skip change artifacts and invent generic edge cases instead;
2. Pinia action/getter contracts, Colyseus message/state shape, props/`emit` contracts, validators;
3. strong analogues;
4. deterministic runtime semantics (auth ready gate, room leave/rejoin, Game board + turn/`sendMove`).

For reachable behavior, examine permitted empty/null/zero/false states, constrained numeric/string boundaries, initial/loading/success/empty/error/retry states, repeated actions, async cleanup, tourist board layout rendering, conditional rendering by auth/room status, `q-banner` visibility, and validation-vs-handler mismatches — **and** every edge/negative path named or implied by the resolved change's specs/design (e.g. empty lobby snapshot, subscribe/unsubscribe, leave-before-enter).

Runtime facts that often create defects:

- router `beforeEach` awaits `auth.whenReady()` then enforces `requiresAuth` / `guest`;
- `leaveGame` swallows leave errors on already-closed rooms;
- `GamePage` calls `rejoinGame(roomId)` if Pinia lost the room (token → `joinById`); failed rejoin → lobby;
- Game board mirrors seats + `currentTurnSessionId`; local select/hints on `isMyTurn`; submit only via `game.sendMove` (not page `room.send`).

### Tourist board CSS sizing (обязательно при касании Game board)

Когда diff/ветка трогает `GamePage.vue` board CSS / layout (`tourist-board`, `grid-template-*`, `--tile`, tile chrome) — **отдельно** проверь, что тайлы получают ненулевую высоту на типичном viewport (контейнер с `width: 100%` и **auto** height).

Инвариант (SC-BOARD-02/03 / design D3): сторона тайла ≤ 60px, gap 6, radius 12; на узком экране доска тянется по ширине content area; клетки **квадратные** (высота = ширина трека).

Для `.tourist-board` независимо проверь:

1. **Ось rows** — чем задана высота рядов (`grid-template-rows` / `grid-auto-rows` / явный `height` на `.tile`).
2. **Резолв `%`** — если размер ряда/тайла зависит от `%` (в т.ч. через `--tile: … calc((100% − …) / 10)`), `%` для **block axis** считается от **высоты** контейнера; при `height: auto` это **0** → все тайлы высотой 0 (невидимое поле).
3. **Safe patterns** — `aspect-ratio: 1` на доске + `repeat(10, 1fr)` по обеим осям; или ширина от контейнера + `aspect-ratio: 1` на клетке **без** `%`-высоты рядов. Cap max tile 60px через `max-width: calc(10 * 60px + 9 * 6px)` (или эквивалент).
4. **Unsafe patterns** — `grid-template-rows` / `height` тайла из `var(--tile)` / `min(60px, calc((100% − …) / N))`, когда тот же `%` должен работать и для rows при auto-height родителя.

Report hard `[defect]` when Game board CSS deterministically collapses tile/row height to 0 (or equivalent invisible board) on a reachable Game screen. If sizing looks risky but proof is incomplete → **Warning** with the suspected `%` / auto-height edge.

### Reactive / async feedback loops (обязательно)

Когда diff/ветка трогает `watch` / `watchEffect`, boot sync, Pinia action с `client.http.*` / `client.auth.*` / `room.send`, или мутацию store-полей из ответа async — **отдельно** проверь, нет ли закальцованности (повторный шторм запросов / эффектов). Ожидание «один restore / один save на событие» — дефолт, пока AC явно не требует polling.

Для каждой такой цепочки независимо проверь:

1. **Триггер** — что именно запускает эффект (`watch` source, `immediate`, `onChange`, route enter).
2. **Side effect** — какой outbound I/O (`client.http.get/post`, auth, `room.send`) или тяжёлая работа внутри.
3. **Мутация watched-состояния** — пишет ли успех/ошибка в поля, от которых зависит тот же `watch` / `watchEffect` (прямо или через новый объект `user = { ...user }`).
4. **Сравнение source** — getter, возвращающий **новый** массив/объект каждый раз (`() => [a, b]`), при любом invalidate даёт «новое» значение по ссылке даже при тех же примитивах → callback снова; предпочтительнее `watch([() => a, () => b], …)` или примитивный/строковый ключ.
5. **Guard** — есть ли in-flight / generation / «уже синкнули для этого identity»; generation, который только игнорирует stale **ответы**, но не блокирует **новые** вызовы, **не** разрывает петлю.
6. **Кратность** — на один логический event (reload, login, identity change) ожидается конечное малое число запросов (обычно 1); бесконечный или «пачка подряд» без нового user action → hard `defect`.

Типичные петли для флага:

```text
watch(auth fields) → GET/POST → patch auth.user / preference
  → watch again → GET/POST again → …
```

- `watch(() => [ready, user?.id, …])` + после ответа `auth.user = { ...auth.user, theme }` (или любая замена watched object);
- `watchEffect` читает store A, пишет в A/B, что снова инвалидирует effect;
- subscribe/listener без unsub + повторный mount → накопление handlers и дубль I/O;
- retry/error path, который снова ставит то же reactive условие без backoff/stop.

Report as hard `[defect]` when a reachable path deterministically storms HTTP/SDK/room messages or spins the effect loop. If the chain looks risky but proof is incomplete → **Warning** with the suspected cycle edges.

### Async races — await gap before listener / first sync (обязательно)

Когда diff/ветка трогает room enter (`_enterRoom` / `create` / `join*` / `_attachRoom`), `room.onStateChange` / `onError` / `onLeave`, lobby `subscribe`/`unsubscribe`, auth `onChange`, или любой path «promise resolve → потом повесить listener / прочитать первый sync» — **отдельно** проверь гонки: первый inbound event (часто единственный) уходит в никуда, UI остаётся пустым.

Контекст Colyseus: `join`/`create` resolve на `JOIN_ROOM`; полный `ROOM_STATE` (с `seats` / `phase` / `maxSeats`) приходит **после** client ACK, часто в узком окне. Любой `await` между `connect()` resolve и регистрацией `onStateChange` может проглотить первый sync — последующих patch может не быть → Pinia `seats[]` навсегда `[]`, доска без фигурок.

Для каждой такой цепочки независимо проверь:

1. **Точка готовности I/O** — что именно означает «уже подключены» (`await client.create/join*`, `await http.get`, auth ready).
2. **Первый критичный inbound** — `ROOM_STATE` / первый `onStateChange`, первый Lobby `rooms`/`+`, первый `auth.onChange` с user — событие, без которого UI/store ломается.
3. **Регистрация listener / mirror** — где вешается `onStateChange` / `onMessage` / `onChange`; есть ли одноразовый mirror текущего `room.state` / уже известного snapshot.
4. **Await-gap** — есть ли `await` (lobby leave, navigation, лишний I/O, `nextTick`) **между** resolve connect и attach listener / mirror. Любой gap → подозревай missed first event.
5. **Идемпотентность после пропуска** — если первый sync пропущен, придёт ли повторный patch сам, или состояние «застынет» (типично для seating: один full state, дальше тишина) → hard defect.
6. **Порядок в `_enterRoom`** — канон: `await connect()` → **сразу** `_attachRoom(room)` (listener + optional `room.state` mirror) → только потом `await unsubscribeLobby()` / прочий cleanup. Обратный порядок (`unsubscribe`/`await` → потом `_attachRoom`) — hard `[defect]`.

Типичные гонки для флага:

```text
await connect()           // JOIN_ROOM done, ACK sent
await unsubscribeLobby()  // ← gap: ROOM_STATE arrives, no listener
_attachRoom(room)         // onStateChange too late → seats stay []
```

- `await connect()` → `await router.push` / page mount → только там `onStateChange` (listeners must stay in store, and before other awaits);
- subscribe Lobby / auth `onChange` after first message already delivered without snapshot read;
- fire-and-forget connect without attach on the success path.

Report as hard `[defect]` when a reachable enter/subscribe path can miss the first sync/event and leave store/UI wrong with no further update. If ordering looks risky but proof is incomplete → **Warning** with the await-gap edges.

Report hard `defect` only when a reachable state deterministically causes wrong UI, runtime failure, invalid value, stuck state, unsafe side effect (including request/effect storms **or missed first-sync races**), or contract violation.

Each hard defect includes condition, current behavior, expected invariant/evidence, file/symbol, and minimum scenario. Verdict:

- `READY` — no proven deterministic hard defect found;
- `BLOCKED` — at least one hard defect must be resolved or explicitly characterized before tests.

Unresolved Open Questions / ambiguous error-branch splits that do not yet prove wrong UI go to **Warnings**, not `BLOCKED`, unless a reachable wrong message/state is already deterministic.

## Axis D — Behavior Preservation

Classify every changed hunk:

- `task` — directly justified by an atomic requirement;
- `incidental` — refactor/rename/cleanup/shared-helper change;
- `neighbor` — unrelated changed behavior in a task file.

Compare before vs after using diff/base, contracts, and callers. Report `regression` only for proven prior behavior changed without task justification: Pinia public actions/getters/state fields, Colyseus message/state contracts, UI conditions/`emit`/prop defaults, route meta/guards, shared-helper results, or auth token flow.

Do not run tests. Do not double-count the same issue as both defect and regression unless the evidence and impact differ.

### Changed props, emits, slots, and DOM contract

Shared Vue 3 components use `<script setup>` `defineProps` / `defineEmits` / slots. When a changed file touches props, emits, slot names, `inheritAttrs`, or `v-bind="$attrs"`, audit the public contract separately from business logic.

For every removed, renamed, or newly namespaced prop/event/slot, independently verify:

1. **Old public surface** — which props, kebab-case attrs on the tag, listeners, and slots existed before.
2. **New resolution path** — where the value must come from now (explicit prop, `$attrs`, `v-model` / update emits).
3. **Caller migration** — grep for the component tag and old prop/event names; untouched call sites after a rename are strong regression evidence.
4. **Rendered DOM/effective value** — what lands on Quasar root / native element; spreading stale attrs is not the same as mapping them.
5. **Indirect paths** — pages wrapping components; Pinia getters feeding props.

Typical regression patterns to flag:

- prop removed, parents still pass old kebab-case attr;
- `emit('…')` renamed without updating parents;
- slot name changed without updating consumers;
- `inheritAttrs: false` added without re-binding previous fallthrough attrs.

Report as `regression` or `defect` when old call sites deterministically stop receiving the intended class, listener, or prop. Mention approximate caller count when grep proves it.

### Pinia, Colyseus protocol, and route meta wiring

This SPA wires cross-tree contracts through Pinia stores, the Colyseus client/room protocol, and router meta. Treat them as public API of the same risk class as props.

When a changed hunk touches `stores/auth`, `stores/theme`, `stores/game`, `boot/colyseus`, `boot/theme`, room `send`/`onStateChange`, App-level `watch` on auth, or `router` meta/guards, independently verify:

1. **Pinia still wired** — `auth` setup-store exports (`register` / `login` / `loginAnonymously` / `logout` / `whenReady` / `isAuthenticated` / `displayName` / `error` / …), `theme` (`syncFromAuthUser` / `toggle` / …), and `game` options-store actions/getters (`subscribeLobby` / `unsubscribeLobby` / `createGame` / `joinGame` / `leaveGame` / `sendMove` / `isMyTurn` / `isInRoom` / …) still match callers; renamed action with old call sites is a regression.
2. **Room protocol** — room names `TOURIST_ROOM = 'tourist'`, `LOBBY_ROOM = 'lobby'`; live listing via LobbyRoom subscribe (HTTP `refreshRooms` unused fallback only); `sendMove` → `move` `{ side, row, col }`; mirror `currentTurnSessionId` (do not require draughts cells `0`–`4`).
3. **Auth lifecycle** — `client.auth.onChange` drives `token`/`user`/`ready`; token key `colyseus-auth-token`; protected routes wait for `whenReady()`.
4. **Theme / preference sync** — registered restore via profile HTTP must not re-enter on every in-memory userdata patch; guest stays localStorage-only; Colyseus HTTP only from store (see **Reactive / async feedback loops**).
5. **Route meta** — `/login` has `meta.guest`; `/lobby` and `/game/:roomId` have `meta.requiresAuth`; hash mode (`/#/…`). Removing or flipping meta without AC is `extra` / `missing`.
6. **Error UX** — failures land in store `error` and pages show `q-banner`; do not expect SHOW_DIALOG / axios interceptors / brand profiles (absent in this app).

Typical regression patterns to flag:

- store action/getter renamed but pages still call old names;
- calling `room.send('move')` from GamePage instead of `game.sendMove`, or inventing alternate turn/move shapes;
- `onStateChange` regresses leave/rejoin or invents unsynced authority on the client;
- `_enterRoom` awaits lobby leave (or anything else) before `_attachRoom` / `onStateChange` → missed first `ROOM_STATE`;
- guard no longer awaits `whenReady()` or ignores `requiresAuth`/`guest`;
- `leaveGame` / rejoin path breaks refresh recovery;
- shared util return shape changed; callers assume old shape;
- theme/auth sync watch storms `GET`/`POST` after patching the same watched user object.

Report as `regression` or `defect` when consumers deterministically lose state, skip auth gates, hide/show errors incorrectly, or storm preference/auth HTTP. Name the store key / message type / route meta field and consumer paths.

## Severity

Every finding goes into exactly one tier. Do not silence softer items; do not inflate them into hard gaps.

| Tier | When | Align cues |
|------|------|------------|
| **Hard gaps** | Explicit requirement / forbidden behavior / required analogue parity / deterministic wrong state / proven unjustified regression | `missing`, `docs-only`, `code-only`, `extra`, required `gap`, `defect`, `regression`; scores and `BLOCKED` use **only** this tier |
| **Warnings** | Strong lean without full proof; ambiguous AC with a clear risk; desirable analogue parity not proven mandatory; Open Question that can change expected behavior; likely client↔server mismatch pending API/аналитика | `[warning]`; does **not** lower scores or force `BLOCKED` alone |
| **Recommendations** | Optional polish vs analogue; docs/OpenSpec sync; unchecked planning tasks; test-coverage ideas that are not defects; soft UX parity | `[recommendation]`; informational only |

Examples:

- GUI requires field X and code omits it → hard `[missing]`.
- AC unclear whether guest may create rooms; code allows it and no wrong UI is proven → **Warning** until discriminant is settled.
- Analogue always shows `q-banner` on store `error` after join failure and CR says «как в лобби» → hard `[gap]` if that page is the cited evidence; if CR is silent → **Recommendation**.
- OpenSpec `tasks.md` still unchecked while code exists → **Recommendation** (docs sync), not hard requirements gap.
- Ambiguous CR wording that was misread once already → **Warning** with both readings, ask for confirmation rather than hard `missing`.
- AC forbids unauthenticated lobby access and route loses `requiresAuth` → hard `[extra]` / `[missing]` gating as appropriate.
- `watch` on auth identity fires `GET /api/theme`, then success replaces `auth.user` and re-triggers the same watch → hard `[defect]` (request storm / feedback loop).
- Watch source returns a fresh array each run and any invalidate of deps re-fires I/O even when primitives unchanged → hard `[defect]` when that I/O is proven; otherwise **Warning**.
- `await connect()` then `await unsubscribeLobby()` then `_attachRoom` → hard `[defect]` (missed first `ROOM_STATE` / empty `seats` on Game).
- `GamePage` `.tourist-board` sets `grid-template-rows` from `--tile` with `100%` while the board has auto height → hard `[defect]` (tile height 0 / invisible board).

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

## Изменившиеся props / emits / слоты и публичный контракт
- [regression|defect] <old prop/event/slot → new path; DOM; call sites>

## Pinia / Colyseus / маршруты
- [regression|defect] <action|getter|message|state field|route meta; before/after; consumers; why broken>

## Реактивные / async-петли
- [defect|warning] <trigger → side effect → mutated watched state → re-trigger; expected 1 call vs storm; file/symbol>

## Async-гонки (await до listener / первый sync)
- [defect|warning] <connect/subscribe resolve → await gap → missed onStateChange/ROOM_STATE|first message; attach/mirror order; file/symbol>

## Tourist board CSS (высота тайлов)
- [defect|warning] <% / auto-height → rows 0 | safe aspect-ratio+fr; file/selector; SC-BOARD / D3>

## Предупреждения
- [warning] <risk / ambiguity / likely gap; evidence; what would promote it to hard>
- нет

## Рекомендации
- [recommendation] <optional polish / docs sync / soft parity>
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
