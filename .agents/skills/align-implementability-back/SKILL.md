---
name: align-implementability-back
description: >-
  Use when judging whether happy-tourist-meta server/BE claims are implementable
  (possibility to build only — not meta doc quality, not client UI layout).
  Against current happy-tourist-server (Colyseus rooms/schema/messages/routes/
  auth/DB/game via projects-map). Also warn when client/UI docs in the change
  family describe data/actions with no server/HTTP/room/schema claim in meta
  (`[fe-only-without-be]`). Classifies aligns/extends/conflicts/unproven;
  verdict FEASIBLE|PARTIAL|INFEASIBLE|UNKNOWN. Finding card: file + lines +
  issue + fix. Not client UI realizability (align-implementability-front; keeps
  `[be-without-fe]`) or meta quality (align-meta). Does not edit files.
---

# Align Implementability (Backend / Server)

Perform a read-only **server implementability** audit: can the **happy-tourist-meta** change be built **as written** against **happy-tourist-server**? Report findings in Russian.

**Scope of judgment — only possibility to implement.** Answers «можно ли реализовать server/контракт по этим claims», not «идеальны ли AC», not UI layout. Completeness/quality of meta → **`align-meta`**. Client screens/routes fit → **`align-implementability-front`** (там же `[be-without-fe]`).

Новый message/route/schema-поле/алгоритм, которого ещё нет в sibling HEAD, но есть ясный аналог для **extends** — нормальная реализуемость, не hard gap. Hard только при **conflicts** с доказанным контрактом/кодом или required parity misstatement.

Output **one scope** with **three tiers** (hard → warnings → recommendations):

1. **Server implementability** — claims vs sibling server evidence (`aligns` / `extends` / `conflicts` / `unproven`).
2. **Client→Server coverage in meta** — UI/client claims in the change family that lack a server/HTTP/room counterpart (`[fe-only-without-be]`).

Every finding is a **card**: file link + line numbers + what is wrong + fix proposal (or «неясно как делать»).

## FE-only-without-BE (обязательно)

Проверяй согласованность **meta UI/client → meta server/API** в change set (+ Parent/Child из `proposal.md`, если есть):

| Ситуация | Как судить |
|----------|------------|
| В UI/client docs описано отображение или ввод данных (поле, статус, действие, HTTP/room message), а в **наборе claims ветки / change и связанных Parent/Child** нет соответствующего server/HTTP/room/schema/DB claim | **Warning** `[fe-only-without-be]` — риск «UI спроектирован, контракта/server-описания нет». **Не** hard |
| В UI есть поле/действие, которого нет в sibling HEAD, но в meta BE/server claim **есть** | Не `[fe-only-without-be]`. Обычный **extends** / lag siblings |
| Ветка явно server-only **без** client children и без UI docs в change family | Не сканируй UI массово; `[fe-only-without-be]` не ставь |

**Что считать потребностью в server для `[fe-only-without-be]`:** поле формы/таблицы, кнопка с действием по room message / HTTP, отображение schema-поля, create options (`mapId`/`packId`/…), auth/role gate, отдельная история/детали с API.

**Не** тащить сюда: Vue routes/pages layout, i18n-only, FE validation UX без данных → `align-implementability-front`. Обратное (контракт есть, UI нет) → **`[be-without-fe]`** только во **front**-скилле.

Не сравнивай sibling server-код с sibling client-кодом «уже реализовано ли» — для этой метки только **meta** UI vs **meta** server.

## Runtime roots

- **Claims subject:** changed / relevant `happy-tourist-meta` artifacts that assert **server** runtime: Colyseus rooms/lifecycle, `@colyseus/schema` fields, `onMessage`, HTTP routes (`/api/...`), auth (`@colyseus/auth`), Drizzle/DB, game algorithms, server skills. For `[fe-only-without-be]` also **read** UI/client analytics in the same change family as *demand* signals.
- **Evidence code:** resolve via [`docs/projects-map.md`](../../../docs/projects-map.md) (+ `projects-map.local.yaml`). Primary: `server` → `../happy-tourist-server/`. Skills: `.agents/skills/server/`. **Not** `client` as primary evidence for axis 1.
- Do **not** treat sibling working trees as the primary diff to “fix”.

## Scope filter (claims)

**Include** atomic claims about:

