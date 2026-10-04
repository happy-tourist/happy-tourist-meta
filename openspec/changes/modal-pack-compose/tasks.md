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

## 4. Client — reusable answer cards in slots (compose + peek)

- [x] 4.1 Прочитать `design.md` D8; delta SC-PACK-263 + `specs/game/board` SC-BOARD-49; skills `work-with-game-board/peek.md` + content-pages / pack-cards (убрать «disable when already placed»); точки: `PackTaskComposeDialog.vue`, `GamePage.vue` peek chips
- [x] 4.2 Compose: убрать uniqueness / used-state chrome (`isCardPlaced` из disable/clickable/outline/color и guard в `pickAnswer`); одна карточка MAY заполнить несколько/все слоты — verify vitest SC-PACK-263
- [x] 4.3 Peek на GamePage: убрать `isAnswerPlaced` disable/outline/color/early-return; chip остаётся доступным без consumed highlight; clear слота без регрессии sync — verify vitest SC-BOARD-49
- [x] 4.4 Skills drift: `peek.md` (+ при необходимости content-pages / pack-cards / content) — reuse без used-state; server skills не трогать
- [x] 4.5 `npm test` (compose + peek/board suites) + `npm run lint` + `npm run typecheck` в `../happy-tourist.github.io` — зелёный прогон; server не обязательный gate

## 5. Server + client — add-task-set pre-submit draft

- [x] 5.1 Прочитать `design.md` D9–D11; delta SC-PACK-264…268; skills server `work-with-routes` / structure / test + client content-pages / `work-with-stores/content`; точки: `putAddTaskSet` / `getAddTaskSet` / neverLive ghosts / cancel; `ContentPackAddTaskSetPage` autosave + top delete
- [x] 5.2 Server: draft put без submit-minima; создать/обновить `task_set` request `status=draft`; GET + neverLive + list drafts включают draft (+ legacy cancelled); Submit `draft`→`pending`; Cancel never-live → `draft` — verify mocha SC-PACK-264/265/267/268
- [x] 5.3 Server: discard/delete never-live draft endpoint (не трогает live sets/pack) — verify mocha SC-PACK-266
- [x] 5.4 Client: quiet autosave `saveAddTaskSet` на любое dirty изменение; reload восстанавливает draft; top delete + confirm; author-only draft/neverLive UX — verify vitest SC-PACK-264/265/266/268
- [x] 5.5 Skills drift (client content* + server routes/structure/test); `npm test` + lint + typecheck в client; mocha затронутых content suites + build в server

## 6. Server — persisted author draft lifecycle

- [x] 6.1 Прочитать `design.md` D12–D14; MODIFIED SC-PACK-234 + SC-PACK-269…275; skills server `work-with-database`, `work-with-routes/content.md`, structure/content, test; подтвердить точки create/list/save/submit/needs-revision/cancel/delete для pack и task-set
- [x] 6.2 Добавить persisted `content_packs.draft_activated` + pack/request `has_unsubmitted_changes` с совместимой migration/backfill: existing packs консервативно activated (старый untouched shell неотличим от metadata-only draft), новые untouched shells скрыты; retained task-set `draft` и legacy retained `cancelled` backfill dirty, staff-actionable `pending|needs_revision` — clean rollout baseline — verify mocha SC-PACK-269/270/271
- [x] 6.3 В SQLite transaction обновлять payload+lifecycle: semantic save (не identical retry) ставит dirty; первый save task-set author создаёт/reuses `draft` request carrier; `pending`→`draft` + clear take; dirty `needs_revision` сохраняет author status/thread и остаётся доступен через author moderation detail/reply, но clear take и исключается из staff queue/approve; Submit переиспользует retained request→clean `pending`; вернуть marker в list/editor/add-task-set payloads — verify mocha SC-PACK-271/272/273 + existing-set author cycle + approve-without-resubmit regression SC-PACK-74/158/163/234
- [x] 6.4 Расширить существующие pack delete и add-task-set discard endpoints на never-published pack/retained cycle в draft|pending|needs_revision: server-derived `canHardDelete` в draft/add-task-set GET, cascade request/messages/revision, все never-live sets текущего cycle; не затрагивать parent/published siblings и запретить direct hard-delete после первой публикации — verify mocha SC-PACK-274/275 + SC-PACK-55/56/114
- [x] 6.5 Запустить затронутые `zz-contentPacks` mocha suites + полный `npm test` и `npm run build` в `../happy-tourist-server`; исправить регрессии

## 7. Client — status and Submit survive reload

- [x] 7.1 Прочитать `design.md` D12–D14; MODIFIED SC-PACK-234 + SC-PACK-269…275; skills client content-pages, `work-with-stores/content`, localization, test; проверить pack editor, Tasks и AddTaskSet lifecycle gates
- [x] 7.2 Расширить content store draft/add-task-set response types и actions server-driven `hasUnsubmittedChanges` + `canHardDelete`; autosave fingerprint оставить только для dedupe, catalog/editor status синхронизировать с server response — verify store vitest SC-PACK-270/271/273/274
- [x] 7.3 Pack/task-set/AddTaskSet author UI: Submit = persisted dirty ∧ minima ∧ ACL/lock; после reload draft/changed needs_revision остаётся ready; собственные `pending` work остаются редактируемыми, а первый semantic save отображает `draft`; dirty needs_revision сохраняет label и author thread/reply — verify vitest SC-PACK-234/271/272/273
- [x] 7.4 Top delete показывать по server `canHardDelete` для never-published pack/retained add-task-set cycle в draft|pending|needs_revision с confirm; переиспользовать `deleteUnpublishedPack` / `discardAddTaskSetDraft`, после первой публикации скрывать; успешный delete очищает list/ghost и возвращает по каноническому route — verify vitest SC-PACK-274/275 + регрессия SC-PACK-266
- [x] 7.5 Empty pack shell не показывать в unified list; после первого сохранённого metadata/card/task изменения показывать author-only draft — verify vitest SC-PACK-269/270
- [x] 7.6 Добавить/обновить `ContentPackDraftLifecycle.test.ts`, `ContentPackDirtySubmit.test.ts`, `ContentPackAddTaskSetDraft.test.ts` по traceability; запустить затронутые vitest suites + полный `npm test`, `npm run lint`, `npm run typecheck` в `../happy-tourist.github.io`; исправить регрессии

## 8. Skills canon

- [x] 8.1 Обновить при drift client content-pages/store/test/locate и server database/routes/structure/test/locate: hidden shell + conservative backfill, persisted dirty, pending→draft, dirty needs_revision вне staff-actionable queue, authoritative `canHardDelete`, delete retained never-live cycle; не менять game/peek skills
