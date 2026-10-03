## 1. Client — answer compose dialog

- [x] 1.1 Прочитать `design.md` D1/D3/D5/D7; delta MODIFIED SC-PACK-222…227 + ADDED 256/257; skills `work-with-pages` / content-pages, `work-with-forms`, `work-with-styles` / pack-cards, `work-with-stores` / content; текущий inline form в `ContentPackEditorPage` — подтвердить точку врезки
- [x] 1.2 Убрать inline answer form; над `answer-card-grid` — кнопка добавления (скрыта при `cardsReadOnly`); create/edit в dialog (fields only, без preview); Edit плитки открывает ту же модалку только если compose allowed — verify vitest SC-PACK-256/257
- [x] 1.3 i18n ключи для add/dialog titles при необходимости; Save/Cancel закрывают dialog; cascade/save logic без смены store API — verify vitest (существующие save/cascade suites + SC-PACK-256/257)

## 2. Client — task compose dialog (Tasks + AddTaskSet)

- [x] 2.1 Прочитать `design.md` D1/D2/D4/D5/D7; delta SC-PACK-258…262 + MODIFIED SC-PACK-222…227 (полный блок); skills content-pages + pack-cards + `work-with-game-board/peek.md` (эталон layout); compose на `ContentPackTasksPage` / `ContentPackAddTaskSetPage`
- [x] 2.2 Shared (или зеркальный) task compose `q-dialog`: question/difficulty → **centered** peek-slot-like row (`justify-center`) → chip answer pool (mirror GamePage `.peek-answers`, не PackAnswerCardTile / не `slot-picker-grid`); add-question над списком (hide при viewOnly/readOnly); Edit плитки → та же модалка — verify vitest SC-PACK-258/259/260
- [x] 2.3 На AddTaskSet удалить верхний `live-answer-card-grid` (+ `addTaskSetLiveCardsHint` / empty stub над compose); live cards только в dialog pool и CSV context — verify vitest SC-PACK-261
- [x] 2.4 `PackTasksCsvControls` остаётся на странице сверху списка заданий (вне dialog); viewOnly hide без регрессии SC-PACK-235/236 — verify vitest SC-PACK-262
- [x] 2.5 Регрессия slot chrome SC-PACK-237 внутри dialog; обновить устаревшие тесты на inline `slot-picker-grid` / page-level compose form (`ContentPackTasksCsvHide` и смежные)

## 3. Client — skills canon + quality gate

- [x] 3.1 Подтвердить meta skills уже согласованы с delta: `work-with-pages/content-pages.md`, `work-with-styles/pack-cards.md`, `work-with-stores/content.md` — compose = dialog; q-chip answer pool **only** inside task compose dialog; при drift — дотянуть те же файлы; **не** менять server skills / HTTP contract docs
- [x] 3.2 `npm test` (затронутые content/pack suites) + `npm run lint` + `npm run typecheck` в `../happy-tourist.github.io` — зелёный прогон; server sibling не трогать / не гонять как обязательный gate этого change
