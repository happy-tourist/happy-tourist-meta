---
name: align-implementability-front
description: >-
  Use when judging whether happy-tourist-meta client/FE claims are implementable
  (possibility to build only — not doc quality, not that server already shipped).
  Against current happy-tourist.github.io (+ FE-facing room/HTTP shapes via
  projects-map). For mock-driven UI also audits Visual Spec completeness
  (sizes, padding, colors hex/rgba, typography, alignment, dividers, icons,
  hover) so FE can implement without inventing tokens (`[visual-spec-incomplete]`).
  Treat missing server sibling changes as expected lag if meta may add them;
  warn when FE-consumable server/HTTP/room claims lack UI (`[be-without-fe]`).
  UI with no server/API in meta (`[fe-only-without-be]`) →
  align-implementability-back. Verdict FEASIBLE|PARTIAL|INFEASIBLE|UNKNOWN.
  Finding card: file + lines + issue + fix. Not server audit
  (align-implementability-back), meta quality (align-meta), or pixel verify
  of shipped UI (verify-mock). No file edits.
---

# Align Implementability (Frontend / Client)

Perform a read-only **client implementability** audit: can the **happy-tourist-meta** change be built **as written** against **happy-tourist.github.io** (and FE-facing contract shapes)? Report findings in Russian.

**Scope of judgment — only possibility to implement.** Answers «можно ли реализовать client по этим claims», not «идеальны ли docs», not «уже ли лежит готовый server в siblings», not «совпадает ли уже написанный UI с макетом» (`verify-mock`). Completeness/quality of meta prose → **`align-meta`**. Deep server/DB/room realizability and UI→server gaps (`[fe-only-without-be]`) → **`align-implementability-back`**. Снять токены с картинки → **`prepare-mock`** (этот скилл только **проверяет**, хватает ли уже записанного канона).

Output **one scope** with **three tiers** (hard → warnings → recommendations):

1. **Client implementability** — claims vs sibling client evidence (`aligns` / `extends` / `conflicts` / `unproven`).
2. **Server→Client coverage in meta** — FE-consumable server/HTTP/room claims without UI (`[be-without-fe]`).
3. **Mock / Visual Spec completeness** — when the change is mock-driven (or claims pixel-faithful UI): are **all** measurable tokens present so FE can implement without inventing sizes/paddings/colors/… (`[visual-spec-incomplete]`).

Every finding is a **card**: file link + line numbers + what is wrong + fix proposal (or «неясно как делать»).

## Server lag and BE-without-FE (обязательно)

При проверке клиента **учитывай, что нужных изменений на сервере в sibling-коде ещё может не быть** — они часто появятся в параллельном/следующем server change. Это **не** hard `INFEASIBLE` само по себе.

Проверяй направление **meta server/API → meta UI/client** в change set (+ Parent/Child):

| Ситуация | Как судить |
|----------|------------|
| В UI/client docs есть поле/действие/message, которого нет в текущем server HEAD, но в meta есть (или ожидается) server claim в этом или связанном change | **Не** hard gap. Client: **extends** / `unproven`; при необходимости Warning `[be-not-yet-in-siblings]` + pointer на `align-implementability-back` |
| В UI описаны данные, а в meta **нет** server/HTTP/room/schema claim | **Не** владелец этой метки → pointer: прогон **`align-implementability-back`** (`[fe-only-without-be]`) |
| В server/HTTP/room analytics есть **FE-потребляемый** контракт (synced fields, private messages, HTTP list filters, create options, role gates для кнопок), а в UI/client docs **нет** экрана/контрола/binding | **Warning** `[be-without-fe]` — риск «контракт есть, UI не спроектирован». **Не** hard |
| UI утверждает конкретный path/message/DTO, а **доказанный** client/server binding уже противоречит и meta не планирует сдвиг | Hard `conflicts` / `code-only` по FE-binding |

**Что считать FE-потребляемым для `[be-without-fe]`:** room messages client sends/handles, synced schema fields for HUD/board, HTTP list/filter/DTO fields operator must see/enter, create options (`mapId`/`packId`/`taskSetIds`/…), auth/`isStaff` gates for UI buttons, private payloads (`budgets`, warnings).

**Не** тащить в `[be-without-fe]`: Drizzle internals, room lifecycle hooks without UI surface, pure server error-code tables — это `align-implementability-back`. Не сравнивай sibling server-код с sibling client-кодом «уже реализовано ли» — только **meta claims**.

