## 0. Server

- Нет server-задач: room/schema/messages/HTTP/DB без изменений (`proposal` Impact; `design` Unchanged server contract).

## 1. Board geometry — per-cell finish

- [x] 1.1 Перед работой: `design.md` D3; `specs/game/board` SC-BOARD-55; `src/lib/boardGeometry.ts` (`buildBoardTiles`); `src/lib/__tests__/boardGeometry.test.ts`
- [x] 1.2 Убрать merge contiguous `7` в один `span`; одна finish-клетка → один tile (`kind: 'center'`, без `span` в style); обновить unit-тесты `boardGeometry` (сейчас ждут 1 center + `span` — после: N centers без `span`, tourist layout = 4) (SC-BOARD-55) — `npm test` в client по этому файлу

## 2. Game board skins + flip digit + finish chrome

- [x] 2.1 Перед работой: `design.md` D1/D4; `specs/game/board` SC-BOARD-01/02/04/42/56/57; `specs/game/move` SC-MOVE-65; skills `work-with-game-board`, `work-with-styles/board.md`; `GamePage.vue` tile CSS / `isTileSelected` / `isTileTarget` / `onTileClick` / `nearestLegalCenterCell`
- [x] 2.2 Подключить PNG скины: `.tile-start`→`start.png`, `.tile-task`→`tile.png`, `.tile-task.tile--flipped`→`tile-back.png`, `.tile-center`→`finish.png`; цифра `#212121` без shadow; `border-radius: 0` на playable tiles; gap 2 сохранить (SC-BOARD-01/02/04/42/56/57)
- [x] 2.3 Finish click/outline per-cell: убрать block-wide `isCenterCell` / `CENTER_CELLS.some` в `isTileSelected`/`isTileTarget` и remap в `onTileClick`; везде `cellKey(tile.row, tile.col)` + membership в legal keys; удалить `nearestLegalCenterCell`; kind остаётся `'center'` (SC-MOVE-65); Vitest board/move UX — `npm test` затронутых suites
- [x] 2.4 Skill notes выровнены под D1–D4 / SC-MOVE-65 (`work-with-styles/board.md`, `work-with-game-board`, `work-with-pages/game-page.md` + `content-pages.md`, `work-with-stores/maps.md`, `client-verify-code`) — при apply только сверить, что runtime совпал

## 3. Map preview + editor palette skins

- [x] 3.1 Перед работой: `design.md` D2; `specs/content/maps` SC-MAP-54/71/72; `MapGridPreview.vue`; `MapEditorPage.vue` palette swatches; `ContentMaps.test.ts` assertions на swatch
- [x] 3.2 Скины kinds в `MapGridPreview` (start/task/finish); palette swatches теми же ассетами, swatch `border-radius: 0` (SC-MAP-54/71/72); обновить Vitest maps — `npm test` затронутых suites

## 4. Client verify

- [x] 4.1 Перед работой: сверка Traceability pending → covered в delta specs после тестов
- [x] 4.2 В client: `npm run lint`, `npm run typecheck`, `npm test` — зелёные; при падении починить до закрытия change
