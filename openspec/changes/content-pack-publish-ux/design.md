## Context

См. `proposal.md` — Why. Канон: dual answers|tasks, personal drafts per (user, pack), live = `liveRevisionId` / `liveTasksRevisionId`, staff queue, cascade slots (SC-PACK-07/78–81), editor task rows already show slots (SC-PACK-84). Публичный live (`ContentPackPage`) всё ещё показывает `slotsCount`; staff tasks — text join; `approved` → «Одобрено»; `detachAuthorDraftIfPinned` только у change author; block API есть, UI скрыт (SC-PACK-59).

Пакеты: **server** (`happy-tourist-server`) + **client** (`happy-tourist.github.io`). Room/tourist не затрагивается.

## Goals / Non-Goals

**Goals:**

- HTTP + UI: pack unpublish / republish; task-set unpublish из live; stale-draft detect + pull (D8)
- UI: слоты на live/staff; «Опубликовано»/«Одобрено»; cascade — outline без fill
- Сохранить last-live снимок при wipe drafts на pack-unpublish

**Non-Goals:**

- Ужесточение foreign pending lock
- Block UI
- Новые npm-зависимости / внешние сервисы

## Decisions

### D1 — Last-live snapshot columns

При pack-unpublish: сохранить текущий merged live revision id в `last_live_revision_id` (и при необходимости `last_live_tasks_revision_id` = тот же merged, если один snapshot). Обнулить `live_revision_id` / `live_tasks_revision_id`. Не удалять строку revision payload — иначе republish (D9) невозможен.

**Альтернатива:** отдельная copy revision «archive» — дороже, не нужно если payload остаётся.

### D2 — Pack unpublish wipe

В одной staff-транзакции: cancel/close все open `content_moderation_requests` (pending|rejected) для пака; удалить все `content_user_drafts` пака; null live pointers; записать last-live. Email опционально (не блокер; можно кратко staff→без mail в этом change).

### D3 — Republish

Staff-only HTTP: выставить `liveRevisionId` / `liveTasksRevisionId` из last-live **без** новой moderation request. После успеха авторы снова проходят обычный `ensureUserDraft` от live.

### D4 — Unpublished edit gate

`hasLive === false` после staff-unpublish отличается от never-published creator draft: флаг/поле `unpublishedByStaff` или «есть lastLive и нет live». Non-staff: Edit скрыт, `getDraft`/`putDraft`/`submit` → 403. Staff с паком в коллекции: Edit разрешён. Creator hard-delete unpublished (SC-PACK-55) — **не** расширять на staff-unpublish без lastLive wipe; при наличии lastLive creator delete остаётся 409 `pack_published`-семантикой или отдельный запрет — в apply: запретить creator hard-delete пока есть lastLive (пак «снят», не «черновик автора»).

### D5 — Task-set unpublish

Staff-only: пересобрать live revision без выбранного task set (copy/merge write); отвергнуть если live task sets count < 2. Cancel **все** open moderation requests пака. Drafts не трогать. UI: кнопка справа в строке task set на cards editor (как список наборов).

### D6 — Stale draft + pull

`GET draft` возвращает `draftStale: true` когда personal draft base не соответствует текущему live (сравнивать fingerprint/revision ref: например `basedOnLiveRevisionId` на draft row vs `pack.liveRevisionId`, или structural diff answers+tasks vs live). Client: banner + «Подтянуть». `POST .../draft/rebase` (или query на put): загрузить live → copy в новый draft revision → наложить локальный diff: карточки/задания, отличающиеся от live по id+content, клонировать с **новыми id**; слоты перепривязать к новым card ids где нужно. Обновить snapshots so dirty flags отражают сохранённый diff.

### D7 — Status copy

i18n: `taskSetStatusMarks.approved` для answers+hasLive → ключ published «Опубликовано»; tasks without hasLive → «Одобрено». Логика в computed subtitle (Editor/Tasks pages).

### D8 — Slots + cascade chrome

Live `ContentPackPage`: чипы слотов как на TasksPage (`slotLabel`). StaffTasksPage: те же чипы вместо `join(' · ')`. Cascade: убрать `bg-warning` с строк; outline/border warning на строке и пустых слотах (SC-PACK-85).

### D9 — Explore prerequisites (закрыты)

D1–D18 из explore закрыты продуктово; технических внешних блокеров нет. Реализация — внутри Vue/Quasar/Colyseus HTTP/Drizzle/SQLite.

## Risks / Trade-offs

- [Wipe drafts on pack unpublish] → безвозвратная потеря локальных правок; mitigation: confirm dialog на client
- [Task-set unpublish cancels answers pending too] → авторы теряют очередь; mitigation: confirm copy
- [Stale rebase id remaps] → сложность слотов; mitigation: server-authoritative rebase, client reload draft after
- [Creator delete vs staff-unpublish] → путаница unpublished; mitigation: D4 lastLive gate

## Migration Plan

- SQLite: `ALTER` columns `last_live_revision_id` (+ optional tasks twin) via existing `ensureContentTables` pattern
- Старые паки: lastLive null; unpublish/republish no-op until first unpublish
- Rollback: feature-flag не нужен; revert deploy + nullable columns harmless

## Open Questions

Нет (продуктовые D* закрыты в explore).

## Точки врезки

**Server** (`../happy-tourist-server/`):

- `src/db/schema.ts` + `src/lib/content.ts` — columns, `unpublishPack`, `republishPack`, `unpublishLiveTaskSet`, draft stale + rebase, edit gates
- `src/app.config.ts` — endpoints staff unpublish/republish, task-set unpublish, draft rebase
- `test/zz-contentPacks.test.ts` — SC-PACK-85…96 (где server)

**Client** (`../happy-tourist.github.io/`):

- `src/stores/content.ts` — API wrappers, stale flags
- `src/pages/ContentCollectionPage.vue` — staff unpublish/republish
- `src/pages/ContentPackPage.vue` — slot chips
- `src/pages/ContentPackEditorPage.vue` / `ContentPackTasksPage.vue` — status, banner/pull, cascade outline, staff task-set unpublish
- `src/pages/ContentStaffTasksPage.vue` — slot chips
- `src/i18n/en-US/index.ts` — published / unpublish / pull strings
- unit tests per Traceability

Чеклист реализации — `tasks.md`.
