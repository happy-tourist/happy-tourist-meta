## 1. Server — DB + content HTTP API

- [x] 1.1 Прочитать `proposal.md`, `design.md` (D1–D9), `specs/content/packs/spec.md`; skills `.agents/skills/server/work-with-database/SKILL.md`, `work-with-routes`, `server-work-with-auth`, `server-work-with-errors`, `server-work-with-structure`, `work-with-config`; сверить аналоги `support` в `../happy-tourist-server` (`db/schema.ts`, `lib/support.ts`, `lib/mailer.ts`, `app.config.ts`)

- [x] 1.2 Добавить SQLite-таблицы контента (packs, live/draft revisions, answer cards, task sets, tasks, slots, collections, moderation requests/messages, blocked) + `ensureContentTables` при старте по design D2; регистрация/login не ломаются

- [x] 1.3 HTTP catalog + pack live GET: только approved non-secret live; blocked видны с флагом (SC-PACK-13/14/25)

- [x] 1.4 HTTP collection: list / add / remove; JWT включая anonymous; unauthenticated 401 (SC-PACK-11/12)

- [x] 1.5 HTTP create pack: только non-anonymous + emailVerified; pack в коллекцию автора; не в каталог до approve (SC-PACK-01/02/03)

- [x] 1.6 HTTP draft get/put: edit только если pack в коллекции + verified + не чужой pending lock; слоты/difficulty/answer cards; смена content referenced card очищает слоты (SC-PACK-04/05/06/07/10/15)

- [x] 1.7 HTTP submit: минимумы ≥2 cards, ≥2 tasks, ≥1 filled slot; один pending на pack; pending-автор может resubmit; race → ошибка + сохранение чужого draft (SC-PACK-08/09/16/17)

- [x] 1.8 HTTP moderation thread: сообщения change author ↔ staff; чужие не читают (SC-PACK-23/24)

- [x] 1.9 Staff HTTP: list pending, preview, approve, reject+comment, cancel, block/unblock; только moderator|admin (SC-PACK-18/19/20/21/25/26); approve → live + catalog + co-author label display; новый цикл → новый thread (SC-PACK-22)

- [x] 1.10 Письма автору заявки (approve / reject / staff message / block) через `sendEmail`; skip anonymous; SPA hash links (SC-PACK-27/28)

- [x] 1.11 Mocha: покрытие SC-PACK-* server scenarios (+ mock mailer); в `../happy-tourist-server` — `npm test`; при падении починить; обновить Traceability в delta-spec на covered

## 2. Client — catalog, collection, editor, staff

- [x] 2.1 Прочитать skills `.agents/skills/client/client-work-with-structure/SKILL.md`, `work-with-pages`, `work-with-forms`, `work-with-stores`, `work-with-localization`, `client-work-with-errors`, `client-work-with-auth`; `design.md` D3/D10; аналоги Support* pages + `stores/auth.ts` (`emailVerified`, `role`)

- [x] 2.2 Pinia content/packs store: весь HTTP I/O; loading/error + q-banner

- [x] 2.3 Routes + pages: каталог, коллекция, pack view (live), pack editor (slots UX +/− / fill / clear), create entry; `requiresAuth`; modal login/verify для ineligible create/edit (SC-PACK-29/31)

- [x] 2.4 Staff pages: очередь pending + preview + approve/reject/cancel/block + thread; UI только moderator|admin (SC-PACK-30)

- [x] 2.5 Навигация в header/lobby к разделу наборов; i18n RU (статусы, blocked, сложности 1–3, ошибки submit/lock)

- [x] 2.6 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении починить; обновить Traceability client SC в delta-spec

## 3. Meta — AGENTS / skills

- [x] 3.1 Прочитать `.agents/AGENTS.md`, sibling `AGENTS.md` client/server; skills `work-with-routes` / `work-with-pages` / `work-with-database` как точки документирования

- [x] 3.2 Обновить Business Entities / HTTP surface hints под `content/packs` (без дублирования полного spec); при необходимости краткий указатель в meta `.agents/AGENTS.md`

## 4. Server — dual submit / lock / staff hub

- [x] 4.1 Прочитать обновлённые `proposal.md`, `design.md` (D2/D4/D5/D9/D13), `specs/content/packs/spec.md` (SC-PACK-07…09, 15…20, 34…37, 39…41); сверить текущий `lib/content.ts` + `zz-contentPacks.test.ts`

- [x] 4.2 Добавить `type` (`answers`|`tasks`) на moderation requests + dirty/last-submitted answers snapshot; миграция/ensure; один pending на `(pack, type)`; verify schema boot и что login не ломается

- [x] 4.3 Разделить submit: `submit/answers` (≥2 cards; не блокируется пустыми слотами; **без** требования live|pending tasks) и `submit/tasks` (≥2 tasks, filled slots, difficulty; reject без cards / при dirty answers); verify SC-PACK-08/09/35 mocha

- [x] 4.4 Lock dirty-answers: deny create/edit tasks пока answers dirty; unlock после submit answers; tasks pending race + resubmit автора; verify SC-PACK-15/16/17/36

- [x] 4.5 Staff list только answers-pending; hub preview answers + nested tasks; approve answers только при live tasks; approve tasks → answers → catalog; verify SC-PACK-18/30/37/39/40 и `npm test`

