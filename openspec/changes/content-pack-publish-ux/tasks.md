## 1. Server — schema и unpublish/republish пака

- [x] 1.1 Прочитать `design.md` (D1–D4), delta `specs/content/packs/spec.md` (SC-PACK-92…94), skills `server-locate-change-points`, `work-with-database`, `work-with-routes`; зафиксировать точки в `schema` / `lib/content` / `app.config`
- [x] 1.2 Добавить `last_live_revision_id` (и twin при необходимости) + migrate в `ensureContentTables`; обнуление live + сохранение last-live в `unpublishPack` — проверить колонками в schema/DB helper
- [x] 1.3 Реализовать staff `unpublishPack`: cancel open moderation, wipe all drafts, null live pointers, retain last-live; HTTP endpoint; verify SC-PACK-92 mocha
- [x] 1.4 Реализовать staff `republishPack` из last-live сразу в каталог; gate non-staff edit/submit при staff-unpublished; verify SC-PACK-93/94 mocha
- [x] 1.5 Запретить creator hard-delete пока есть lastLive без live (D4); verify 409/403 в mocha

## 2. Server — unpublish task set и stale draft pull

- [x] 2.1 Прочитать design D5–D6 и SC-PACK-90/91/95/96; skills `work-with-database`, `server-work-with-test`
- [x] 2.2 Staff `unpublishLiveTaskSet`: только при ≥2 live sets; пересобрать live; cancel all open mod; drafts intact; reject last set; HTTP; verify SC-PACK-95/96 mocha
- [x] 2.3 `GET draft`: `draftStale` когда base ≠ current live; `rebase`/`pull` endpoint с D8 id-правилами; verify SC-PACK-90/91 mocha
- [x] 2.4 Из sibling server: `npm test` (zz-contentPacks / новые SC) — зелёный для SC-PACK-92…96 и 90/91

## 3. Client — store, i18n, слоты и статусы

- [x] 3.1 Прочитать design D7–D8, SC-PACK-85…89; skills `client-locate-change-points`, `work-with-stores` (content), `work-with-localization`, `work-with-pages`
- [x] 3.2 Store: wrappers unpublish/republish/task-set-unpublish/rebase + `draftStale`; i18n published/unpublish/pull/confirm
- [x] 3.3 Live pack + staff tasks: слоты как чипы (SC-PACK-88/89); cascade outline без `bg-warning` (SC-PACK-85)
- [x] 3.4 Editor/Tasks subtitles: «Опубликовано» / «Одобрено» (SC-PACK-86/87)
- [x] 3.5 Vitest: статусы + слоты live/staff + cascade outline — SC-PACK-85…89; `npm test` зелёный по ним

## 4. Client — коллекция staff, pull banner, task-set unpublish

- [x] 4.1 Прочитать SC-PACK-90…96; skills `work-with-pages`, `work-with-forms` (confirm), `work-with-test`
- [x] 4.2 Collection: staff unpublish/republish рядом с Edit + confirm; скрыть Edit non-staff после unpublish (SC-PACK-92…94)
- [x] 4.3 Editor: banner + Подтянуть при `draftStale` (SC-PACK-90/91)
- [x] 4.4 Cards editor task-set row: staff «снять» справа при ≥2 sets (SC-PACK-95/96)
- [x] 4.5 Vitest UI/store для SC-PACK-90…96; из client: `npm run lint`, `npm run typecheck`, `npm test` — зелёные
