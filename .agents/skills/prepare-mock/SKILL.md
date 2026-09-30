---
name: prepare-mock
description: >-
  Extracts measurable UI requirements from an attached design mock
  (sizes, padding, button heights, radii, colors, pale dividers between every
  stats row and above actions, column grids, copy, gaps, icons, text alignment,
  hover/focus border-shadow-bg diffs, light/dark, vertical fill). Lists missing
  icons needed for implementation. Produces a structured inventory and folds it
  into OpenSpec design/requirements. Use when the user runs prepare-mock, asks
  to measure a mock/макет/screenshot into specs, or to capture visual tokens
  before propose/apply.
---

# Prepare Mock — снять с макета требования

Из **прикреплённого макета** (скрин / изображение light и/или dark) снять **все видимые UI-требования**: размеры, отступы, высоты кнопок, радиусы, цвета, разделители, тексты, расстояния между элементами, иконки, порядок блоков — и **перенести** их в требования OpenSpec (прежде всего `design.md` Visual Spec; copy/структура — в delta specs при необходимости).

Скилл **не** пишет runtime-код и **не** коммитит. Не подменяет [`verify-mock`](../verify-mock/SKILL.md) (сверка кода с макетом) и не подменяет [`prepare-changes`](../prepare-changes/SKILL.md) (нарезка scope).

Типичная цепочка: **prepare-mock** → `openspec-propose` / `openspec-update-change` (если артефакты уже есть) → apply → … → **verify-mock**.

## Когда применять

- Пользователь запускает `prepare-mock` / просит «снять размеры с макета» / «перенести макет в требования»
- Перед propose или доработкой change, когда есть скрин, а в design ещё нет измеримого Visual Spec
- Когда verify-mock / apply спотыкаются о «макет есть, но в specs нет чисел/copy»

Не вызывать без изображения и без явного пути к макету.

## Репозитории и пути

Из корня `happy-tourist-meta` через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ local YAML):

| Ключ | Default | Роль |
|------|---------|------|
| meta | `.` | OpenSpec change / design / specs |
| `client` | `../happy-tourist.github.io` | опционально: существующие токены/ Quasar vars для сопоставления |

## Источник макета

Тот же порядок, что у [`verify-mock`](../verify-mock/SKILL.md):

1. Вложения текущего сообщения / `file_attachments` / `assets/…`
2. Макеты из диалога сессии
3. Ссылки в active change + файлы рядом с change
4. Явный путь от пользователя

**Объявить:** `Using mock: <path>` (+ light/dark если оба кадра).

Если макета нет — остановиться: «приложите скрин или укажите путь». Не выдумывать пиксели.

Прочитать изображение инструментом Read (vision). Light и dark на одном кадре — извлекать **оба** набора токенов; помечать `light` / `dark` / `shared`.

**Уточнение человека** по макету — ground truth; переснять inventory, не спорить.

### Anti-miss при снятии

| Риск | Что фиксировать в Visual Spec |
|------|-------------------------------|
| **Бледные dividers** | Список **всех** линий: между каждыми соседними stats-rows **и** над actions. Low-contrast / 1px / soft grey — всё равно MUST «есть», плюс цвет light/dark. Не опускать «между уровнями», если на кадре еле видно. |
| **Колонки** | Lead width = ширина 3-dot group; total icon centered in lead; labels одна левая вертикаль; counts right. |
| **Выравнивание текста** | У каждого текстового элемента: `left` / `center` / `right` (+ baseline/vertical center в ряду). Title vs badge vs stats vs button label — **не** считать «как обычно» без взгляда на макет. Общая левая кромка labels; counts на одной правой кромке; badge/title часто center. |
| **Размер шрифтов** | У **каждого** текстового роли: `font-size` (+ weight, line-height). Title / badge / stats label / count / button — отдельные токены в Visual Spec. Approx OK с якорем (ширина карточки); не оставлять «просто меньше». |
| **Vertical fill** | Как блоки делят высоту карточки; reserved top при отсутствии badge. |
| **Button height** | Approx CSS px (якорь: ширина карточки); явно vs Quasar default/dense. |
| **Размеры всех элементов** | Overall card W×H (или min-height) **и** W/H (или min-height) + pad каждого видимого блока (badge, title, rows, buttons, icons, dots, dividers). Не ограничиться одной «шириной карточки». |
| **Hover** | **Отдельная** таблица default vs hover (и selected/focus, если на кадре): какие токены меняются — border color/width, shadow, bg, opacity; что **не** меняется (scale/enlarge часто запрещён). Light и dark отдельно. Не смешивать с обязательным chrome покоя. |