Не требуй, чтобы server sibling уже содержал новый контракт, прежде чем признать client **FEASIBLE**/**extends**. Hard только при противоречии доказанному client/FE-facing контракту или невозможности встроить UI в существующие client-паттерны **как написано**.

## Runtime roots

- **Claims subject:** changed / relevant `happy-tourist-meta` artifacts that assert **client** runtime: pages, routes, components, Pinia stores, Quasar UX, i18n, permissions on UI, Colyseus client I/O (paths/messages/DTO as client must call them), **and** (when mock-driven) Visual Spec / mock-linked design tokens. For `[be-without-fe]` also **read** FE-consumable server claims in the same change family as *supply* signals.
- **Evidence code:** resolve via [`docs/projects-map.md`](../../../docs/projects-map.md) (+ local YAML). Primary: `client` → `../happy-tourist.github.io/`. Skills: `.agents/skills/client/`. Secondary for FE contract fit only: message/HTTP shapes the client must bind — do **not** deep-audit Drizzle/room internals here (pointer to `align-implementability-back`).
- **Visual Spec evidence:** `openspec/changes/<name>/design.md` section **`## Visual Spec (from mock)`** (или эквивалент); linked mock assets near the change. Канон чеклиста — [`prepare-mock`](../prepare-mock/SKILL.md).
- Do **not** treat client working tree as the primary diff to “fix”.

## Scope filter (claims)

**Include** atomic claims about:

1. Route / page / component / modal / HUD / board placement;
2. UI flows, empty/error states, tooltips, i18n keys when asserted as product behavior;
3. Client form validation rules stated for the SPA;
4. Client use of auth/roles (`requiresAuth`/`requiresStaff`/…) as displayed in UI;
5. Room message / HTTP path / DTO **as client must call/bind** when UI/API docs assert them for frontend work;
6. Client skill procedures (directories under `happy-tourist.github.io`, conventions);
7. For coverage scan only: FE-consumable server claims that imply a UI surface (`[be-without-fe]`);
8. **Mock-driven visual tokens** — sizes, padding/gap/margin, radii, colors (hex/rgba), typography, text alignment, dividers, icons, hover/focus — when the change is mock-driven or claims pixel-faithful UI.

**Exclude** (defer to `align-implementability-back` with at most a one-line pointer):

- Room lifecycle internals, Drizzle, server-only algorithms without FE consumption claim;
- `[fe-only-without-be]` — owned by back skill.

If the branch is server-only and has no client claims, verdict **`UNKNOWN`** with note to run `align-implementability-back` — do not invent UI surfaces; не ставь массовый `[be-without-fe]`; **не** гоняй Visual Spec checklist.

## Required Axis

1. **Client codebase fit / implementability** — client claims vs strong evidence in `happy-tourist.github.io` (+ FE-facing contract shapes).
2. **Server→Client meta coverage** — scan FE-consumable server claims for missing UI (`[be-without-fe]`); skip mass scan when branch is clearly server-only without UI docs/children.
3. **Mock / Visual Spec completeness** — when mock-driven: checklist that FE can implement layout **без изобретения** размеров/отступов/цветов (см. ниже). Skip when change has no mock and no pixel-faithful UI claim.

Optionally note meta-internal defects only as a one-line pointer to `align-meta`. UI→server gaps → one-line pointer to `align-implementability-back`. Missing tokens that should be *extracted from a mock* → pointer to **`prepare-mock`**, not invent numbers here.

## Hard Boundary

Do not edit, create, delete, format, or commit files. Do not apply fixes from align mode. Deliver the complete result in chat. Verdict is evidence only (`править не надо`). **Не** снимать пиксели с макета и **не** дописывать Visual Spec в этом скилле — только отчёт о пробелах.

## Input And Store

Accept pasted requirements and an optional OpenSpec change name. If the branch clearly targets an OpenSpec change and no extra source is given, audit from the change artifacts + client evidence; ask only when the goal cannot be inferred.

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

Union is the claim file set. Narrow only when the user explicitly scopes. Then **filter** to client-relevant claims.

После фильтра — быстрый проход **server → UI** по тому же change set (+ Parent/Child):

1. **Server → UI (обязательно):** для каждого **FE-потребляемого** server claim есть ли UI/client claim? Нет → Warning `[be-without-fe]`. Не изобретай экраны, если ветка явно server-only без client children.
2. **UI → server:** полный audit **не** здесь. Если заметно, что UI описывает данные без API в meta — one-line pointer на `align-implementability-back` (`[fe-only-without-be]`). Есть server в meta, нет в sibling HEAD → Warning `[be-not-yet-in-siblings]`, не hard.
3. **Mock / Visual Spec (обязательно, если mock-driven):** см. ось ниже. Иначе одна строка в отчёте: `Visual Spec: n/a (не mock-driven)`.

## Когда change mock-driven (триггер оси 3)

Считай change **mock-driven** (ось Visual Spec **обязательна**), если верно **хотя бы одно**:

- в change есть / ссылается макет (png/jpg/webp/svg в `openspec/changes/<name>/`, assets, вложение, явный путь в proposal/design);
- в `design.md` есть / обещана секция **Visual Spec** / «from mock»;
- proposal/tasks/specs требуют pixel-faithful / «по макету» / «как на скрине» для UI surface;
- пользователь в запросе указал макет или Visual Spec как источник FE.

Не mock-driven → не раздувай чеклист Visual Spec; не ставь массовый `[visual-spec-incomplete]`.

## Load Sibling Evidence (client)

For each atomic client claim:

1. Resolve `client` via projects-map (+ local YAML).
2. Grep/read Vue/TS pages, components, stores, router, i18n, tests, Colyseus client usage that prove or refute the claim.
3. Prefer current HEAD of the sibling clone; if missing, **Warning** (`sibling-missing`), not a hard gap.

Do not invent sibling screens/APIs. Absence of a clone ≠ proof that meta is wrong. Before deep reads — `{client}/AGENTS.md`.

Для оси Visual Spec sibling-код **не** primary evidence: канон — `design.md` Visual Spec (+ наличие макета). Sibling смотри только чтобы понять, есть ли уже токены/Quasar vars как аналог **extends** (не заменяет отсутствующий Spec).

## Axis — Client implementability judgment

Compare independently per claim:

1. **UI surface** — page/component/modal/HUD exists or clear analogue for **extends**;
2. **Placement / composition** — claimed insertion point matches real App/Lobby/Game/content layout;
3. **Permissions UX** — route guards / role checks as client already does for analogues;
4. **API / room binding** — message names, HTTP paths, DTO shapes client must use match conventions when the change asserts FE contract;
5. **Forms / validation** — stated FE rules sit on existing Quasar/`q-form` patterns;
6. **Skill procedure** — client directories and conventions under `.agents/skills/client/`;
7. **Edge UX** — empty, error, disabled, loading when the change claims completeness.

Classify each claim: **aligns** | **extends** | **conflicts** | **unproven**.

Hard `gap` only with concrete client (or FE-facing contract) evidence + meta misstatement + required parity.

**Do not** treat «в текущем server sibling ещё нет поля/метода» as hard `gap` / `INFEASIBLE` — см. **Server lag and BE-without-FE**.

## Axis — Mock / Visual Spec completeness (обязательно если mock-driven)

Цель: ответить «можно ли сверстать UI **по записанному канону** без угадывания px/цветов». Не сверять уже написанный код с макетом (`verify-mock`). Не invent недостающие числа.

**Где искать канон:** `design.md` → `## Visual Spec (from mock)` (и подсекции). Specs/proposal могут дублировать поведение/copy, но **числа и hex = design**. Если Visual Spec пуст/отсутствует при mock-driven → Hard `[visual-spec-incomplete]` + предложение прогнать **`prepare-mock`**.

Пройди чеклист (канон детализации — [`prepare-mock`](../prepare-mock/SKILL.md) A–F). Для **каждого** видимого на макете/описанного в UI surface элемента отметь: есть измеримый токен | отсутствует | явно «не на макете / Open Question».

### Чеклист обязательных токенов

| Блок | Что MUST быть в Visual Spec (или явный `n/a` + почему) | Нет → |
|------|--------------------------------------------------------|-------|
| **Структура** | Порядок блоков сверху вниз; все dividers (в т.ч. бледные); состояния (badge/status variants) | Hard `[visual-spec-incomplete]` |
| **Copy** | Точные строки / i18n intent для видимых labels/buttons (хотя бы в specs или Visual Spec) | Hard если UI claim «по макету» без текста; иначе Warning |
| **Размеры overall** | Card/surface W×H (или min-height), border-radius, border-width | Hard |
| **Размеры элементов** | W/H (или min-H) **каждого** видимого блока: badge, title, rows, buttons, icons, dots, dividers | Hard на пропущенный видимый элемент |
| **Отступы / ритм** | padding (t/r/b/l) корня; gap/margin **между** соседними блоками (по парам, если ритм разный) | Hard |
| **Кнопки** | height (+ pad); full-width vs content; vs Quasar default/dense | Hard |
| **Типографика** | per-role: `font-size` (+ weight, line-height) — title / badge / stats / button отдельно | Hard |
| **Выравнивание** | per-element `left`/`center`/`right`; shared edges labels/counts если колонки | Hard если колонки/карточка; иначе Warning |
| **Цвета** | hex/rgba (или `currentColor→token`) light **и** dark для: bg, border, fg, badge bg+fg+icon ink, dots, divider, action outline/label/icon, muted opacity | Hard если только «grey/amber» или нет dark при заявленном dark | 
| **Icon ink / sizes** | size CSS px + ink token на каждую иконку; таблица missing icons (или `- none`) | Hard на отсутствующий size/ink видимой иконки; missing asset с Material placeholder → Warning |
| **Hover / focus** | таблица default→hover (border/shadow/bg/scale) light+dark **или** явная строка «hover не показан на макете» | Warning если секции нет; Hard только если proposal/tasks требуют hover chrome, а токенов нет |
| **Неоднозначно** | список Open Questions / «не на макете» — не маскировать под MUST числа | Recommendation доуточнить; не invent |

Правила суждения:

1. `approx` + диапазон (напр. `28–32`) — **достаточно** для FEASIBLE по этому токену.
2. «как в Quasar по умолчанию» **без** чисел — **не** закрывает чеклист, если макет/pixel-faithful claim есть → Hard/Warning `[visual-spec-incomplete]`.
3. Один пропущенный критичный блок (overall size **или** pads **или** цвета **или** typography ролей) → Hard; несколько soft-пробелов (только hover absent без claim) → Warnings.
4. Не требуй pixel-sample confidence `high` — `med`/`approx` OK. Требуй **наличие** токена, не пересъёмку.
5. Предложение в карточке: «дописать в Visual Spec …» или «прогнать `prepare-mock`» — **не** выдумывать конкретные px/hex в этом скилле.

Score **Код проекта: N/10** only from hard `gap` / `conflicts` / required `code-only` **и** hard `[visual-spec-incomplete]` (когда mock-driven). Warnings `[be-without-fe]`, `[be-not-yet-in-siblings]`, soft Visual Spec gaps **не** снижают score. `[fe-only-without-be]` — зона back-скилла.

### Суждение о реализуемости

| Вердикт | Когда |
|---------|--------|
| `FEASIBLE` | No hard `conflicts`; claims **aligns** or **extends**; mock-driven Visual Spec закрывает обязательный чеклист (или n/a); remaining risks are Warnings only (в т.ч. server lag, `[be-without-fe]`, hover absent) |
| `PARTIAL` | Mixed aligns/extends vs hard gap/conflicts, critical `unproven`, **или** mock-driven но часть обязательных Visual Spec токенов отсутствует |
| `INFEASIBLE` | Hard **conflicts** / required `gap` — **не** из‑за отсутствия ещё не смерженного server и **не** из‑за одного только `[be-without-fe]`. Также если mock-driven **и** Visual Spec отсутствует/пуст при обязательном pixel-faithful UI (FE вынужден invent весь chrome) |
| `UNKNOWN` | Not enough client sibling evidence — do **not** invent screens; list what was searched. Отсутствие clone **не** оправдывает пропуск Visual Spec checklist |

`UNKNOWN` alone → Warning, not a score hit.

## Severity

| Tier | When | Align cues |
|------|------|------------|
| **Hard gaps** | Required FE parity missing; claim conflicts proven sibling evidence; mock-driven без критичных Visual Spec токенов (sizes/pads/colors/typography/structure) | `gap`, `conflicts`, `code-only` (when parity required); `[visual-spec-incomplete]` |
| **Warnings** | Strong lean without full proof; sibling missing; critical `unproven`; server lag; FE-consumable server without UI; soft Visual Spec gaps (hover absent, Material placeholder icons) | `[warning]`; `unproven`; `[be-without-fe]`; `[be-not-yet-in-siblings]`; soft `[visual-spec-incomplete]` |
| **Recommendations** | Optional polish vs analogue; soft wording; Open Questions на макете | `[recommendation]` |

Examples:

- UI doc claims new Lobby control with clear `LobbyPage` extension → **extends**.
- Client docs send message `foo` but proven store/room uses `bar` and meta does not plan rename → hard **conflicts**.
- UI показывает поле без server claim в meta → pointer на `align-implementability-back` / `[fe-only-without-be]`.
- Server analytics описывает synced field / filter, а UI docs не показывают контрол → **Warning** `[be-without-fe]`.
- Нужный message ещё нет в server HEAD, но design/spec планирует его → **Warning** `[be-not-yet-in-siblings]`, client всё ещё может быть **FEASIBLE**/**extends**.
- `happy-tourist.github.io` not cloned → **Warning** `[sibling-missing]`.
- Mock-driven change, Visual Spec есть, но нет pad/gap между badge→title и нет hex для `divider` → Hard `[visual-spec-incomplete]`.
- Цвета только «grey / amber» без hex/rgba при макете → Hard `[visual-spec-incomplete]`.
- Hover секции нет, на макете только покой, tasks не требуют hover → **Warning** или отметить «hover не показан»; не Hard.
- Visual Spec отсутствует, proposal говорит «по макету» → Hard `[visual-spec-incomplete]` → `prepare-mock`.
- Drizzle migration missing → **not this skill** → `align-implementability-back`.
- Broken link in proposal → **not this skill** → `align-meta`.
- Код уже сверстан не в те px → **not this skill** → `verify-mock`.

