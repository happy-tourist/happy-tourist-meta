## Context

См. `proposal.md` — Why. Сейчас: `content_user_drafts` на (user, pack), dual submit answers|tasks, soft-lock foreign pending, Edit для verified + inCollection. В runtime мог остаться слой publish-ux (last-live / unpublish / draftStale) — снять вместе с drafts.

Пакеты: **server** + **client**. Room/tourist не затрагивается.

## Goals / Non-Goals

**Goals:**

- Одна рабочая копия до первого catalog approve; после live — freeze для non-staff
- Единый first-submit + единый Approve / Needs revision; Cancel автором
- Add-only новый task set (verified + inCollection); staff live edit + exclusive lock без очереди
- Support change-pack topic + catalog select + link
- Удалить personal drafts API/UI и unpublish/republish/stale-pull

**Non-Goals:**

- Block UI продукта
- Новые внешние сервисы / npm deps
- Peek / tourist-room binding

## Decisions

### D1 — Working copy storage

Хранить **один** mutable revision/payload на pack до `liveRevisionId` (creator-owned), без строк `content_user_drafts` на пользователя. После первого approve рабочая копия non-staff закрыта; add-task-set — отдельный pending payload set’а, не полный pack-draft.

**Альтернатива:** оставить drafts только для creator — отвергнуто (всё равно dual-path).

### D2 — Moderation requests

First publish: один request type «pack» (или объединённый answers+tasks) с одной кнопкой Approve = выставить live cards + tasks. Add-task-set: request type `tasks` / `task_set` на один set. Статус staff: pending | needs_revision (без terminal rejected). Author Cancel → closed.

### D3 — Staff lock

Колонка/строка `edit_locked_by` + `edit_locked_at` на pack. `POST/GET edit-lock` при входе в Edit; heartbeat или short TTL (например 2–5 мин) + unlock on leave. Concurrent Edit → 409. Save пишет сразу в live revision (published) или working copy (unpublished).

### D4 — Remove publish-ux extras

Удалить HTTP/UI: unpublish, republish, draft rebase/stale, last_live* если больше не нужны. Migrate: nullable columns harmless или drop в ensureContentTables.

### D5 — Support change-pack

Topic id kebab/snake в каноне API (например `change_pack`); i18n «Изменить набор карточек». Create body: `{ topic, body, packId }` когда topic=change_pack. Server валидирует pack в каталоге (has live). В первое сообщение / метаданные тикета — URL live pack (из `CLIENT_APP_URL` + hash route). Staff list показывает topic + pack ref.

### D6 — Client surfaces

- Unpublished: creator → полный editor + Submit
- Published non-staff: только «Добавить набор заданий» (отдельный flow/page), без списков карточек/set’ов в edit-режиме
- Staff: Edit на live/collection/catalog без Collect; без кнопки Submit на модерацию
- My-moderation: оставить список открытых заявок автора

## Risks / Trade-offs

- [Миграция чужих drafts] → потеря незапушенных правок; mitigation: один раз drop/archive drafts при деплое
- [Staff lock TTL] → забытый lock; mitigation: короткий TTL + staff может перебить только после expiry (не steal mid-session без expiry)
- [Единый Approve] → staff обязан проверить cards и tasks; mitigation: UI checklist / disable Approve если 0 task sets

## Migration Plan

1. Deploy server: schema working-copy + lock; stop writing `content_user_drafts`; migrate creator unpublished → working copy; delete draft rows
2. Deploy client: новые ACL/surfaces; убрать draft/stale/unpublish UI
3. Rollback: feature трудно откатить без возврата drafts — держать change атомарным в одном релизе client+server

## Open Questions

Нет (продуктовые D* закрыты в explore).

## Точки врезки

**Server** (`../happy-tourist-server/`):

- `src/db/schema.ts`, `src/lib/content.ts` — working copy, submit/approve/needs-revision/cancel, add-task-set, staff save, lock; удалить drafts/unpublish/rebase
- `src/app.config.ts` — content + lock endpoints
- `src/lib/support.ts` + routes — topic + packId
- `test/zz-contentPacks.test.ts`, `test/support.test.ts`

**Client** (`../happy-tourist.github.io/`):

- `src/stores/content.ts`, `src/stores/support.ts`
- Content* pages (collection/live/editor/tasks/staff/my-moderation) — ACL + add-only + staff lock
- `src/pages/SupportPage.vue` — topic + catalog select
- `src/i18n/en-US/index.ts`
- unit tests

Чеклист — `tasks.md`.
