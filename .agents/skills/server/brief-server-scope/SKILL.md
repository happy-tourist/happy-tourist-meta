---
name: brief-server-scope
description: >-
  Read-only server/BE scope brief from an OpenSpec change: contract summary
  (room/schema/messages/HTTP/auth/DB), planned server behavior, reuse vs adapt
  vs new, plus review cards for расхождения/пробелы/неоднозначности. Use when
  the user asks to brief server scope, разбери постановку на бек/server, what
  the server must do from the change, or to understand BE work before coding.
  Chat report only by default; does not prepare a per-file change plan
  (server-locate-change-points) or audit code (server-align-code).
---

# Brief Server Scope

Read-only brief of **what the server must expose and enforce**, grounded in an OpenSpec change (+ Parent/Child), plus a short **reuse / adapt / new** readout vs `happy-tourist-server`.

**Home:** `.agents/skills/server/` in **happy-tourist-meta**. Runtime evidence: projects-map key `server`.

## Hard Boundary

- Do **not** edit, create, delete, format, or commit files — except when called from [`brief-change`](../../brief-change/SKILL.md) (override: after report, apply meta fixes) or the user **explicitly** asks to change named meta files / this skill.
- Deliver the **entire** result **in chat only** (unless override above).
- Do **not** run `server-locate-change-points` or produce a «куда класть файлы» plan.
- Do **not** run `align-implementability-back` / `server-align-code` (implementability / code vs AC).
- **Allowed for reuse:** light read of `happy-tourist-server` (via projects-map) and sibling changes — as analogs, not as an implementation checklist of new paths.

**Report language: Russian.** Keep room names, message names, schema fields, HTTP paths as in the sources.

## When To Use

- Clear server picture from a change before implementation / align
- Triggers: brief server scope / разбери постановку на бек / что должен сделать server / contract digest
- After OpenSpec artifacts exist

## Relation To Other Skills

| Skill | Role relative to this |
|-------|------------------------|
| [`brief-client-scope`](../../client/brief-client-scope/SKILL.md) | Client UI digest — pair via `brief-change` |
| `align-implementability-back` | Can we build it? — not this |
| `server-locate-change-points` | Where to edit files — not this |
| `prepare-changes` / `openspec-propose` | Create/slice change — not this |

## Input

Resolve from the user message (ask only if nothing resolves):

| Input | Meaning |
|-------|---------|
| OpenSpec change name | Primary: `openspec/changes/<name>/` |
| Pasted excerpts | Secondary if tools/paths unavailable |

Optional filters: «только change» (no sibling runtime for issues — reuse may still peek server); «без sibling»; «остальное готово» (assume client already planned — do not list missing UI as gaps).

## Source Priority

1. **OpenSpec change** wins (`proposal`, `design`, `specs`, `tasks`).
2. Conflicts **inside** the change → **Расхождения** cards.
3. For **Переиспользование**: sibling changes + light server read; do not invent reuse.

Do not invent messages, schema fields, routes, or algorithms not in the loaded docs.

## Workflow

1. Resolve change root (`openspec status --change "<name>" --json` when useful).
2. Read server-relevant artifacts: proposal, design, delta specs, tasks. Honor exclusions («не смотри design»).
3. Extract server-facing scope: room/lifecycle, schema/sync, messages, HTTP, auth, DB, game rules.
4. **Reuse scan:** docs + siblings + light server look → already exists / adapt / new.
5. Collect **расхождения / пробелы / неоднозначности** → review cards.
6. Write the chat report. Stop (unless brief-change override).

## Output Format

Stacked lists. **Do not use markdown tables** in the report body (cards OK with bullets).

```markdown
## Контекст
- change: …
- Parent/Child: … (если есть)

## Контракт
- room name / lifecycle — что server владеет
- schema / synced / private payloads — ключевые поля
- messages — имя + payload + когда
- HTTP — method path + зачем
- auth / roles — onAuth / gates
- DB — таблицы/поля если в постановке

## Поведение server
- правила / алгоритмы из постановки
- ошибки / reject / empty — только если описаны

## Переиспользование
- Уже есть / as-is
- Доработать
- С нуля
(пустой список — «нет»)

## Расхождения
… (карточки; опустить если пусто)

## Пробелы
… (опустить если пусто)

## Неоднозначности
… (если пусто — «не найдено»)
```

### Карточки

Каждый пункт:

```markdown
### <короткий заголовок>
- **Что не так:** …
- **Предлагаю:** …
- **Ревью**
  - файл: `openspec/changes/<name>/…`
  - строки: Lnn
  - сообщение: <paste-ready без markdown-ссылок>
```

Якорь — одно место правки. Не выдумывать API/поля в **Контракт** / **Поведение**.

## Не делать

- Не plan файлов, не apply runtime, не commit.
- Не путать с implementability audit.
- Не предлагать follow-up skills списком в конце (кроме краткого «далее: brief-client / locate» если пользователь спросил next step).
