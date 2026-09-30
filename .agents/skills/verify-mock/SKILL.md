---
name: verify-mock
description: >-
  Compares client UI implementation against an attached or referenced design
  mock (screenshot/image): spacing, colors, typography/copy, icons, dividers,
  button chrome. Reports Violations / Warnings / Recommendations. Use when the
  user asks to verify-mock, check UI vs mock/макет/screenshot, or when
  implement-change runs the mock phase after align-code.
---

# Verify Mock — UI vs attached design mock

Сверить **реализованный client UI** с **прикреплённым или явно указанным макетом** (скрин / изображение). Фокус: отступы, цвета, тексты/copy, иконки, разделители, кнопки, иерархия блоков. Не подменяет `align-code` (постановка/skills) и не подменяет `client-verify-code` (skill compliance).

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
| **Отступы / ритм** | padding карточки, gap между rows, высота actions, alignment label↔count |
| **Типографика** | вес title, размер stats, truncate |
| **Иконки** | leading total icon, action icons, badge icons (если на макете и в scope) |
| **Интерактив chrome** | outline buttons full-width; hover: только если макет/design требуют (часто **без** scale) |
| **Разделители** | линии между секциями; цвет/контраст в обеих темах |

Сверять с **макетом как primary visual evidence**. OpenSpec/design — для разрешённых отклонений (out of scope на макете → не Violation). Conflict макет vs design Decision → **Warning** + указать оба; не «чинить» в сторону макета молча, если design явно запрещает (напр. no hover enlarge).

## Severity (ровно три уровня)

Каждый finding — ровно в один tier. Не замалчивать soft; не раздувать в Violations.

| Tier | Когда | Примеры |
|------|--------|---------|
| **Violations** | Доказуемое расхождение с макетом на in-scope surface: отсутствует обязательный элемент макета; неверный продуктовый текст; цвета dots/theme ломают читаемость относительно макета; нет обязательных divider; кнопка с длинным copy вместо короткого с макета | нет `#` в title; mono dots вместо green/amber/red; «Снять с публикации» на card вместо «Снять»; отсутствует total-row icon когда макет показывает |
| **Warnings** | Сильный lean без полной proof; макет неоднозначен (сжатый скрин); light ок / dark сомнительно; spacing «похоже, но не точно»; макет vs design tension | hover lift на макете, design forbid scale; badge icon на макете, design «icons later» |
| **Recommendations** | Опциональный polish; будущие asset names; микро-kerning; docs/skills sync после fixes | заменить Material placeholder на `task-set-card-tasks.svg` later; подтянуть shadow 2px |

**Не** Violation:

- Out of scope в proposal/design (badge SVG later, catalog unchanged)
- Server / ACL / payload
- Пиксель-perfect до 1px без измеримого макета — максимум Recommendation / Warning

## Hard Boundary (standalone)

- Не править, не создавать, не удалять runtime/meta без явной просьбы после отчёта.
- Не коммитить / не пушить.
- В `implement-change` boundary снимается **только** override-промптом после отчёта (как у align/check).

## Workflow

```
Verify-Mock Progress:
- [ ] 1. Resolve paths + OpenSpec change
- [ ] 2. Resolve mock image(s) — or SKIPPED
- [ ] 3. Map mock regions → client files / i18n
- [ ] 4. Diff structure / copy / color / spacing / icons / dividers / actions
- [ ] 5. Report (three tiers)
```

## Report

Язык: **русский** для prose findings; пути и символы — as-is. Tier-заголовки **как у verify** (English): Violations / Warnings / Recommendations.

```markdown
# Verify Mock — отчёт

Using OpenSpec change: <name | none>
Using mock: <path / light+dark>
Scope surfaces: <list>
Client files reviewed: <paths>

## Сводка
| Violations | Warnings | Recommendations | Verdict |
|------------|----------|-----------------|---------|
| N          | N        | N               | FAIL если Violations>0 иначе PASS |

### Violations
- [violation] <что на макете> → <что в коде> · <file:symbol> · fix: <минимально>

### Warnings
- [warning] <…> · evidence · what would promote to Violation

### Recommendations
- [recommendation] <optional polish>

## Что делать дальше
- Violations сначала; затем Warnings; Recommendations последними
```

Пустые секции: одна строка `- none` (не смешивать items и `none`).

При `SKIPPED: no mock` — короткий отчёт без выдуманных findings.

## Не делать

- Не подменять align/verify skills.
- Не требовать pixel-perfect без основания.
- Не расширять на поверхности вне макета/Scope.
- Не считать отсутствие будущих SVG Violation, если design отложил asset.