При сомнении — crop/zoom карточки или спросить; не invent «нет линии».

## Резолв OpenSpec change

1. Имя от пользователя.
2. Иначе единственный active (`openspec list --json`) → **Using existing change: \<name\>**.
3. Иначе из контекста диалога.
4. Иначе `OpenSpec: none` — только отчёт extract; предложить propose после согласования.

## Hard Boundary

- Не править client/server runtime.
- Не коммитить / не пушить.
- Писать **только** planning-артефакты change (см. Capture), не invent новые capability folders без update/propose workflow.
- Не создавать change directory вручную — только через `openspec new change` если пользователь явно просит новый change.

## Что снимать (чеклист извлечения)

Пройти макет **сверху вниз** по каждой карточке/секции. Для каждого элемента зафиксировать:

### A. Структура и copy

| Поле | Пример |
|------|--------|
| Порядок блоков | badge → title → total → **divider** → diff1 → **divider** → diff2 → **divider** → diff3 → **divider** → actions (перечислить **каждую** линию) |
| Точный текст | `Набор заданий #1`, `Заданий:`, `Лёгкие:`, `Снять`, `Вернуть` |
| Варианты состояний | ДОРАБОТАТЬ / НА ПРОВЕРКЕ / СНЯТО (+ иконка бейджа если видна) |
| Truncate / max lines | title 1–2 строки |
| **Типографика** | per-element: `font-size` (px/`rem`/`em`), `font-weight`, `line-height`; badge vs title vs stats vs button — **отдельные** строки |
| **Выравнивание текста** | per-element: title `center`? badge `center`? stats labels `left` + shared edge; counts `right` + shared edge; button label `center` (icon+text); multi-line title — center each line |
| Колонки stats | lead (dots width) \| label left-edge shared \| count right |
| Vertical fill | блоки тянут высоту; без badge — пусто только top |

### B. Геометрия (px или относительные оценки)

Если скрин без линейки — давать **оценку в CSS px** с пометкой `approx` и якорем (напр. «ширина карточки ≈ 150–160 относительно соседних»). Лучше диапазон, чем выдуманная точность 1px.

**Обязательно снять:**

1. **Корень / карточка (overall):** width, height или min-height, border-radius, border-width.
2. **Каждый видимый блок/элемент** (badge, title, total-row, каждая difficulty-row, каждый divider, actions zone, каждая кнопка, иконки, dots): **width и/или height** (или min-height), плюс padding/margin/gap где видно.
3. **Расстояния между** соседними элементами (не только «gap rows» одной цифрой — если ритм разный, по парам).

| Поле | Пример |
|------|--------|
| Card width / height (or min-height) / radius | 156×228 (or min-height 228), `border-radius: 12` |
| Card padding (top/right/bottom/left) | status `4–6` top; body `6–8` x; actions pad `…` |
| Gap / margins между блоками | badge→title `…`; title→total `…`; row↔divider `…` |
| Badge size | height ≈ `…`; pad; radius (pill) |
| Title block | **font-size** / weight / line-height; max height (2 lines) |
| Total-row / each stats-row | row height ≈ `…`; **label + count font-size**/weight; icon size; count width |
| Button | **full width** of content; **height** ≈ `28–32`; **label font-size**/weight; pad |
| Dot size / gap | `5–6` diameter, gap `2` |
| Lead column width | ≈ width of 3 dots (+ gaps); icon centered in it |
| Icon size (total / action / badge) | ≈ `14–16` |
| Divider thickness / inset / **где** | `1px` pale; **между всеми** stats-rows + над actions; full content width |

