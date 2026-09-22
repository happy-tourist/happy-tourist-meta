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
