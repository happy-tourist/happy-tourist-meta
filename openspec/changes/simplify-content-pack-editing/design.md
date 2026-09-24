## Context

См. `proposal.md` — Why. Сейчас: `content_user_drafts` на (user, pack), dual submit answers|tasks, soft-lock foreign pending, Edit для verified + inCollection. В runtime мог остаться слой publish-ux (last-live / unpublish / draftStale) — снять вместе с drafts.

После первой реализации apply: staff Edit cards работает через `staff-save`, но переход на tasks снимает lock (`releaseEditLock` в `onBeforeUnmount` editor) → `staffEditTarget` null → tasks `persist` зовёт working-copy → `pack_published`. Условные caption’ы (`questionNeedsSlot`, submit/tasks hints) появляются через `v-if` и скачут высоту.

Пакеты: **server** + **client**. Room/tourist не затрагивается.

## Goals / Non-Goals

**Goals:**

- Одна рабочая копия до первого catalog approve; после live — freeze для non-staff
- Единый first-submit + единый Approve / Needs revision; Cancel автором
- Add-only новый task set (verified + inCollection); staff live edit + exclusive lock без очереди
- Support change-pack topic + catalog select + link
- Удалить personal drafts API/UI и unpublish/republish/stale-pull
- Staff Edit: одна сессия cards↔tasks без сброса lock; без my-moderation nav; staff hint без «отправки на модерацию»
- Add-task-set affordance у секции «Задания» на live; не в списке коллекции / не в шапке live
- Content-редакторы: подсказки без скачков высоты

**Non-Goals:**

- Block UI продукта
- Новые внешние сервисы / npm deps
- Peek / tourist-room binding
- Отдельная роль admin ≠ moderator в content (оба `isStaff`)

## Decisions

### D1 — Working copy storage

Хранить **один** mutable revision/payload на pack до `liveRevisionId` (creator-owned), без строк `content_user_drafts` на пользователя. После первого approve рабочая копия non-staff закрыта; add-task-set — отдельный pending payload set’а, не полный pack-draft.

**Альтернатива:** оставить drafts только для creator — отвергнуто (всё равно dual-path).

### D2 — Moderation requests

First publish: один request type «pack» (или объединённый answers+tasks) с одной кнопкой Approve = выставить live cards + tasks. Add-task-set: request type `tasks` / `task_set` на один set. Статус staff: pending | needs_revision (без terminal rejected). Author Cancel → closed.

### D3 — Staff lock + Edit session

Колонка/строка `edit_locked_by` + `edit_locked_at` на pack. `POST/GET edit-lock` при входе в Edit; heartbeat или short TTL (например 2–5 мин). Concurrent Edit → 409. Save пишет сразу в live revision (published) или working copy (unpublished).

**Session (follow-up):** lock держится на всём staff Edit (cards editor + tasks page). **Не** вызывать `releaseEditLock` / не обнулять `staffEditTarget` при навигации editor → tasks и обратно. Unlock — только при выходе из Edit целиком (уход на live/collection/catalog/другой pack) или TTL. Tasks `persist` для staff → всегда `staff-save`, не working-copy. Heartbeat можно держать на обеих страницах.

### D4 — Remove publish-ux extras

Удалить HTTP/UI: unpublish, republish, draft rebase/stale, last_live* если больше не нужны. Migrate: nullable columns harmless или drop в ensureContentTables.

### D5 — Support change-pack

Topic id kebab/snake в каноне API (например `change_pack`); i18n «Изменить набор карточек». Create body: `{ topic, body, packId }` когда topic=change_pack. Server валидирует pack в каталоге (has live). В первое сообщение / метаданные тикета — URL live pack (из `CLIENT_APP_URL` + hash route). Staff list показывает topic + pack ref.

### D6 — Client surfaces

- Unpublished: creator → полный editor + Submit
- Published non-staff: только «Добавить набор заданий» flow/page, без списков карточек/set’ов в edit-режиме
- Staff: Edit на live/collection/catalog без Collect; без кнопки Submit на модерацию; subtitle/hint про мгновенное сохранение (`staffEditSubtitle`), не «отправка на модерацию»
- My-moderation: список открытых заявок **только для non-staff**; nav «На модерации» скрыт при `auth.isStaff` (catalog + collection)

### D7 — Add-task-set placement (follow-up)

На live pack page: кнопка «Добавить набор заданий» **рядом с заголовком секции «Задания»** (как «Набор заданий» у staff в editor). Убрать: row `playlist_add` в списке коллекции; шапочную `addTaskSetNav` в header live. Staff по-прежнему добавляет set через полный Edit (кнопка у секции в editor).

### D8 — Layout-stable editor hints (follow-up)

В ContentPackEditor / Tasks / AddTaskSet: не показывать условные `text-caption` через `v-if`, которые появляются и толкают layout (`questionNeedsSlot` под формой, `submitHint` / `tasksSaveHint` снизу при смене canSubmit). Предпочтительно: `q-tooltip` на disabled-кнопке; либо всегда видимый статичный hint / зарезервированная высота. Цель — **нет скачков высоты** при наборе текста / заполнении слотов.

## Risks / Trade-offs

- [Миграция чужих drafts] → потеря незапушенных правок; mitigation: один раз drop/archive drafts при деплое
- [Staff lock TTL] → забытый lock; mitigation: короткий TTL + staff может перебить только после expiry (не steal mid-session без expiry)
- [Единый Approve] → staff обязан проверить cards и tasks; mitigation: UI checklist / disable Approve если 0 task sets
- [Edit session без unlock на route change] → lock «висит» если закрыть вкладку; mitigation: TTL + unlock on leave Edit

## Migration Plan

1. Deploy server: schema working-copy + lock; stop writing `content_user_drafts`; migrate creator unpublished → working copy; delete draft rows
2. Deploy client: новые ACL/surfaces; убрать draft/stale/unpublish UI; staff session + placement + stable hints
3. Rollback: feature трудно откатить без возврата drafts — держать change атомарным в одном релизе client+server

## Open Questions

Нет (продуктовые D* закрыты в explore, follow-up подтверждён).

## Точки врезки

**Server** (`../happy-tourist-server/`):

- `src/db/schema.ts`, `src/lib/content.ts` — working copy, submit/approve/needs-revision/cancel, add-task-set, staff save, lock; удалить drafts/unpublish/rebase
- `src/app.config.ts` — content + lock endpoints
- `src/lib/support.ts` + routes — topic + packId
- `test/zz-contentPacks.test.ts`, `test/support.test.ts`

**Client** (`../happy-tourist.github.io/`):

- `src/stores/content.ts`, `src/stores/support.ts` — staffEditTarget / releaseEditLock timing
- `ContentPackEditorPage.vue`, `ContentPackTasksPage.vue` — session unlock + staff persist + stable hints
- `ContentPackPage.vue`, `ContentCollectionPage.vue`, `ContentCatalogPage.vue` — my-moderation nav; add-task-set placement
- `ContentPackAddTaskSetPage.vue` — stable hints
- `src/i18n/en-US/index.ts`
- unit tests (ACL + staff session + layout hints)

Чеклист — `tasks.md`.
