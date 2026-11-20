---
name: brief-client-scope
description: >-
  Read-only client/FE scope brief from an OpenSpec change: contract binding
  summary (room/HTTP as client uses them), planned UI, fields/display, reuse vs
  adapt vs new, plus review cards (file+line+quote+proposal+paste-ready message)
  for расхождения/пробелы/неоднозначности. Use when the user asks to brief
  client scope, разбери постановку на фронт/client, what the UI should show, or
  to understand FE work before coding. Supports filters: только change, без
  sibling, остальное готово. Chat report only by default; does not prepare a
  per-file plan (client-locate-change-points) or pixel-verify (verify-mock).
---

# Brief Client Scope

Read-only brief of **what the client should show and how it binds the contract**, grounded in an OpenSpec change (+ Parent/Child), plus a short **reuse / adapt / new** readout vs `happy-tourist.github.io`.

**Home:** `.agents/skills/client/` in **happy-tourist-meta**. Runtime evidence: projects-map key `client`.

## Hard Boundary

- Do **not** edit, create, delete, format, or commit files — except when called from [`review-change`](../../review-change/SKILL.md) (override: after report, apply meta fixes) or the user **explicitly** asks to change named meta files / this skill.
- Deliver the **entire** result **in chat only** (unless override above).
- Do **not** run `client-locate-change-points` or produce a «куда класть файлы» plan.
- Do **not** run `align-implementability-front` / `client-align-code` / `verify-mock` / `prepare-mock`.
- Do **not** suggest follow-up skills at the end of the report (unless the user asked for next step).
- **Allowed for reuse:** light read of `happy-tourist.github.io` (via projects-map) and sibling changes — as analogs, not as an implementation checklist.

**Report language: Russian.** Keep UI labels, message names, store keys, routes as in the sources.

## When To Use

- Clear client picture from a change before implementation / align
- Triggers: brief client scope / разбери постановку на фронт / что показывать в UI / field digest
- After OpenSpec artifacts exist

## Relation To Other Skills

| Skill | Role relative to this |
|-------|------------------------|
| [`brief-server-scope`](../../server/brief-server-scope/SKILL.md) | Server contract digest — pair via `review-change` |
| [`review-change`](../../review-change/SKILL.md) | Orchestrator: brief both sides then align — not this alone |
| `align-implementability-front` | Can we build UI? / Visual Spec completeness — not this |
| `client-locate-change-points` | Where to edit Vue — not this |
| `prepare-mock` / `verify-mock` | Measure / pixel-check mock — not this |
| `client-align-code` | Code vs requirements — not this |

## Input

Resolve in this order (ask only if still nothing resolves):

1. **Explicit in the user message** — OpenSpec change name and/or pasted excerpts.
2. **Current git branch (default)** — if the user did not name a change: take a slug hint from the current branch name in `happy-tourist-meta` (e.g. `feature/add-tourist-move` → prefer matching `openspec/changes/<name>/`):
   - Prefer exact / prefix match of branch kebab segment to a change folder name.
   - If **exactly one** active change exists (`openspec list --json`) and branch is ambiguous — use that change.
   - If **zero or several equally plausible** matches remain — ask; do not guess.
3. Pasted excerpts — only when links/paths are unavailable.

| Input | Meaning |
|-------|---------|
| OpenSpec change name | Primary: `openspec/changes/<name>/` |
| Pasted excerpts | Secondary if needed |

Optional filters (honor when stated):

| Filter | Effect |
|--------|--------|
| «только change» / «только текущий change» | Do not treat sibling changes as in-scope for **Расхождения/Пробелы/Неоднозначности** (siblings OK only for **Переиспользование** analogs unless user forbids) |
| «без sibling» | Do not read sibling OpenSpec changes for issues (reuse scan may still use client runtime if available) |
| «остальное готово» | Assume server/contract already delivered; do not list missing server as gaps — only client/UI issues in the scoped docs |

Default: load the primary change (+ Parent/Child links the change itself points to). Prefer pasted excerpts only when paths are unavailable.

## Source Priority

1. **OpenSpec change** is the only source of truth for planned client behavior (`proposal`, `design`, `specs`, `tasks`, Visual Spec if present, plus **linked Parent/Child** docs in scope).
2. Conflicts **inside** the OpenSpec scope → **Расхождения**.
3. Gaps / ambiguities for review cards → only where the **change tree** is silent or unclear for client.
4. For **Переиспользование**: sibling changes + light client read; do not invent reuse.

Do not invent screens, fields, empty/error, or permissions not in the loaded docs.

