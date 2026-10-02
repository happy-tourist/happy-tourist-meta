## Why

Карточки карт в списке Maps всё ещё показывают автора и ёмкость одной строкой `players×tourists`, а chrome (flat-кнопки, hover через secondary, бейджи Quasar solid) расходится с уже согласованными карточками паков и наборов заданий. Нужен макет-aligned редизайн: без автора на карточке, явные строки игроков/туристов, статус поверх превью по центру, outline-actions и shared `--pack-card-*` — с приоритетом консистентности с существующим набором паков/заданий над пикселями макета.

После apply: lead/badge SVG через CSS mask рисуются как **сплошной квадрат** `currentColor` (черновик / игроки / туристы), хотя сами файлы SVG корректны — сломанный `mask-image` из-за `url(${viteImport})` без кавычек при data-URL инлайне Vite.

## What Changes

- Редизайн **карточек карты в Maps list**: resting width около **180**, mini preview сверху, **две** строки ёмкости (ИГРОКОВ / ТУРИСТОВ) с lead-иконками, без автора на карточке.
- Status badge **overlay по центру** превью; short labels как у task-set (`ЧЕРНОВИК` / `НА ПРОВЕРКЕ` / `ДОРАБОТАТЬ` / `СНЯТО`); published in-catalog **без** бейджа «ОПУБЛИКОВАНО».
- Card actions: outline + icon как у паков/заданий; короткие «Снять» / «Вернуть»; confirm/header длинные ключи maps остаются.
- Soft-unpublished: muted (opacity ~0.72 + dashed border) как у pack cards.
- Shared chrome: потреблять `--pack-card-*` (не локальный map-only hover `--q-secondary`).
- Lead icons: **reuse** existing `map-card-players.svg` / `map-card-tourists.svg` (уже в sibling `src/assets/content/`); Material placeholder только если файла нет.
- **Hotfix:** CSS mask wiring — `url("${importedSvg}")` (Vite canon) на `MapListCardTile` и том же паттерне `PackTaskSetCardTile`, чтобы силуэты иконок не деградировали в сплошной квадрат.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/maps`: chrome Maps list card; убрать author с карточки списка; явная ёмкость двумя строками; ревизия SC-MAP-55 (+ связанные формулировки list row про author на карточке); **SC-MAP-68** — lead/badge SVG mask силуэт (не сплошной квадрат).

## Scope

- **Capability ID:** `content/maps`
- **Пакеты:** client only
- Поверхность: Maps list cards; Visual Spec из prepare-mock (light+dark)
- Консистентность: кнопки / статусы / отступы / цвета = канон pack/task-set (`--pack-card-*`, short badges/actions); макет — структура и copy ёмкости
- Ширина карточки **180** (отдельный каталог от паков — ок)
- Mock asset в change `assets/`
- Hotfix mask URL quoting: `MapListCardTile` + симметричный паттерн на `PackTaskSetCardTile` (тот же `url(${…})` без кавычек)

## Out of scope

- Server / HTTP maps API — **без** изменений `GET /api/content/maps` (и unpublish/republish): payload по-прежнему отдаёт `authorDisplayName`, `players`, `touristsPerPlayer`, `grid`, `inCatalog`, `moderationStatus` и т.д.; карточка списка просто **не рендерит** автора
- Редизайн Map editor / View meta (автор на View остаётся; данные автора — из того же API)
- Staff moderation queue chrome
- Lobby create map picker
- Catalog packs / task-set **body** redesign (кроме одной строки mask `url("${…}")` на существующем `PackTaskSetCardTile`)
- Перерисовка / замена SVG-ассетов (файлы корректны; чинится wiring)
- Hover scale / enlarge
- Published badge «ОПУБЛИКОВАНО»
- Смена ACL / фильтров Maps / soft-unpublish semantics
- Длинные confirm unpublish copy
- Удаление `authorDisplayName` из list/detail HTTP (mocha SC-MAP-06 и View SC-MAP-47)

## Impact

- Client: Maps list tile + wiring badges/actions; i18n short card labels; **reuse** lead SVG (`map-card-players` / `map-card-tourists`); vitest SC-MAP-55 (+ SC-MAP-66/67/68); mask `url("${…}")` на map + task-set tiles
- Delta: `content/maps`
- Skills: `work-with-styles/pack-cards.md` (note Vite quoted `url()` for JS-built mask vars), maps pages/stores topics
- References: prepare-mock extract 2026-10-02; explore redesign map cards + mask square bug 2026-10-02; prior pack/task-set redesigns

## References

- Prepare-mock / explore: map card mock light+dark; consistency > mock pixels; badge center; no author; width 180
- Explore 2026-10-02: solid-square icons → Vite `url("${imported}")` for CSS mask vars
- Mock: `openspec/changes/redesign-map-list-cards/assets/map-card-mock.jpg`
- Prior: archive `2026-10-02-redesign-pack-list-cards`, `2026-10-01-redesign-task-set-cards` (maps were out of scope)
- Main `openspec/specs/content/maps`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `docs/projects-map.md`
- Vite docs: Static Asset Handling — quote SVG URL inside manually built `url()`
