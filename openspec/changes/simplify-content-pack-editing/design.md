## Context

См. `proposal.md` — Why. Базовая модель: одна working copy до first approve; после live — freeze для non-staff; add-task-set; staff lock + session cards↔tasks; без personal drafts / stale.

После apply блоков 1–5: unpublish/republish HTTP/UI сняты (task 1.5 / D4 старый). Explore вернул потребность staff soft-unpublish.

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
- **Staff soft-unpublish / republish:** публичный каталог скрывает; staff видят в общем каталоге; коллекция gray+подпись; enter/Edit только staff; republish одной кнопкой без новой модерации

**Non-Goals:**

- Block UI продукта (block ≠ unpublish)
- Новые внешние сервисы / npm deps
- Peek / tourist-room binding
- Отдельная роль admin ≠ moderator в content (оба `isStaff`)
- Hard-delete published / soft-unpublished packs

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
- Staff: «Снять с публикации» в **каталоге** (row) и **внутри набора** (live/staff view); «Опубликовать снова» в тех же местах, когда снято
- Non-staff коллекция: строка gray/disabled, подпись «Снято с публикации», **не кликабельна**; trash/remove-from-collection остаётся
- Non-staff deep-link на pack → отказ / «снято» (не read-only live)

**ACL:** только `isStaff` вызывают unpublish/republish и открывают/Edit soft-unpublished. Creator после first approve не получает working-copy Edit при unpublish.

**Отличие от block:** block — видимый бейдж + запрет edit (UI скрыт); unpublish — скрытие из публичного каталога + gray collection. Не объединять.

**Альтернатива:** clear `liveRevisionId` + stash last_live — отвергнуто: лишняя миграция; staff Edit/republish проще при сохранённом live.

## Risks / Trade-offs

- [Миграция чужих drafts] → потеря незапушенных правок; mitigation: один раз drop/archive drafts при деплое
- [Staff lock TTL] → забытый lock; mitigation: короткий TTL + staff может перебить только после expiry
- [Единый Approve] → staff обязан проверить cards и tasks; mitigation: UI checklist / disable Approve если 0 task sets
- [Edit session без unlock на route change] → lock «висит»; mitigation: TTL + unlock on leave Edit
- [Soft-unpublished в staff catalog] → путаница с never-published; mitigation: явный бейдж/подпись «Снято с публикации» vs «Черновик автора»
- [Open add-task-set pending при unpublish] → default: оставить pending; staff может needs_revision/cancel; non-staff amend через deep-link недоступен — зафиксировать в apply если всплывёт UX-дыра

## Migration Plan

1. Deploy server: working-copy + lock (уже); добавить inCatalog/soft-unpublish + endpoints; migrate existing live packs → inCatalog true
2. Deploy client: ACL surfaces (уже) + unpublish/republish UI + collection gray
3. Rollback unpublish: feature-flag или revert endpoints; публичный каталог снова показывает все live

## Open Questions

Нет (explore D1–D7 закрыты; defaults: trash на gray-строке; block отдельно; pending add-task-set не авто-cancel).

## Точки врезки

**Server** (`../happy-tourist-server/`):

- `src/db/schema.ts`, `src/lib/content.ts` — working copy, submit/approve, add-task-set, staff save/lock; **inCatalog / unpublish / republish**; listCatalog staff vs public; getLive ACL
- `src/app.config.ts` — content + lock + unpublish/republish endpoints
- `src/lib/support.ts` — change_pack select только in-catalog
- `test/zz-contentPacks.test.ts`, `test/support.test.ts`

**Client** (`../happy-tourist.github.io/`):

- `src/stores/content.ts` — unpublish/republish wrappers; catalog item flags; collection gray
- `ContentCatalogPage.vue`, `ContentPackPage.vue` — staff unpublish/republish controls
- `ContentCollectionPage.vue` — gray disabled row + i18n label; no row navigate when unpublished
- `ContentPackEditorPage.vue` / Tasks — staff Edit soft-unpublished (уже staff-save path)
- `src/i18n/en-US/index.ts`
- unit tests SC-PACK-120…

Чеклист — `tasks.md`.
