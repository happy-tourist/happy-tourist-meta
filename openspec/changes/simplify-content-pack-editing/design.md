## Context

См. `proposal.md` — Why. Базовая модель: одна working copy до first approve; после live — freeze для non-staff; add-task-set; staff lock + session cards↔tasks; без personal drafts / stale.

После apply блоков 1–7: soft-unpublish пака, cascade CSS, slot chips, add-task-set thread в runtime. Explore follow-up 4: unpublish пака не во всех списках (нет в коллекции); нет confirm; нет soft-hide task set; live разворачивает все вопросы; AddTaskSet answer tiles — прямоугольные `q-btn`.

Пакеты: **server** + **client**. Room/tourist не затрагивается. `block` остаётся отдельным API без UI.

## Goals / Non-Goals

**Goals:**

- Одна рабочая копия до первого catalog approve; после live — freeze для non-staff (включая creator)
- Единый first-submit + единый Approve / Needs revision; Cancel автором
- Add-only новый task set (verified + inCollection); staff live edit + exclusive lock без очереди
- Support change-pack topic + catalog select + link
- Удалить personal drafts API/UI и draftStale-rebase
- Staff Edit: одна сессия cards↔tasks без сброса lock; без my-moderation nav; staff hint без «отправки на модерацию»
- Add-task-set affordance у секции «Задания» на live; не в списке коллекции / не в шапке live
- Content-редакторы: подсказки без скачков высоты
- **Staff soft-unpublish / republish pack:** публичный каталог скрывает; staff видят в каталоге **и коллекции**; confirm перед unpublish; republish one-click; gray+подпись; enter/Edit только staff
- **Soft-unpublish / republish task set:** flag на set; ≥1 published set; gray row + no enter; staff Edit + republish на строке и внутри Tasks; confirm на unpublish set
- **Live set summary + drill-in:** ответы + строки sets (count + diff 1/2/3); вопросы+слоты только после клика
- **Cascade yellow на набор:** outline на task-set row в cards editor, пока есть cascade gaps (паритет с task yellow на Tasks)
- **Slot chips везде в списках вопросов:** staff hub, add-task-set, tasks editor, live drill-in
- **Add-task-set thread UX:** статус needs_revision/pending + moderation thread + reply как на cards editor
- **AddTaskSet answer tiles:** `q-chip` parity with Tasks

**Non-Goals:**

- Block UI продукта (block ≠ unpublish)
- Новые внешние сервисы / npm deps
- Peek / tourist-room binding
- Отдельная роль admin ≠ moderator в content (оба `isStaff`)
- Hard-delete published / soft-unpublished packs
- Hard-delete published / soft-unpublished task sets (позже)

## Decisions

### D1 — Working copy storage

Хранить **один** mutable revision/payload на pack до `liveRevisionId` (creator-owned), без строк `content_user_drafts` на пользователя. После первого approve рабочая копия non-staff закрыта; add-task-set — отдельный pending payload set’а, не полный pack-draft.

**Альтернатива:** оставить drafts только для creator — отвергнуто (всё равно dual-path).

### D2 — Moderation requests

First publish: один request type «pack» (или объединённый answers+tasks) с одной кнопкой Approve = выставить live cards + tasks. Add-task-set: request type `tasks` / `task_set` на один set. Статус staff: pending | needs_revision (без terminal rejected). Author Cancel → closed.

### D3 — Staff lock + Edit session

Колонка/строка `edit_locked_by` + `edit_locked_at` на pack. `POST/GET edit-lock` при входе в Edit; heartbeat или short TTL (например 2–5 мин). Concurrent Edit → 409. Save пишет сразу в live revision (published / soft-unpublished с сохранённым live) или working copy (never-published).

**Session (follow-up):** lock держится на всём staff Edit (cards editor + tasks page). **Не** вызывать `releaseEditLock` / не обнулять `staffEditTarget` при навигации editor → tasks и обратно. Unlock — только при выходе из Edit целиком (уход на live/collection/catalog/другой pack) или TTL. Tasks `persist` для staff → всегда `staff-save`, не working-copy. Heartbeat можно держать на обеих страницах.

### D4 — Remove draft-stale extras (not unpublish)

Удалить HTTP/UI: draft rebase/stale, wipe-drafts. **Не** удалять unpublish/republish навсегда — см. D9 (restore). `last_live*` колонки: использовать только если soft-hide модель их потребует; иначе flag на pack достаточен.

### D5 — Support change-pack

Topic id kebab/snake в каноне API (например `change_pack`); i18n «Изменить набор карточек». Create body: `{ topic, body, packId }` когда topic=change_pack. Server валидирует pack в **публичном** каталоге (has live **и** in-catalog). В первое сообщение / метаданные тикета — URL live pack (из `CLIENT_APP_URL` + hash route). Staff list показывает topic + pack ref. Soft-unpublished packs MUST NOT appear in change_pack select.

