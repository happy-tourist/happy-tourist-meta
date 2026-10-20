---
name: brief-client-scope
description: >-
  Read-only client/FE scope brief from an OpenSpec change: contract binding
  summary (room/HTTP as client uses them), planned UI, fields/display, reuse vs
  adapt vs new, plus review cards for расхождения/пробелы/неоднозначности. Use
  when the user asks to brief client scope, разбери постановку на фронт/client,
  what the UI should show, or to understand FE work before coding. Chat report
  only by default; does not prepare a per-file plan (client-locate-change-points)
  or pixel-verify (verify-mock).
---

# Brief Client Scope

Read-only brief of **what the client should show and how it binds the contract**, grounded in an OpenSpec change (+ Parent/Child), plus a short **reuse / adapt / new** readout vs `happy-tourist.github.io`.

**Home:** `.agents/skills/client/` in **happy-tourist-meta**. Runtime evidence: projects-map key `client`.

## Hard Boundary

- Do **not** edit, create, delete, format, or commit files — except when called from [`brief-change`](../../brief-change/SKILL.md) (override: after report, apply meta fixes) or the user **explicitly** asks to change named meta files / this skill.
- Deliver the **entire** result **in chat only** (unless override above).
- Do **not** run `client-locate-change-points` or produce a «куда класть файлы» plan.
- Do **not** run `align-implementability-front` / `client-align-code` / `verify-mock` / `prepare-mock`.
- **Allowed for reuse:** light read of `happy-tourist.github.io` (via projects-map) and sibling changes — as analogs, not as an implementation checklist.

**Report language: Russian.** Keep UI labels, message names, store keys, routes as in the sources.

## When To Use

- Clear client picture from a change before implementation / align
- Triggers: brief client scope / разбери постановку на фронт / что показывать в UI / field digest
- After OpenSpec artifacts exist

## Relation To Other Skills

| Skill | Role relative to this |
|-------|------------------------|
| [`brief-server-scope`](../../server/brief-server-scope/SKILL.md) | Server contract digest — pair via `brief-change` |
| `align-implementability-front` | Can we build UI? / Visual Spec completeness — not this |
| `client-locate-change-points` | Where to edit Vue — not this |
| `prepare-mock` / `verify-mock` | Measure / pixel-check mock — not this |

## Input

Resolve from the user message (ask only if nothing resolves):

| Input | Meaning |
|-------|---------|
| OpenSpec change name | Primary: `openspec/changes/<name>/` |
| Pasted excerpts | Secondary if needed |

Optional filters: «только change»; «без sibling»; «остальное готово» (assume server/contract already delivered — do not list missing server as gaps).

## Source Priority

1. **OpenSpec change** wins (`proposal`, `design`, `specs`, `tasks`, Visual Spec section if present).
2. Conflicts inside the change → **Расхождения**.
3. For **Переиспользование**: sibling changes + light client read.

Do not invent screens, fields, empty/error, or permissions not in the loaded docs.

## Workflow

1. Resolve change root.
2. Read client-relevant artifacts: proposal, design (incl. Visual Spec if any), delta specs, tasks.
3. Extract UI scope: pages/routes, components/modals/HUD, actions, bindings to messages/HTTP, permissions, loading/empty/error **if documented**.
4. **Reuse scan** → already exists / adapt / new.
5. Collect review cards for gaps/conflicts/ambiguity.
6. Write the chat report. Stop (unless brief-change override).

## Output Format

Stacked lists. **Do not use markdown tables** in the report body.

```markdown
## Контекст
- change: …
- Parent/Child: …

## Контракт (как client вызывает)
- messages / HTTP — зачем UI вызывает
- ключевые request / response / synced fields — что приходит на экран

## Интерфейс
- экраны / панели / модалки
- таблицы / формы / board / HUD
- действия пользователя
- доступы / guards — только если описаны
- пустые / loading / ошибки — только если описаны

## Схема UI
… (дерево в fenced code block — только из источников)

## Поля
- field / лейбл
  - откуда: message / HTTP / schema
  - как выводить / как использовать

## Переиспользование
- Уже есть / as-is
- Доработать
- С нуля

## Расхождения
…

## Пробелы
…

## Неоднозначности
… (если пусто — «не найдено»)
```

### Схема UI

Полное дерево из источников (не урезать, не дорисовывать). Экраны → панели → кнопки/поля. У поля: `лейбл (binding) → как выводить`.

### Карточки

```markdown
### <короткий заголовок>
- **Что не так:** …
- **Предлагаю:** …
- **Ревью**
  - файл: `openspec/changes/<name>/…`
  - строки: Lnn
  - сообщение: <paste-ready без markdown-ссылок>
```

**Предлагаю** — рекомендация ревью / FE-default; не подменять постановку в **Схеме UI** / **Полях**.

## Не делать

- Не plan файлов, не apply runtime, не commit.
- Не invent UI в схеме.
- Не путать с Visual Spec measurement (`prepare-mock`) или pixel verify.