В Report — таблица **«Размеры элементов»**: одна строка на элемент (`token | w | h | pad | notes | confidence`). Overall card — первая строка. Пустых «забыли кнопку» не оставлять: если на макете видно — строка обязательна.

Не путать: **overall** карточки ≠ сумма children без учёта padding/gap — при расхождении пометить `approx` / Open Question.

### C. Цвета (hex / rgba / product sense)

Для **light и dark** отдельно, где отличаются:

| Поле | Пример |
|------|--------|
| Card bg / border / title / muted | `#fff` / `#2a2a2a`, border soft |
| Dot filled 1/2/3 | green / amber / red |
| Dot empty | outline ring, не solid muted |
| Badge bg/fg per status | grey / yellow tint «НА ПРОВЕРКЕ» |
| Divider color | **low-contrast** line (light: soft grey on white; dark: soft grey on `#2a2a2a`) — всё равно фиксировать |
| Button outline / icon / label | Quasar outline sense; tint warning/primary vs нейтральный mock — явно |
| Muted / soft-unpublished overlay | сниженная opacity / darker card |

### D. Иконки

Для **каждой** иконки на макете заполнить строку:

| Место | Описание на макете | Нужен custom asset? | Reserved asset name | Временный Material | Есть в client? |
|-------|--------------------|---------------------|---------------------|--------------------|----------------|
| Total row | document/papers | да (часто) | `task-set-card-tasks.svg` | `description` | проверить `src/assets/**` |
| Edit / unpublish / … | pencil / eye-off | нет, если ок Material | — | `edit` / `visibility_off` | Quasar Material set |
| Badges | X / clock / eye-off | да, если макет ≠ Material | `task-set-badge-*.svg` | `close` / `schedule` / `visibility_off` | проверить assets |

**Сверка «есть в client?»** (read-only):

1. Список иконок с макета (место + смысл).
2. Поиск custom файлов: `{client}/src/assets/**` (и смежные static), имя ≈ reserved / описание.
3. Если на apply достаточно **стандартного Quasar/Material** имени и макет не требует уникальной формы — считать **закрытым временно** (не «missing asset»), но отметить `placeholder: Material <name>`.
4. Если макет явно кастомный / user сказал «добавлю в ассеты» / reserved SVG ещё нет в репо → **missing**.

### D2. Недостающие иконки (обязательный отдельный список)

После таблицы D вывести **отдельную** секцию отчёта и при Capture — в `design.md` Visual Spec:

```markdown
## Недостающие иконки (для реализации)
| ID | Место на макете | Зачем | Suggested file | Temp Material | Блокер apply? |
|----|-----------------|-------|----------------|---------------|---------------|
| I1 | total-row | document icon | task-set-card-tasks.svg | description | нет (placeholder OK) |
| I2 | badge СНЯТО | eye-off custom | task-set-badge-unpublished.svg | visibility_off | нет / да* |
```

Правила:

- Список **всегда** присутствует в Report: если всё закрыто Material/файлами — одна строка `- none`.
- `Блокер apply?` = **да** только если без точного asset нельзя удовлетворить MUST (редко); обычно **нет** + placeholder.
- Не смешивать missing icons с «Неоднозначно / не на макете».
- Не дублировать один и тот же glyph дважды (один ID на роль).

### E. Интерактив — состояние наведения (обязательно отдельным блоком)

Если на макете есть карточка/кнопка «в покое» и «при наведении» (или selected/focus) — **не** сводить к одной фразе «есть hover». Заполнить отдельную таблицу **default → hover** (и при наличии → selected/focus).

| Поле | Что фиксировать | Пример |
|------|-----------------|--------|
| Target | card body / action button / badge | card |
| Theme | light / dark | both |
| Border | цвет покоя → цвет hover; толщина; inset ring? | soft grey → white/secondary; +1px ring |
| Shadow | none → elevation / blur / spread | soft lift shadow **без** scale |
| Background | меняется ли bg / tint | no / slight brighter |
| Opacity | card или chrome | unchanged |
| Scale / translate | enlarge? lift px? | **no scale**; optional translateY approx |
| Cursor / other | если видно | pointer |
| Что **не** меняется | явно | title color, dots, dividers unchanged |
| На кадре видно? | yes / inferred / absent | если absent — написать «hover не показан на макете» |

