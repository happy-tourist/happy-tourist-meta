## Why

Карточки карт в списке Maps всё ещё показывают автора и ёмкость одной строкой `players×tourists`, а chrome (flat-кнопки, hover через secondary, бейджи Quasar solid) расходится с уже согласованными карточками паков и наборов заданий. Нужен макет-aligned редизайн: без автора на карточке, явные строки игроков/туристов, статус поверх превью по центру, outline-actions и shared `--pack-card-*` — с приоритетом консистентности с существующим набором паков/заданий над пикселями макета.

## What Changes

- Редизайн **карточек карты в Maps list**: resting width около **180**, mini preview сверху, **две** строки ёмкости (ИГРОКОВ / ТУРИСТОВ) с lead-иконками, без автора на карточке.
- Status badge **overlay по центру** превью; short labels как у task-set (`ЧЕРНОВИК` / `НА ПРОВЕРКЕ` / `ДОРАБОТАТЬ` / `СНЯТО`); published in-catalog **без** бейджа «ОПУБЛИКОВАНО».
- Card actions: outline + icon как у паков/заданий; короткие «Снять» / «Вернуть»; confirm/header длинные ключи maps остаются.
- Soft-unpublished: muted (opacity ~0.72 + dashed border) как у pack cards.
- Shared chrome: потреблять `--pack-card-*` (не локальный map-only hover `--q-secondary`).
- Lead icons: **reuse** existing `map-card-players.svg` / `map-card-tourists.svg` (уже в sibling `src/assets/content/`); Material placeholder только если файла нет.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/maps`: chrome Maps list card; убрать author с карточки списка; явная ёмкость двумя строками; ревизия SC-MAP-55 (+ связанные формулировки list row про author на карточке).

## Scope

- **Capability ID:** `content/maps`
- **Пакеты:** client only
- Поверхность: Maps list cards; Visual Spec из prepare-mock (light+dark)
- Консистентность: кнопки / статусы / отступы / цвета = канон pack/task-set (`--pack-card-*`, short badges/actions); макет — структура и copy ёмкости
- Ширина карточки **180** (отдельный каталог от паков — ок)
- Mock asset в change `assets/`

## Out of scope

- Server / HTTP maps API — **без** изменений `GET /api/content/maps` (и unpublish/republish): payload по-прежнему отдаёт `authorDisplayName`, `players`, `touristsPerPlayer`, `grid`, `inCatalog`, `moderationStatus` и т.д.; карточка списка просто **не рендерит** автора
- Редизайн Map editor / View meta (автор на View остаётся; данные автора — из того же API)
- Staff moderation queue chrome
- Lobby create map picker
- Catalog packs / task-set body redesign
- Hover scale / enlarge
- Published badge «ОПУБЛИКОВАНО»
- Смена ACL / фильтров Maps / soft-unpublish semantics
- Длинные confirm unpublish copy
- Удаление `authorDisplayName` из list/detail HTTP (mocha SC-MAP-06 и View SC-MAP-47)

## Impact

- Client: Maps list tile + wiring badges/actions; i18n short card labels; **reuse** lead SVG (`map-card-players` / `map-card-tourists`); vitest SC-MAP-55 (+ SC-MAP-66/67)
- Delta: `content/maps`
- Skills: `work-with-styles/pack-cards.md`, maps pages/stores topics
- References: prepare-mock extract 2026-10-02; explore redesign map cards; prior pack/task-set redesigns

## References

- Prepare-mock / explore: map card mock light+dark; consistency > mock pixels; badge center; no author; width 180
- Mock: `openspec/changes/redesign-map-list-cards/assets/map-card-mock.jpg`
- Prior: archive `2026-10-02-redesign-pack-list-cards`, `2026-10-01-redesign-task-set-cards` (maps were out of scope)
- Main `openspec/specs/content/maps`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `docs/projects-map.md`
