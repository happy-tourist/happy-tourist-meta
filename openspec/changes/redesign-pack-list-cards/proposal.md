## Why

Карточки паков в едином каталоге (`150×200`, только title + description) не показывают, какие наборы заданий внутри, и визуально отстают от уже согласованного chrome наборов заданий. Нужен макет-aligned редизайн карточки пака: title/description + preview существующих наборов + outline-actions — с лёгкими данными списка с сервера.

## What Changes

- Редизайн **карточек пака в unified catalog**: размер около **180×260**, uppercase title, description, список наборов (иконка + «Набор заданий #{n}» + taskCount), max **4** видимых + «ещё {k}», outline+icon actions снизу; звезда TL как сейчас.
- **Лёгкий preview** наборов в `GET /api/content/packs` (без полного revision payload).
- Soft-unpublished сеты в preview (muted); never-live ghost — только автору сета, с актуальным count.
- Status badge «ДОРАБОТАТЬ» — red outline как на макете (короткий uppercase; не `content.statuses.needs_revision` «Нужна доработка»). Остальные статусы каталога — **текущие** catalog labels/colors (pending/draft/unpublished as today), не обязательно short task-set pills.
- **BREAKING** (UX/spec): SC-PACK-228 больше не фиксирует `150×200` без set-preview и text-only actions.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: chrome catalog pack card; set preview на карточке; list API preview; ревизия SC-PACK-228 / запрета «catalog MUST NOT adopt task-set summary rows».

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client + server (лёгкий list preview)
- Поверхность: unified packs catalog (карточки пака)
- Контракт HTTP: на каждый pack в `GET /api/content/packs` — полный `taskSetsPreview[]` (`id`, 1-based `ordinal`, `taskCount`, `inCatalog`, `neverLive?`); клиент режет до 4 + «ещё K»
- Mock / Visual Spec: prepare-mock light+dark → design
- Skills/docs: `work-with-styles/pack-cards.md` (catalog tile)

## Out of scope

- Редизайн `PackTaskSetCardTile` / answer/task playing-cards / maps cards
- Staff hub как card-grid
- Hover scale / enlarge
- Увеличение gutter `.pack-card-grid`
- Custom SVG для action-кнопок (Material как у task-set)
- Смена ACL soft-unpublish / favorite / list filters
- Длинный confirm unpublish copy
- Drop `authorDisplayName` / coauthor columns
- Lobby create picker chrome (кроме косвенного ordinal copy уже без автора)

## Impact

- Client: catalog pack tile chrome + i18n overflow/actions; store types для preview; tests SC-PACK-228 revision + new scenarios
- Server: `listCatalog` лёгкий `taskSetsPreview` (counts без slots/cards; батч по revId); author never-live merge как на live (`neverLiveAuthorTaskSets` rules)
- Delta: `content/packs`
- Skills: `pack-cards.md`
- References: prepare-mock extract; explore decisions D1–D3 / N=4

## References

- Explore 2026-10-01: redesign pack list cards; D1 lightweight preview; D2 max 4 + ещё K; D3 soft-unpub everywhere pack cards; never-live for author; Material actions; whole-card navigate
- Prepare-mock / Visual Spec: `openspec/changes/redesign-pack-list-cards/assets/pack-card-catalog-mock.jpg` (+ light/dark crops рядом)
- Prior: archive `2026-10-01-redesign-task-set-cards` (catalog was explicitly out of scope — now in scope)
- Main `openspec/specs/content/packs`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