Правила:

- Секция **«Состояние наведения»** всегда в Report: либо таблица diff, либо `- hover не показан на макете`.
- Light и dark — отдельные строки, если отличаются.
- Scale/enlarge vs только border/shadow — **разные** решения; не путать.
- Disabled / muted card — отдельными строками (не путать с hover).
- При Capture → `design.md` Visual Spec подсекция **`### Hover / focus`**.

### F. Не на макете / неоднозначно

Отдельным списком: что **нельзя** вывести (напр. exact typeface, motion timing) → не писать в MUST specs как числа; оставить Recommendation или Open Question.

## Workflow

```
Prepare-Mock Progress:
- [ ] 1. Resolve mock (+ light/dark)
- [ ] 2. Resolve OpenSpec change (or none)
- [ ] 3. Vision extract → inventory (A–F) + missing icons (D2) + hover table (E)
- [ ] 4. Show extract report to user (incl. **Недостающие иконки** + **Состояние наведения**)
- [ ] 5. Capture into requirements (design ± specs)
- [ ] 6. Point next step (propose / update / verify-mock later; asset drop-in)
```

### 5. Capture into requirements

После отчёта — **перенести** inventory в артефакты:

| Куда | Что |
|------|-----|
| `design.md` | Секция **`## Visual Spec (from mock)`** (или обновить существующую): токены A–E, light/dark таблицы, approx пометки, asset names, **`### Missing icons`** (= D2), **`### Hover / focus`** (= E). Decisions могут ссылаться сюда. |
| `proposal.md` | Кратко в What Changes / Impact, если появились новые user-visible copy/chrome; Out of scope для неоднозначного |
| `specs/<capability>/spec.md` | **Поведение и copy**, не CSS-классы и не сырые hex как «implementation constants»: структура карточки, обязательные строки, цвет **как product sense** (green/amber/red), наличие divider/icon, short action labels. Номера SC — расширить/добавить сценарии при новых MUST. |
| `tasks.md` | При необходимости новые `- [ ]` на перенос токенов в код; уже `[x]` не снимать без причины |

Правила OpenSpec:

- Specs: GIVEN/WHEN/THEN; без имён `.vue` / CSS class.
- Design: файлы, токены, approx px, hex OK.
- Язык артефактов: русский prose; SHALL/MUST / headings structural — English где принято в проекте.
- Не дублировать весь inventory и в proposal, и в design — **канон чисел = design Visual Spec**; proposal — intent; specs — проверяемое поведение.

Если `OpenSpec: none` — не писать файлы; отдать полный extract и предложить `/openspec-propose` или `openspec new change` + capture.

Если change есть — править существующие артефакты по правилам [`openspec-update-change`](../openspec-update-change/SKILL.md) (coherence proposal ↔ design ↔ specs ↔ tasks). Перед первой записью в этом запуске кратко объявить файлы, которые будут изменены; при `rules.update` репо — писать сразу и отчитаться.

После записи: `openspec validate <name>` (если CLI доступен).

## Report (extract)

Язык: **русский**.