- [x] 4.6 Обновить mail/thread на type-scoped requests; Traceability pending→covered для server SC; verify `npm test` зелёный

## 5. Client — split editor + nav + autosave

- [x] 5.1 Прочитать design D10/D13 и skills pages/stores/forms/localization; сверить Content* pages + `stores/content.ts`

- [x] 5.2 Pinia: dual submit endpoints, pending flags per type, `answersDirty`; loading/error + q-banner; **без** badge state; verify typecheck на store

- [x] 5.3 Answers page: title/description, одна форма + список cards (edit icon), nested task-set list (author/coauthor), submit answers; create → redirect answers; confirm delete; autosave draft; нет badge; verify SC-PACK-01/34/41 lint

- [x] 5.4 Task-set nested page: форма вопроса, слоты +/− (min 1), тайлы ответов, список вопросов, submit tasks; gate без cards и при dirty answers (disable + i18n); verify SC-PACK-05/07/09/36 UX

- [x] 5.5 Lobby → collection first; catalog link from collection; staff answers hub + nested tasks approve order; i18n RU; verify SC-PACK-30/41

- [x] 5.6 `npm run lint` + `npm run typecheck` в client; Traceability client SC → covered; починить падения

## 6. Meta — docs after dual flow

- [x] 6.1 Обновить sibling AGENTS + skills (routes/pages/stores/database/tests) под dual answers/tasks; краткий указатель в meta `.agents/AGENTS.md`; verify тексты без полного dump spec

## 7. Server — status / tasksDirty / pending locks

- [x] 7.1 Прочитать обновлённые `proposal.md`, `design.md` (D5′/D15–D20), `specs/content/packs/spec.md` (SC-PACK-42…50); сверить `lib/content.ts` + `zz-contentPacks.test.ts`

- [x] 7.2 Last tasks snapshot + pack `tasksDirty` + per-task-set `needsModeration` flags в draft GET (F5-safe); verify schema/ensure и login не ломаются

- [x] 7.3 Draft GET: статусы answers/tasks (`pending`|`rejected`|`approved`|none) + requestId для открытого треда; verify payload

- [x] 7.4 Enforce: только answers-pending author → submit/answers; другие не submit/tasks при answers pending; author A MAY submit/tasks; verify SC-PACK-42/43 mocha

- [x] 7.5 Staff preview: task-set list marks без полного expand; nested tasks route data; Traceability 42–44 → covered; `npm test` зелёный

## 8. Client — cards UX, threads, staff nested

- [x] 8.1 Убрать top «Сохранение…»; loading на кнопках; submit disabled без dirty/minima; tooltip create task set; add question требует filled slot; i18n «Набор карточек»; verify SC-PACK-46/47/50 lint

- [x] 8.2 Статусы answers/tasks на страницах карточек и заданий; embedded threads + reply; reject comment виден; per-set needs-moderation marks; verify SC-PACK-45/48/49

- [x] 8.3 Staff hub = list task sets → nested tasks page; approve/reject answers на hub, tasks на nested; verify SC-PACK-44

- [x] 8.4 Lock UI: чужой не сабмитит answers/tasks при чужом answers pending; `npm run lint` + `typecheck`; Traceability client SC → covered

## 9. Meta — hints after UX polish

- [x] 9.1 Обновить sibling AGENTS + skills (pages/stores/routes/localization) под «Набор карточек», статусы, threads, staff nested; verify без полного dump spec

## 10. Server — marks fix, D1′ lock, author delete

- [x] 10.1 Прочитать обновлённые `proposal.md`, `design.md` (D1′/D21–D26), `specs/content/packs/spec.md` (SC-PACK-51…60); сверить `lib/content.ts` + `zz-contentPacks.test.ts`

- [x] 10.2 Fix sticky needsModeration after approve (rewrite snapshots on detach and/or stable ids in `copyRevision`); verify SC-PACK-60; login не ломается

- [x] 10.3 D1′: dirty without pending → block tasks for all; answers-pending author A MAY create/edit tasks while dirty; others blocked; verify SC-PACK-15/36/57/58 mocha

- [x] 10.4 Author delete unpublished pack (createdBy only) + cascade moderation/collections; delete task set syncs draft + liveTasks; reject non-creator / published; verify SC-PACK-55/56; `npm test`

## 11. Client — staff nav, collection gates, delete UX, hide block

- [x] 11.1 Staff: после approve tasks → answers hub; list без промодерированных sets; убрать «Открыть задания»; нет `pack_not_public` dead-end; verify SC-PACK-51/52/44

- [x] 11.2 Убрать Edit с live pack page; Edit только из коллекции; remove-from-collection с confirm; verify SC-PACK-53/54

- [x] 11.3 D1′ UI lock на tasks; author delete pack / task set (unpublished) с confirm; скрыть block/unblock; verify SC-PACK-57/58/55/56/59

- [x] 11.4 `npm run lint` + `typecheck`; Traceability SC-PACK-51…60 → covered

## 12. Meta — hints after follow-up

- [x] 12.1 Обновить sibling AGENTS + skills (pages/stores/routes/localization/database) под collection-only edit, confirm remove, author delete unpublished, D1′, staff redirect, no block UI; verify без полного dump spec