## Workflow

1. Resolve change root (`openspec status --change "<name>" --json` when useful).
2. Read client-relevant artifacts: proposal, design (incl. Visual Spec if any), delta specs, tasks. Honor exclusions («не смотри design»).
3. Extract UI scope: pages/routes, components/modals/HUD, actions, bindings to messages/HTTP, permissions, loading/empty/error **if documented**.
4. **Reuse scan** → already exists / adapt / new.
5. Collect review cards for gaps/conflicts/ambiguity **only from the OpenSpec change tree**.
6. Write the chat report. Stop (unless review-change override).

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
… (см. правила ниже)

## Поля
…

## Переиспользование
Обязательный раздел. Три списка (пустой список — одна строка «нет»):

- Уже есть / переиспользовать as-is
  - что / где / зачем
- Доработать (есть база, но не совпадает с постановкой)
  - что / где / разрыв
- Сделать с нуля / нет готового аналога
  - что из постановки ещё не покрыто

## Расхождения
… (карточки)

## Пробелы
…

## Неоднозначности
…
```

### Схема UI

Схематичное дерево экрана в fenced code block (обязательно). **Каждый раз — полная схема**, не урезанная: все экраны/панели/модалки, все кнопки и интерактивы, пустые/loading/ошибки, и **для каждого поля — как выводить**. Только то, что есть в источниках — не дорисовывать.

Формат:

````markdown
```
Родительский экран / панель
└─ под-панель / модалка
   │
   ├─ [кнопка «…»]  (если есть в источниках)
   ├─ [список / таблица / board «…»]
   │    Поле (binding)     → как выводить / формат
   │    …
   │
   └─ [модалка «…»]
        ├─ пустое / loading / ошибка — если описаны
        ├─ !! РАЗВОРОТ МАССИВА: parent[] × child[] → N строк  (если есть в источниках)
        │    parent[i]  → общие поля: только в 1-й строке группы / копировать / иное — как в источниках
        │    └─ child[j] → отдельная строка / слот
        └─ [кнопка «…»]
```
````

Правила схемы:

- вложенность = реальная иерархия UI
- **не сокращать**: каждый раз выводить полный набор из источников
- у каждого поля: `лейбл (binding) → как выводить`
- кнопки и интерактивы в квадратных скобках `[…]`
- **массивы, которые дают несколько UI-строк или вложенный вывод** — подсвечивать отдельно:
  - строка `!! РАЗВОРОТ МАССИВА: A[] × B[] → N строк` (или `!! МАССИВ В ЯЧЕЙКЕ` / `!! ВЛОЖЕННЫЙ СПИСОК`)
  - ниже — дерево `parent[i]` / `child[j]`
  - **`!!` — только при реальном развороте** (parent×child / flatten). Обычный список `items[]` → по одной строке на элемент **не** помечать `!!`
- если кнопок/фильтров нет — явно одной строкой под схемой: «Кнопок / фильтров нет»

## Поля

Для каждого поля (или логической группы), друг под другом:

- `binding` / UI-лейбл (если есть)
  - откуда: message / HTTP / schema
  - как выводить: формат, маппинг, «—», скрытие, …
  - как использовать: колонка, ключ строки, условие показа, …

### Карточки: Расхождения / Пробелы / Неоднозначности

Каждый пункт — **карточка** (как в `align-meta`: файл + строки + суть + предложение). Кратко; с **цитатами**; жирным — на что смотреть.

**Куда что класть:**

| Раздел | Когда |
|--------|--------|
| **Расхождения** | Два (и более) места в **OpenSpec change tree** говорят разное (proposal vs design, Parent vs Child, две секции одного файла) |
| **Пробелы** | Client нужно поведение, а в change tree его нет / не дописано «как» |
| **Неоднозначности** | Текст есть, но два разумных чтения без канона |

```markdown
### <короткий заголовок>
- **Что не так:** …
- **Предлагаю:** …
- **Ревью**
  - файл: `openspec/changes/<name>/…`
  - строки: Lnn
  - сообщение: <paste-ready текст без markdown-ссылок>
```

**Предлагаю** — рекомендация ревью / FE-default; не подменять постановку в **Схеме UI** / **Полях**.
Якорь — одно место правки. В `сообщение`: цитата «здесь» + (если нужно) другая цитата + одна фраза сути. **Запрещено:** «выше»/«ниже», markdown-ссылки.

## Не делать

- Не plan файлов, не apply runtime, не commit.
- Не invent UI в схеме.
- Не путать с Visual Spec measurement (`prepare-mock`) или pixel verify.
