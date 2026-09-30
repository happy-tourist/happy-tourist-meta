---
name: verify-mock
description: >-
  Compares client UI implementation against an attached or referenced design
  mock (screenshot/image): spacing, colors, typography/copy, icons, pale
  dividers between every stats row, column alignment, slim button height,
  vertical fill. Strictly enforces Visual Spec sizes (exact or in-range) as
  Violations. Uses design Visual Spec from prepare-mock when present. Reports
  Violations / Warnings / Recommendations. Use when the user asks to
  verify-mock, check UI vs mock/макет/screenshot, or when implement-change
  runs the mock phase after align-code.
---

# Verify Mock — UI vs attached design mock

Сверить **реализованный client UI** с **прикреплённым или явно указанным макетом** (скрин / изображение). Фокус: отступы, цвета, тексты/copy, иконки, разделители, кнопки, иерархия блоков. Не подменяет `align-code` (постановка/skills) и не подменяет `client-verify-code` (skill compliance). Измеримый канон заранее снимает [`prepare-mock`](../prepare-mock/SKILL.md) в `design.md` → **Visual Spec (from mock)** — при сверке считать его рядом с картинкой.

Скилл **только анализирует и отчитывается** — не правит runtime-код и не коммитит. Правки — только по явной просьбе после отчёта (в `implement-change` после отчёта обязателен override «Добавь и поправь что считаешь нужным»).

## Когда применять

- Пользователь запускает `verify-mock` / просит сверить UI с макетом / скрином
- В пайплайне `implement-change` — фаза сразу **после** align-code
- Change/design ссылается на макет light/dark, а код уже (частично) реализован

Не вызывать «на всякий случай», если нет макета и пользователь не просил визуальную сверку.

## Репозитории и пути

Из корня `happy-tourist-meta` через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ local YAML):

| Ключ | Default | Роль |
|------|---------|------|
| `client` | `../happy-tourist.github.io` | runtime UI для сверки |
| meta | `.` | OpenSpec change, skills |

Перед правками/чтением client — `{client}/AGENTS.md`. Skills стилей: `.agents/skills/client/work-with-styles/` (+ `pack-cards.md` и др. topic при касании).

## Источник макета (обязательно резолвить)

Искать **в порядке**:

1. Изображения, прикреплённые к **текущему** сообщению / явно переданные в prompt субагента (`file_attachments`, пути в `assets/…`).
2. Макеты из **диалога** этой сессии (сохранённые Cursor image paths), если пользователь говорил «этот скрин» / «макет».
3. Ссылки в артефактах **активного OpenSpec change** (`proposal` / `design` / `tasks` → «макет», «скрин») + файлы рядом с change, если есть.
4. Явный путь от пользователя.

**Объявить:** `Using mock: <path-or-description>` (light/dark frames if both).

Если макета **нет** после поиска:

- Standalone: остановиться — «макет не найден; приложите скрин или укажите путь».
- Внутри `implement-change`: вернуть `SKIPPED: no mock` (не `HUMAN_BLOCKER`), кратко почему; не выдумывать пиксели.

Прочитать изображение инструментом Read (vision). Если light **и** dark на одном кадре — сверять оба; findings помечать `light` / `dark` / `both`.

**Уточнение человека по макету** (например «разделители бледные, но есть») — **ground truth** над первой vision-описью; пересмотреть findings, не спорить со скрином словами.

### Anti-miss при чтении макета

Vision и сжатый скрин часто **теряют** бледный chrome. Перед выводом «нет X»:

| Риск | Что делать |
|------|------------|
| **Бледные dividers** | Не заключать «нет линий». Проверить **каждую** пару соседних stats-rows (total↔diff1, diff1↔diff2, …) **и** линию над actions. Zoom/crop карточки; при сомнении — luminance dip / спросить. Low-contrast ≠ absence. |
| **Колоночная сетка** | Lead-колонка = ширина индикатора (напр. 3 dots); icon в total-row **по центру этой ширины**; labels («Заданий:» / «Лёгкие:» / …) — **одна** левая вертикаль; counts — правый край. |
| **Вертикальный fill** | Блоки badge → title → stats → actions распределены по высоте карточки. Опциональный badge: без статуса пусто **только сверху** (зарезервированный top), не схлопывать весь ритм. |
| **Высота кнопки / размеры** | Сверить **каждый** токен из Visual Spec «Размеры элементов» (overall + badge/title/rows/buttons/icons/dots/dividers/pad/gap). Вне диапазона Spec / вне точного значения → **Violation**. Quasar default (~36–40) при Spec ~28–32 — Violation. |
| **Hover enlarge** | На макете часто «придуманный» lift; design может запрещать scale → Warning (tension), не чинить scale молча. |