1. Room name / lifecycle (`onCreate`/`onJoin`/`onLeave`/`onDrop`/`onReconnect`/`onDispose`);
2. Schema fields / synced state / private messages;
3. `onMessage` handlers and payload shapes;
4. HTTP path / method / status / body (Express routes);
5. Auth / roles / JWT `onAuth` / email flows as server behavior;
6. DB tables / Drizzle schema / persistence rules;
7. Game algorithms (move, traps, peek, budgets, …) owned by server;
8. Server skill procedures (directories under `happy-tourist-server`, conventions);
9. For coverage scan only: UI/client fields/actions that imply a server contract (`[fe-only-without-be]`).

**Exclude** (defer to `align-implementability-front` with at most a one-line pointer):

- Vue routes, pages, components, stores as UI, i18n, FE validation UX **as FE implementability**;
- `[be-without-fe]` — owned by front skill;
- FE consumption casing/binding details unless they force a **server contract** contradiction already in scope.

If the branch is client-only and has **no** server claims: всё равно прогони `[fe-only-without-be]` по UI docs vs Parent/Child server claims; axis sibling-fit может быть **`UNKNOWN`**/тонким.

## Required Axis

1. **Server codebase fit / implementability** — server claims vs strong evidence in `happy-tourist-server`.
2. **Client→Server meta coverage** — scan UI/client claims for missing server/API (`[fe-only-without-be]`); skip mass scan only when branch is clearly server-only without UI docs/children.

Optionally note meta-internal defects only as a one-line pointer to `align-meta`.

## Hard Boundary

Do not edit, create, delete, format, or commit files. Do not apply fixes from align mode. Deliver the complete result in chat. Verdict is evidence only (`править не надо`).

## Input And Store

Accept pasted requirements and an optional OpenSpec change name. If the branch clearly targets an OpenSpec change and no extra source is given, audit from the change artifacts + server evidence; ask only when the goal cannot be inferred.

If a standalone OpenSpec store is named, run `openspec store list --json`, resolve its id, and pass `--store <id>`. Otherwise use **`happy-tourist-meta/openspec/`**.

## Load Branch Work (claims set)

Inspect read-only **in happy-tourist-meta** (repo root):

```bash
git status --short
git diff
git diff --cached
git branch -vv
```

Resolve merge-base: prefer `origin/master`, else `origin/main`, `master`, `main` — first ref that exists.

When the branch has commits beyond base:

```bash
git merge-base HEAD <base-ref>
git log --oneline <merge-base>..HEAD
git diff --stat <merge-base>...HEAD
git diff <merge-base>...HEAD
```

Also collect untracked: `git ls-files --others --exclude-standard` and read those files.

Union of commits + staged + unstaged + untracked is the claim file set. Narrow only when the user explicitly scopes. Then **filter** to server-relevant claims.

После фильтра — быстрый проход **UI/client → server** по тому же change set (+ Parent/Child):

1. Для каждого поля/действия/эндпоинта, которое UI обещает показать или вызвать, есть ли server/HTTP/room/schema claim в meta? Нет → Warning `[fe-only-without-be]`.
2. Есть в meta, нет в sibling HEAD → не `[fe-only-without-be]`; судить как **extends** / lag.
3. Ветка server-only без UI docs → шаг 1 не раздувать.

Обратное (контракт есть, UI нет) **не** проверяй здесь → `align-implementability-front` / `[be-without-fe]`.

## Load Sibling Evidence (server)

For each atomic server claim:

1. Resolve `server` via projects-map (+ local YAML).
2. Grep/read rooms, schema, messages, routes, auth, db, game libs, tests that prove or refute the claim.
3. Prefer current HEAD of the sibling clone; if missing, **Warning** (`sibling-missing`), not a hard gap.

Do not invent sibling APIs. Absence of a clone ≠ proof that meta is wrong. Before deep reads — `{server}/AGENTS.md`.

## Axis — Server implementability judgment

Compare independently per claim:

1. **Schema / state** — field exists or clear analogue for **extends**; typing/sync match;
2. **Room message / HTTP** — name, payload, auth gate match evidence or analogue;
3. **Lifecycle** — claimed hooks match real room patterns;
4. **Persistence / auth** — Drizzle / `@colyseus/auth` patterns;
5. **Game rule** — algorithm sits on existing tourist-room patterns;
6. **Skill procedure** — server directories and conventions under `.agents/skills/server/`;
7. **Edge paths** — error, empty, permission, reconnect when the change claims completeness.

Classify each claim: **aligns** | **extends** | **conflicts** | **unproven**.

Hard `gap` only with concrete sibling evidence + meta misstatement + required parity. Use `code-only` when meta asserts a contract that proven sibling evidence contradicts.