### D6 — Client surfaces

- Never-published: creator → полный editor + Submit
- In-catalog published non-staff: только «Добавить набор заданий» flow/page, без списков карточек/set’ов в edit-режиме
- Soft-unpublished non-staff: коллекция — серая disabled строка + «Снято с публикации»; row не кликабельна; deep-link / live GET → отказ; add-task-set недоступен; creator после первого approve = как ordinary user (D7 explore)
- Staff: Edit на live/collection/catalog (вкл. soft-unpublished) без Collect; без кнопки Submit на модерацию; subtitle/hint про мгновенное сохранение (`staffEditSubtitle`)
- My-moderation: список открытых заявок **только для non-staff**; nav «На модерации» скрыт при `auth.isStaff`

### D7 — Add-task-set placement (follow-up)

На live pack page (in-catalog): кнопка «Добавить набор заданий» **рядом с заголовком секции «Задания»**. Убрать: row `playlist_add` в списке коллекции; шапочную `addTaskSetNav` в header live. Staff добавляет set через полный Edit.

### D8 — Layout-stable editor hints (follow-up)

В ContentPackEditor / Tasks / AddTaskSet: не показывать условные `text-caption` через `v-if`, которые скачут высоту. Предпочтительно: `q-tooltip` на disabled-кнопке; либо reserved/static space.

### D9 — Staff soft-unpublish / republish (explore D1–D7)

**Модель:** soft-hide. Не обнулять `liveRevisionId` при unpublish — сохранить live payload. Флаг на pack (например `inCatalog` / `catalogListed` boolean, default true когда есть live; или `unpublishedAt`). Публичный `listCatalog` / collect / non-staff `getLive` фильтруют `inCatalog === true`. Staff `listCatalog` возвращает и soft-unpublished с полем статуса. Republish = выставить inCatalog true **без** новой moderation request.

**UI:**
- Staff: «Снять с публикации» в **каталоге**, **коллекции** и **внутри набора** (live); «Опубликовать снова» в тех же местах, когда снято
- **Confirm** перед unpublish пака; republish пака — **без** confirm (one-click)
- Non-staff коллекция: строка gray/disabled, подпись «Снято с публикации», **не кликабельна**; trash/remove-from-collection остаётся
- Non-staff deep-link на pack → отказ / «снято» (не read-only live)
- Staff moderation queue / my-moderation: **без** unpublish controls

**ACL:** только `isStaff` вызывают unpublish/republish и открывают/Edit soft-unpublished. Creator после first approve не получает working-copy Edit при unpublish.

**Отличие от block:** block — видимый бейдж + запрет edit (UI скрыт); unpublish — скрытие из публичного каталога + gray collection. Не объединять.

**Альтернатива:** clear `liveRevisionId` + stash last_live — отвергнуто: лишняя миграция; staff Edit/republish проще при сохранённом live.

### D10 — Cascade yellow CSS on cards editor (explore)

`ContentPackEditorPage` уже вешает `cascade-gap-outline` на строку набора через `taskSetHasCascadeGap`. Стили `.cascade-gap-outline` сейчас **только** в `ContentPackTasksPage` (`scoped`) — на editor рамки нет. **Fix:** тот же outline CSS на Editor (scoped или shared). Логику `markCascadeGaps` / store не менять, если gaps уже ставятся после delete/content cascade.

### D11 — Answer slots on every question list row (explore Q1)

Во **всех** списках вопросов (не только TasksPage) каждая строка задания MUST показывать slot chips (filled content / empty): `ContentStaffRequestPage` hub, `ContentPackAddTaskSetPage` list, `ContentPackTasksPage` (уже), live drill-in view. Паттерн chips как на TasksPage. Не возвращать отдельную nested staff-tasks page ради слотов.

### D12 — Add-task-set moderation thread parity (explore Q2=A)

`ContentPackAddTaskSetPage` MUST показывать тот же UX-блок, что cards editor: статус под заголовком (pending / needs_revision), список `moderationThread` messages, reply при open status. Данные: reuse `loadModeration` / `postModerationMessage` (API уже резолвит open `task_set`); опционально добавить `messages` в GET add-task-set — не обязательно, если client грузит thread отдельно. My-moderation → add-task-set для type task_set остаётся.

### D13 — Soft-unpublish task set (explore follow-up 4)

**Модель:** soft-hide на уровне **task set** (не отдельных questions). Флаг на set в live revision payload / schema (напр. `inCatalog` / `published` boolean, default true при approve/append). Soft-unpublish set **не** трогает pack `inCatalog`. **Запрет:** нельзя снять set, если он единственный с published/inCatalog=true на live паке (server 409 + client disable).