## Scope реализации

1. Резолв OpenSpec change (как в align-code): имя от пользователя → диалог → единственный active → иначе спросить / `OpenSpec: none`.
2. Из change: surfaces в Scope (напр. live pack + cards editor task-set cards). Без change — поверхности из запроса пользователя.
3. Найти соответствующие Vue/CSS/i18n в client (locate via skills / grep / design Decision).
4. Сверить **код + i18n** с макетом. Опционально: browser screenshot текущей страницы, если окружение доступно и пользователь/пайплайн не запретил — не блокер, если UI не поднят.

Не раздувать scope на server, catalog packs, maps и т.п., если макет про другое.

## Что сверять (чеклист)

Для каждого видимого блока на макете:

| Ось | Примеры |
|-----|---------|
| **Структура** | порядок сверху вниз; наличие/отсутствие строк, бейджа, actions, divider |
| **Copy / тексты** | точные продуктовые строки (решётка `#`, «Заданий:», «Снять» vs длинное); i18n keys vs rendered sense |
| **Цвета** | filled dots, badge tint, card bg/fg light+dark, muted soft-unpublish |
| **Отступы / ритм / размеры** | **Обязательно:** overall W×H + pad/gap + W/H каждого элемента vs Visual Spec (и макет). См. «Размеры — строгая сверка» ниже |
| **Типографика / выравнивание** | **font-size / weight / line-height** per role vs Visual Spec (строго, как размеры); truncate; **text-align** per element; vertical center в ряду |
| **Иконки** | leading total icon (центр над шириной dots), action icons, badge icons (если на макете и в scope) |
| **Интерактив chrome** | outline buttons full-width; slim height vs Quasar default; **hover отдельной сверкой**: border color/width, shadow, bg — vs Visual Spec `### Hover / focus`; scale только если design/mock явно разрешили |
| **Разделители** | бледные линии **между каждыми** stats-rows **и** над actions; контраст light/dark (low-contrast ≠ «нет») |

Сверять с **макетом как primary visual evidence**. Если в active change есть `design.md` → **Visual Spec (from mock)** ([`prepare-mock`](../prepare-mock/SKILL.md)) — сверять код и с ним: расхождение с Visual Spec на in-scope токене → Violation (даже если скрин сжатый). **Размеры из Visual Spec обязательны** (см. ниже). OpenSpec/design Decisions — для разрешённых отклонений (out of scope на макете → не Violation). Conflict макет vs design Decision → **Warning** + указать оба; не «чинить» в сторону макета молча, если design явно запрещает (напр. no hover enlarge).

## Размеры — строгая сверка (обязательно)

`verify-mock` **требует** соответствия размеров Visual Spec / измеримому макету — не «похоже», а попадание в канон.

| Источник канона | Правило |
|-----------------|---------|
| Visual Spec: одно число (напр. `height: 30`) | Код MUST быть **равен** этому значению (CSS computed / declared). Допуск **только** если Spec явно пишет `±N` или `tolerance`. |
| Visual Spec: диапазон / `approx` (напр. `150–160`, `≈28–32`) | Код MUST попадать **внутрь** диапазона inclusive. Вне → Violation. |
| Visual Spec есть, в коде размер отсутствует / другой механизм (default Quasar) | Violation — зафиксировать ожидаемое vs фактическое. |
| Visual Spec **нет** строки на элемент, но элемент на макете мерится | Снять оценку с макета (как prepare-mock), сверить код; вне разумного `approx` с макета → Violation; заодно Recommendation дописать Spec. |
| Ни Spec, ни измеримый макет по размеру | не invent Violation по 1px; Warning «нет канона размера». |

