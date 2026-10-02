## 0. Server

- [x] 0.1 **Нет server-задач.** Не менять `listMaps` / maps HTTP / mocha SC-MAP-06/16 payload (`authorDisplayName` остаётся). Apply = client-only — verify: в diff sibling server пусто

## 1. Client — tile + assets

- [x] 1.1 Прочитать `design.md` (Decisions + Visual Spec), delta `specs/content/maps/spec.md` (SC-MAP-55/66/67), skills `.agents/skills/client/work-with-styles/pack-cards.md` + `work-with-pages/content-pages.md` (maps), текущие `MapListCardTile` / `MapsListPage` / `PackTaskSetCardTile` (badge + action pattern) — verify: точки врезки ясны
- [x] 1.2 **Reuse** (уже в sibling HEAD) `src/assets/content/map-card-players.svg` и `map-card-tourists.svg` — Material placeholder только если файла нет; i18n `maps.mapCardPlayers` / `maps.mapCardTourists` («ИГРОКОВ:» / «ТУРИСТОВ:») — verify: mask URL + ключи на месте
- [x] 1.3 Обновить `MapListCardTile`: width ~180; preview + absolute centered `#status` overlay; два stat-row (lead 22 + label + count) + divider; без author/capacity props; `--pack-card-*` aliases (host `.pack-card-grid`); muted 0.72+dashed; outline actions slot (pad ~3/8/8); hover border+shadow без `--q-secondary` / без scale — verify: vitest SC-MAP-55 / 66 / 67 (компонент или page)

## 2. Client — Maps list wire

- [x] 2.1 `MapsListPage`: убрать prop author; передать players / touristsPerPlayer; short badges (`content.taskSetCardBadge.*` + tone/icon helpers как task-set / catalog revise — **не** `content.statuses.*` / `maps.draftOnly` / `maps.unpublishedByStaff` на pill); overlay status; `:muted` для soft-unpub; outline+icon actions (`content.edit` / `maps.staffEdit` / `content.taskSetCardUnpublish` / `content.taskSetCardRepublish`; confirm длинные `maps.unpublish*`) — verify: vitest SC-MAP-06 / 16 / 31 / 32 / 45 / 55 / 66 / 67 (card без Alice/`seatConfig`)
- [x] 2.2 Обновить skills: `pack-cards.md` (maps section — сейчас «bottom text actions»), `work-with-stores/maps.md`, `work-with-pages/content-pages.md` / locate-change-points при необходимости — verify: текст отражает новый chrome (180, no author, overlay, shared tokens)

## 3. Client verification

- [x] 3.1 `npm run lint`, `npm run typecheck`, `npm test` (затронутые ContentMaps / MapListCardTile) в sibling client — verify: все зелёные