**Absence of the new API/field in sibling HEAD is not automatically a hard gap** when meta clearly specifies an **extends** path. Hard when the claim **conflicts** proven contract/code.

Score **Код проекта: N/10** only from hard `gap` / `conflicts` / required `code-only` items. `[fe-only-without-be]` **не** снижает score.

### Суждение о реализуемости

| Вердикт | Когда |
|---------|--------|
| `FEASIBLE` | No hard `conflicts`; claims **aligns** or **extends**; remaining risks are Warnings only |
| `PARTIAL` | Mixed aligns/extends vs hard gap/conflicts, or critical pieces **unproven** |
| `INFEASIBLE` | Hard **conflicts** / required `gap` — **не** только потому что «в HEAD ещё нет» при ясном extends |
| `UNKNOWN` | Not enough sibling evidence — do **not** invent APIs; list what was searched |

`UNKNOWN` alone → Warning, not a score hit.

## Severity

| Tier | When | Align cues |
|------|------|------------|
| **Hard gaps** | Required parity missing or claim conflicts proven sibling evidence | `gap`, `conflicts`, `code-only` (when parity required) |
| **Warnings** | Strong lean without full proof; sibling missing; critical `unproven`; UI in meta without server in meta | `[warning]`; `unproven`; `[fe-only-without-be]` |
| **Recommendations** | Optional polish vs analogue; soft wording | `[recommendation]` |

Examples:

- Spec claims message `moveX` but room proves `move` with different payload → hard `conflicts` / `code-only`.
- New private message absent but `budgets` analogue exists → **extends** — **FEASIBLE**.
- Sibling not cloned → **Warning** `[sibling-missing]`.
- UI показывает поле без schema/HTTP/room claim в meta → **Warning** `[fe-only-without-be]`.
- Broken link in proposal → **not this skill** → `align-meta`.
- Vue page / layout missing → **not this skill** → `align-implementability-front`.

## Finding card (обязательный формат)

```markdown
### <N>. <короткий заголовок>
- Метка: `[gap|code-only|warning|recommendation|fe-only-without-be]` (+ обязательно `aligns|extends|conflicts|unproven` для runtime-claim)
- Файл: [`path/from/repo/root.ext`](path/from/repo/root.ext) — **L12–L18** (или **L12, L45**)
- Связанные (если нужно): [`../happy-tourist-server/...`](../happy-tourist-server/...) — **L3–L5**
- Что не так: <факт: meta / sibling / противоречие>
- Предложение: <конкретная правка meta-контракта или уточнение extends>  
  **или** Неясно как делать: <варианты A/B без выбора канона>
```

Пути — от корня `happy-tourist-meta` или sibling (`../happy-tourist-server/...`). Строки по актуальному диску.

## Report

Write in Russian.

```markdown
## Вердикт
- Код проекта (server): N/10 — <основание; только hard; fe-only-without-be не снижает>
- Реализуемость server (только возможность): FEASIBLE | PARTIAL | INFEASIBLE | UNKNOWN — <кратко; новый extends ≠ INFEASIBLE>
- Режим: docs/openspec | skills | mixed | server-only
- Источник claims: <OpenSpec change / docs / skill>
- Работа в ветке meta: <staged / unstaged / untracked / commits>
- Предыдущий шаг: при необходимости сначала `align-meta`
- Client↔Server в meta: перечислить `[fe-only-without-be]`; UI-fit и `[be-without-fe]` → `align-implementability-front`

## Claims (meta, server)
- `path` — <какие server runtime-утверждения проверяли>
- UI→server gaps: <кратко или «нет»>

## Доказательная база (server sibling)
- `sibling-path` — <что подтвердили / не нашли / clone missing>

## Замечания по возможности реализовать (server)

### Ошибки (hard)
- нет

### Предупреждения
- нет

### Рекомендации
- нет

## Карта аналогов (server)
- `path` — <зачем смотреть>

## Что делать дальше
- <hard → warnings → recommendations; UI-fit / be-without-fe → align-implementability-front; meta-quality → align-meta>
```

Never apply fixes. Карточки — текстом; файлы не менять.

## Not This Skill

| Skill | Difference |
|-------|------------|
| `align-implementability-front` | Client UI/routes realizability; owns `[be-without-fe]` |
| `align-meta` | Quality of meta artifacts — not sibling realizability |
| `align-code` | Runtime branch vs requirements after code exists |
| `openspec-verify-change` | Implementation vs OpenSpec (after code exists) |
| `server-align-code` / `server-verify-code` | Server code quality/compliance, not meta claims vs siblings |