## Finding card (обязательный формат)

```markdown
### <N>. <короткий заголовок>
- Метка: `[gap|code-only|warning|recommendation|be-without-fe|be-not-yet-in-siblings|visual-spec-incomplete]` (+ обязательно `aligns|extends|conflicts|unproven` для runtime-claim; для Visual Spec — `unproven` если токена нет)
- Файл: [`path/from/repo/root.ext`](path/from/repo/root.ext) — **L12–L18** (или **L12, L45**)
- Связанные (если нужно): [`../happy-tourist.github.io/...`](../happy-tourist.github.io/...) — **L3–L5**
- Что не так: <факт: meta / sibling / Visual Spec пробел / противоречие>
- Предложение: <конкретная правка meta-контракта, дописать Visual Spec токен, или `prepare-mock`>  
  **или** Неясно как делать: <варианты A/B без выбора канона>
```

Пути — от корня `happy-tourist-meta` или sibling (`../happy-tourist.github.io/...`, при необходимости `../happy-tourist-server/...`).

## Report

Write in Russian.

```markdown
## Вердикт
- Код проекта (client): N/10 — <основание; только hard; server lag / be-without-fe / soft visual gaps не снижают>
- Реализуемость client (только возможность): FEASIBLE | PARTIAL | INFEASIBLE | UNKNOWN — <кратко; отсутствие server в siblings ≠ INFEASIBLE; пустой Visual Spec при mock-driven может → PARTIAL/INFEASIBLE>
- Режим: docs/openspec | skills | mixed | client-only
- Источник claims: <OpenSpec change / docs / skill>
- Работа в ветке meta: <staged / unstaged / untracked / commits>
- Предыдущий шаг: при необходимости сначала `align-meta`
- Server→Client в meta: перечислить `[be-without-fe]`; UI→server (`[fe-only-without-be]`) → `align-implementability-back`
- Visual Spec (mock): n/a | complete | incomplete — <кратко; список missing блоков чеклиста>

## Claims (meta, client)
- `path` — <какие client runtime-утверждения проверяли>

## Доказательная база (client sibling)
- `sibling-path` — <что подтвердили / не нашли / clone missing>

## Visual Spec / макет
- Mock-driven: yes | no
- Канон: `design.md` Visual Spec | отсутствует | частичный
- Чеклист: структура | copy | overall | element sizes | pads/gaps | buttons | typography | alignment | colors L+D | icons | hover — ok / missing / n/a
- Missing tokens: <список или «нет»>

## Замечания по возможности реализовать (client)

### Ошибки (hard)
- нет

### Предупреждения
- нет

### Рекомендации
- нет

## Карта аналогов (client)
- `path` — <зачем смотреть>

## Что делать дальше
- <hard → warnings → recommendations; visual-spec gaps → prepare-mock / дописать design; fe-only-without-be → align-implementability-back; meta-quality → align-meta; после apply → verify-mock>
```

Never apply fixes. Карточки — текстом; файлы не менять.

## Not This Skill

| Skill | Difference |
|-------|------------|
| `align-implementability-back` | Server/room/HTTP/DB realizability; owns `[fe-only-without-be]` |
| `align-meta` | Quality of meta artifacts — not sibling realizability / Visual Spec token checklist for FE build |
| `align-code` | Runtime branch vs requirements after code exists |
| `prepare-mock` | **Extracts** sizes/colors from mock into Visual Spec — this skill only **audits** completeness |
| `verify-mock` | Shipped UI vs mock/Visual Spec pixels — not «хватает ли токенов до apply» |
| `openspec-verify-change` | Implementation vs OpenSpec (after code exists) |
| `client-align-code` / `client-verify-code` | Client code quality/compliance, not meta claims vs siblings |
| `check-changes` | Unstaged client/server → skill/docs drift plan |
