## Why

Карточки паков в едином каталоге (`150×200`, только title + description) не показывают, какие наборы заданий внутри, и визуально отстают от уже согласованного chrome наборов заданий. Нужен макет-aligned редизайн карточки пака: title/description + preview существующих наборов + outline-actions — с лёгкими данными списка с сервера.

**Follow-up (explore 2026-10-01):** на карточке preview должен показывать **только опубликованные** наборы (без soft-unpub и never-live), с цветом строк как у title; ordinal / «ещё K» — только среди опубликованных. Live pack не меняем.

**Follow-up (explore chrome 2026-10-01):** кнопки и resting chrome каталожной карточки пака должны быть **консистентны с уже апрувнутым task-set card** (короткие «Снять»/«Вернуть», общий фон/рамка/hover/action outline/muted). Макет с длинным «Снять с публикации» и отдельным outline **не** канон — ориентир task-set. Shared CSS-токены вынести в `app.scss` и переиспользовать.

## What Changes

- Редизайн **карточек пака в unified catalog**: размер около **180×260**, uppercase title, description, список наборов (иконка + «Набор заданий #{n}» + taskCount), max **4** видимых + «ещё {k}», outline+icon actions снизу; звезда TL как сейчас.
- **Лёгкий preview** наборов в `GET /api/content/packs` (без полного revision payload); API по-прежнему может отдавать soft-unpub / never-live (SC-PACK-252/253).
- **Catalog card UI:** показывать только сеты с `inCatalog: true` и без `neverLive`; soft-unpublished и never-live ghost на карточку **не** выводить; ordinal / overflow считать по отфильтрованному списку (client).
- Цвет label / count / icon активных строк preview = **title.fg** (light near-black / dark near-white), не muted grey.
- Status badge «ДОРАБОТАТЬ» — red outline как на макете (короткий uppercase; не `content.statuses.needs_revision` «Нужна доработка»). Остальные статусы каталога — **текущие** catalog labels/colors (pending/draft/unpublished as today), не обязательно short task-set pills.
- **Card actions (catalog):** короткие labels как на task-set — `taskSetCardUnpublish` «Снять» / `taskSetCardRepublish` «Вернуть» (+ `content.edit`); confirm dialogs и header live pack остаются длинными (`content.unpublish` / `content.republish`).
- **Shared pack-card chrome:** CSS custom properties `--pack-card-*` в `src/css/app.scss` (рядом с `.pack-card-grid`); `PackListCardTile` и `PackTaskSetCardTile` потребляют resting surface (bg/fg/muted/border/hover/shadow/splitter/action-h) + soft-unpub muted (`opacity: 0.72` + `border-style: dashed`). Layout/тело карточек остаются разными.
- **BREAKING** (UX/spec): SC-PACK-228 больше не фиксирует `150×200` без set-preview и text-only actions; SC-PACK-249 больше не требует muted soft-unpub / never-live rows на карточке; SC-PACK-250 больше не требует длинного «Снять с публикации» на catalog card actions.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: chrome catalog pack card; set preview на карточке; list API preview; ревизия SC-PACK-228 / 249 / 250; shared chrome SC-PACK-255.

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client (+ server list preview уже сделан)
- Поверхность: unified packs catalog (`PackListCardTile`); механический wire `PackTaskSetCardTile` на shared tokens (без редизайна тела task-set)
- Контракт HTTP: без изменения — полный `taskSetsPreview[]`; клиент фильтрует для UI
- Mock / Visual Spec: prepare-mock light+dark → design; chrome/actions **superseded** task-set canon где расходятся
- Skills/docs: `work-with-styles/pack-cards.md` (catalog + shared tokens)

## Out of scope

- Продуктовый редизайн тела `PackTaskSetCardTile` / answer/task playing-cards / **maps** (`MapListCardTile`)
- Смена нумерации / скрытие soft-unpub на **live** pack / editor (карточка каталога ≠ live)
- Фильтрация soft-unpub / never-live на **сервере** в `taskSetsPreview` (оставляем API; UI режет)
- Staff hub как card-grid
- Hover scale / enlarge
- Увеличение gutter `.pack-card-grid`
- Custom SVG для action-кнопок (Material как у task-set)
- Смена ACL soft-unpublish / favorite / list filters
- Длинный confirm unpublish copy / длинные pack header buttons на live pack page
- Drop `authorDisplayName` / coauthor columns
- Lobby create picker chrome (кроме косвенного ordinal copy уже без автора)

## Impact

- Client: catalog pack tile + **shared `--pack-card-*` + short card actions**; store types для preview; published-only filter + title.fg; tests SC-PACK-228 / 249 / 250 / 254 / 255
- Server: `listCatalog` лёгкий `taskSetsPreview` (уже); SC-PACK-252/253 без обязательной смены
- Delta: `content/packs`
- Skills: `pack-cards.md`
- References: prepare-mock extract; explore D1–D3 / N=4; follow-up published-only + title.fg; follow-up chrome consistency + shared tokens

## References

- Explore 2026-10-01: redesign pack list cards; D1 lightweight preview; D2 max 4 + ещё K; Material actions; whole-card navigate
- Explore follow-up: D1 published-only on card; D2 ordinal among published; D3 same change; Q1 card≠live; Q2 client filter
- Explore chrome: D1 consistency with task-set (buttons + bg/border); D2 same change; short card labels; shared `--pack-card-*` in `app.scss`; muted dashed; maps out
- Prepare-mock / Visual Spec: `openspec/changes/redesign-pack-list-cards/assets/pack-card-catalog-mock.jpg` (+ light/dark crops рядом); action/outline tokens overridden by task-set canon
- Prior: archive `2026-10-01-redesign-task-set-cards` (catalog was explicitly out of scope — now in scope)
- Main `openspec/specs/content/packs`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