```markdown
# Prepare Mock — extract

Using mock: <path> (light | dark | both)
Using OpenSpec change: <name | none>
Surface: <e.g. task-set summary card>

## Структура (сверху вниз)
1. …
2. …

## Copy (точные строки)
| Элемент | Текст light/dark | Состояния | Align H | Align V (в ряду) | font-size | weight | line-height |
|---------|------------------|-----------|---------|------------------|-----------|--------|-------------|
| Title | Набор заданий #{n} | | center | — | … px | 600 | … |
| Badge | СНЯТО | | center | — | … | … | … |
| Stats label | Лёгкие: | | left (shared edge) | center | … | … | … |
| Stats count | 24 | | right (shared edge) | center | … | … | … |
| Button | Снять | | center (icon+text) | center | … | … | … |

## Типографика (сводка размеров шрифтов)
| Роль | font-size | weight | line-height | Notes | Confidence |
|------|-----------|--------|-------------|-------|------------|
| title | … | … | … | | |
| badge | … | … | … | | |
| stats-label | … | … | … | | |
| stats-count | … | … | … | | |
| button-label | … | … | … | | |

## Выравнивание текста (сводка)
| Правило с макета | Токен / notes |
|------------------|---------------|
| Labels одна левая вертикаль | … |
| Counts одна правая вертикаль | … |
| Title / badge centered | … |
| Button label centered | … |

## Геометрия (approx OK)
| Токен | Значение | Confidence |
|-------|----------|------------|
| card.width | 150–160px | high |
| card.height / min-height | … | |
| … | | |

## Размеры элементов (overall + каждый блок)
| Элемент | W | H (или min-H) | Pad / gap | Notes | Confidence |
|---------|---|----------------|-----------|-------|------------|
| card (overall) | … | … | … | radius … | |
| badge | … | … | … | | |
| title | … | … | … | | |
| total-row | … | … | … | icon …×… | |
| diff-row ×3 | … | … | … | dots … | |
| divider | full | 1px | … | each location | |
| action button | full | … | … | vs Quasar | |
| … | | | | | |

## Цвета
### Light
| Токен | Значение |
|-------|----------|
### Dark
| Токен | Значение |

## Разделители / иконки / actions
| Где | Есть? | Contrast | Notes |
|-----|-------|----------|-------|
| total → diff1 | yes | pale | |
| diff1 → diff2 | yes | pale | |
| diff2 → diff3 | yes | pale | |
| diff3 → actions | yes | pale | |
| Lead / labels / counts alignment | … | | |
| Button height approx | … px | | vs Quasar default |

## Недостающие иконки (для реализации)
| ID | Место на макете | Зачем | Suggested file | Temp Material | Блокер apply? |
|----|-----------------|-------|----------------|---------------|---------------|
| I1 | … | … | … | … | нет |
| … | | | | | |
(- none — если custom asset не нужен / уже в репо)

## Состояние наведения (default → hover)
| Target | Theme | Свойство | Default | Hover | Notes |
|--------|-------|----------|---------|-------|-------|
| card | light | border-color | … | … | |
| card | light | border-width / ring | … | … | |
| card | light | shadow | … | … | |
| card | light | background | … | … | |
| card | light | scale / translate | none | none / … | **no enlarge** if so |
| card | dark | … | … | … | |
| action btn | … | … | … | … | if shown |
(- hover не показан на макете — если на кадре только покой)

**Не меняется при hover:** …
**Muted / disabled (не hover):** …

## Неоднозначно / не на макете
- …

## Capture
- design.md Visual Spec: written | pending
- Missing icons → design ### Missing icons: written | pending
- Hover → design ### Hover / focus: written | pending
- specs touched: <ids or none>
- tasks added: <n or none>
```

## Связь с verify-mock

После apply `verify-mock` обязан считать **Visual Spec в design** первичным измеримым каноном рядом с картинкой: если код ≠ Visual Spec → Violation; если макет нечитаем, а Visual Spec есть — сверять со Spec.

## Не делать

- Не invent точные px «для красоты», если на скрине не видно — `approx` + range.
- Не ограничиваться overall карточки: **каждый** видимый элемент — строка в «Размеры элементов».
- Не переносить в specs имена компонентов/CSS.
- Не реализовывать UI в этом скилле.
- Не архивировать change и не запускать implement-change.
- Не пропускать бледные dividers между stats-rows / над actions в Visual Spec.
- Не игнорировать уточнение пользователя по макету.
- Не пропускать **выравнивание текста** (left/center/right + общие кромки labels/counts); не подставлять «как в Quasar по умолчанию» без макета.
- Не пропускать **размеры шрифтов** (font-size / weight / line-height) по ролям — отдельная таблица в extract и Visual Spec.
- Не сваливать hover в одну строку «есть тень»: отдельно border/shadow/bg/scale; light vs dark; что не меняется.