**ACL:** только `isStaff`. Confirm перед unpublish set. Republish set — one-click на **строке** (Editor / live list) и **внутри** Tasks page. Не показывать unpublish в staff moderation queue.

**UX снятого set:** серый + бейдж «Снято с публикации»; non-staff / ordinary viewers **не** входят (drill-in disabled); staff MAY Edit (staff-save) и Republish. Hard-delete published/soft-unpublished set — **out of scope**.

**Альтернатива:** hard-delete set с live — отвергнуто (explore: сначала soft; delete later).

### D14 — Live pack: set summary + drill-in (explore)

`ContentPackPage` (live): секция ответов + **список строк** task sets (как Editor): caption = число заданий + разбивка по сложности 1/2/3. **Не** разворачивать вопросы на месте. Клик по **опубликованной** строке → страница просмотра вопросов со слотами (reuse `ContentPackTasksPage` read-only **или** эквивалент; staff в Edit session → editable Tasks). Снятая строка: gray + бейдж; клик non-staff no-op; staff → Edit / Republish на строке.

### D15 — AddTaskSet answer tile shape (explore)

На `ContentPackAddTaskSetPage` picker «карточки для слотов» сейчас `q-btn` (прямоугольные). **Fix:** `q-chip clickable outline` как на `ContentPackTasksPage`.

### D16 — Confirm on pack unpublish (explore Q1)

Любой staff «Снять с публикации» для **пака** (catalog / collection / live) — через confirm dialog. Republish пака — без confirm.

## Risks / Trade-offs

- [Миграция чужих drafts] → потеря незапушенных правок; mitigation: один раз drop/archive drafts при деплое
- [Staff lock TTL] → забытый lock; mitigation: короткий TTL + staff может перебить только после expiry
- [Единый Approve] → staff обязан проверить cards и tasks; mitigation: UI checklist / disable Approve если 0 task sets
- [Edit session без unlock на route change] → lock «висит»; mitigation: TTL + unlock on leave Edit
- [Soft-unpublished в staff catalog] → путаница с never-published; mitigation: явный бейдж/подпись «Снято с публикации» vs «Черновик автора»
- [Open add-task-set pending при unpublish] → default: оставить pending; staff может needs_revision/cancel; non-staff amend через deep-link недоступен — зафиксировать в apply если всплывёт UX-дыра
- [Два open request pack+task_set] → getModerationThread предпочитает pack; post-publish у автора обычно только task_set — OK
- [Unpublish последнего published set] → pack в каталоге без playable sets; mitigation: server reject + client disable
- [Live drill-in vs Edit route] → путаница режимов; mitigation: read-only Tasks когда не staff Edit session

## Migration Plan

1. Deploy server: task-set published flag + unpublish/republish set endpoints; migrate existing live sets → published true
2. Deploy client: collection pack unpublish + confirms; live summary + drill-in; set soft-hide UI; AddTaskSet chips
3. Rollback follow-up 4: revert client + server set-flag endpoints; pack soft-hide остаётся

## Open Questions

Нет (explore follow-up 4: D1–D8 + Q1 confirm только Unpublish).

## Точки врезки

**Server** (`../happy-tourist-server/`):

- `src/db/schema.ts`, `src/lib/content.ts` — working copy, submit/approve, add-task-set, staff save/lock; pack `inCatalog`; **task-set published/inCatalog**; unpublish/republish pack+set; listCatalog; getLive ACL + set summaries; moderation thread
- `src/app.config.ts` — content + lock + unpublish/republish (pack + set) endpoints
- `src/lib/support.ts` — change_pack select только in-catalog
- `test/zz-contentPacks.test.ts`, `test/support.test.ts`

**Client** (`../happy-tourist.github.io/`):

- `src/stores/content.ts` — pack/set unpublish/republish; loadModeration
- `ContentCatalogPage.vue`, `ContentCollectionPage.vue`, `ContentPackPage.vue` — staff pack unpublish/republish + **confirm**; collection buttons; live **set summary rows** + drill-in
- `ContentPackEditorPage.vue` — set row unpublish/republish + cascade CSS
- `ContentPackTasksPage.vue` — read-only live drill-in; staff set republish/unpublish inside; task yellow + slots
- `ContentStaffRequestPage.vue` — slot chips (уже); **без** unpublish
- `ContentPackAddTaskSetPage.vue` — slot chips + thread + **q-chip answer tiles**
- `src/i18n/en-US/index.ts`
- unit tests SC-PACK-120… + 126… + **129…**

Чеклист — `tasks.md`.