Обязательный проход по таблице Spec «Размеры элементов» (overall + каждый блок) **и** по таблице типографики (`font-size` / weight / line-height per role). В отчёте — секция **Sizes check** (включая `title.font-size`, `stats-label.font-size`, …): `token | expected | actual | pass/fail`.

Любой **fail** по размеру **или font-size/weight/line-height из Spec** → **Violation** (не Warning «типографика похожа»).

## Severity (ровно три уровня)

Каждый finding — ровно в один tier. Не замалчивать soft; не раздувать в Violations.

| Tier | Когда | Примеры |
|------|--------|---------|
| **Violations** | Доказуемое расхождение с макетом / Visual Spec: структура, copy, цвета, dividers, align, icons, actions copy; **любой размер или font-size/weight/line-height вне канона Spec** | нет `#`; button `40px` при Spec `28–32`; title `font-size 16` при Spec `12–13`; pad `16` при Spec `8` |
| **Warnings** | Сильный lean без полной proof; макет неоднозначен **и** нет числа в Spec; light ок / dark сомнительно; макет vs design tension (не размеры при наличии Spec); hover border tension | hover lift vs forbid scale; badge icon later; нет Spec-строки на элемент и скрин мыльный |
| **Recommendations** | docs/skills sync; asset drop-in; дописать недостающую строку в Visual Spec после замера | заменить Material placeholder; добавить token в Spec |

**Не** Violation:

- Out of scope в proposal/design (badge SVG later, catalog unchanged)
- Server / ACL / payload
- Отсутствие hover-scale, если design запретил enlarge (даже если на макете карточка «приподнята»)
- Размеры **требуются**, если канон есть в Visual Spec / измерим на макете (pixel-perfect escape hatch снят)

## Hard Boundary (standalone)

- Не править, не создавать, не удалять runtime/meta без явной просьбы после отчёта.
- Не коммитить / не пушить.
- В `implement-change` boundary снимается **только** override-промптом после отчёта (как у align/check).

## Workflow

```
Verify-Mock Progress:
- [ ] 1. Resolve paths + OpenSpec change
- [ ] 2. Resolve mock image(s) — or SKIPPED
- [ ] 3. Map mock regions → client files / i18n + load Visual Spec sizes
- [ ] 4. Diff structure / copy / color / **sizes (strict)** / icons / dividers / actions / hover
- [ ] 5. Report (three tiers + Sizes check)
```

## Report

Язык: **русский** для prose findings; пути и символы — as-is. Tier-заголовки **как у verify** (English): Violations / Warnings / Recommendations.

```markdown
# Verify Mock — отчёт

Using OpenSpec change: <name | none>
Using mock: <path / light+dark>
Scope surfaces: <list>
Client files reviewed: <paths>
Visual Spec sizes: yes | no

## Сводка
| Violations | Warnings | Recommendations | Verdict |
|------------|----------|-----------------|---------|
| N          | N        | N               | FAIL если Violations>0 иначе PASS |

## Sizes check
| Token | Expected (Spec/mock) | Actual (code) | Result |
|-------|----------------------|---------------|--------|
| card.width | … | … | pass / fail |
| … | | | |

### Violations
- [violation] <что на макете/Spec> → <что в коде> · <file:symbol> · fix: <минимально>

### Warnings
- [warning] <…> · evidence · what would promote to Violation

### Recommendations
- [recommendation] <optional polish / дописать Spec token>

## Что делать дальше
- Violations сначала (в т.ч. size fails); затем Warnings; Recommendations последними
```

Пустые секции: одна строка `- none` (не смешивать items и `none`).

При `SKIPPED: no mock` — короткий отчёт без выдуманных findings.

## Не делать

- Не подменять align/verify skills.
- Не ослаблять размеры до Warning «похоже», если в Visual Spec есть число/диапазон — это **Violation**.
- Не расширять на поверхности вне макета/Scope.
- Не считать отсутствие будущих SVG Violation, если design отложил asset.
- Не писать «нет разделителей», пока не проверены бледные линии между **всеми** stats-rows и над actions (см. Anti-miss).
- Не игнорировать уточнение пользователя по макету в пользу первой vision-описи.
