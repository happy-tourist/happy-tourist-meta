## 1. Server — schema, roles bootstrap, support HTTP



- [x] 1.1 Прочитать `proposal.md`, `design.md` (D1–D7), specs `support/tickets` + `support/roles`; skills `.agents/skills/server/work-with-database/SKILL.md`, `work-with-routes`, `server-work-with-auth`, `work-with-config`, `server-work-with-structure`; сверить `../happy-tourist-server/src/db/schema.ts`, `app.config.ts`, `lib/mailer.ts`, `.env.example`



- [x] 1.2 Добавить `role` на users (default `user`); таблицы `support_tickets` + `support_messages` по design D2; регистрация/login не падают на NOT NULL



- [x] 1.3 `BOOTSTRAP_ADMIN_IDS` в `.env.example` + идемпотентный bootstrap admin при старте (SC-ROLE-02); userdata отдаёт `role`



- [x] 1.4 HTTP author API: create / list own / detail / message / close; валидация topic+body; anonymous OK; unauthenticated 401/403 (SC-SUP-01/02/03/05/06/14/19/20)



- [x] 1.5 Rate limits: 5 creates/day, ≤3 open, 30 msg/hour/ticket (SC-SUP-16)



- [x] 1.6 Staff API: list all, take, status, staff message; только moderator|admin (SC-SUP-08/09/17, SC-ROLE-06)



- [x] 1.7 Автор reply из `awaiting_response` → `in_progress`; closed rejects messages (SC-SUP-07/15)



- [x] 1.8 Письма автору при смене статуса через `sendEmail` (RU; auto-close отдельный текст; ссылка `#/support/:id`); skip anonymous (SC-SUP-10/11/13)



- [x] 1.9 Автозакрытие: lazy + periodic ~1h; 3 суток от `awaitingSince` (SC-SUP-12)



- [x] 1.10 Admin API: list users + PATCH role; только admin; moderator cannot assign (SC-ROLE-03/04/05/07)



- [x] 1.11 Mocha: SC-SUP-* и SC-ROLE-* server scenarios с mock mailer; в `../happy-tourist-server` — `npm test`; при падении починить



## 2. Client — support UI + lobby link + admin users



- [x] 2.1 Прочитать skills `.agents/skills/client/client-work-with-structure/SKILL.md`, `work-with-pages`, `work-with-forms`, `work-with-stores`, `work-with-localization`, `colyseus-client`, `client-work-with-errors`; `design.md` D8–D9; routes, `LobbyPage` / App header, `stores/auth.ts`



- [x] 2.2 Pinia support store: HTTP create/list/detail/message/close/staff/admin; loading/error + q-banner pattern



- [x] 2.3 Routes + pages: support (create+my list), ticket detail thread, staff all-tickets; `requiresAuth`; guest warning no-email (SC-SUP-04/18)



- [x] 2.4 Staff UI: take / awaiting / close / reply; closed read-only + CTA new ticket



- [x] 2.5 Admin users page: list + change role; UI только admin (SC-ROLE-08); moderator без role UI



- [x] 2.6 Ссылка «Поддержка» в хедере лобби для любого JWT (в т.ч. anonymous) (SC-SUP-18)



- [x] 2.7 i18n RU: темы, статусы, предупреждение гостя, ошибки лимитов; display «Гость» для anonymous в треде



- [x] 2.8 Auth store: читать `role` из userdata для gating навигации (server всё равно enforce)



- [x] 2.9 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении починить



## 3. Meta — AGENTS / skills hints



- [x] 3.1 Обновить sibling AGENTS (client + server): домен Support, роли, HTTP support/admin; убрать/ослабить «no admin API» где устарело



- [x] 3.2 При необходимости точечно дополнить `work-with-routes` / `work-with-database` / `work-with-pages` упоминанием support endpoints (без отдельного skill, если не требуется)



## 4. Server — polish (create mail, filters, admin list)



- [x] 4.1 Прочитать обновлённые `design.md` (D3/D4/D11), specs SC-SUP-21/22, SC-ROLE-07; `lib/support.ts`, admin/staff routes, `support.test.ts`



- [x] 4.2 Ack-письмо при create (RU, отличный текст от status/auto-close; ссылка `#/support/:id`); skip anonymous; mock mailer тест SC-SUP-21



- [x] 4.3 Staff list query: `topic` (optional), `status`=`open`|`closed`|`all` (default `open`); cap после фильтра; тест SC-SUP-22



- [x] 4.4 Admin `GET /api/admin/users`: исключить anonymous; отдавать `emailVerified`; тест SC-ROLE-07



- [x] 4.5 В `../happy-tourist-server` — `npm test`; при падении починить



## 5. Client — polish (filters, admin badge, form/thread/guest warn)



- [x] 5.1 Прочитать design D8; skills `work-with-forms`, `work-with-pages`, `work-with-localization`; Support*/AdminUsers pages + support store



- [x] 5.2 Staff UI: фильтры тема (default все) + статус open|closed|all (default open); refresh списка (SC-SUP-23)



- [x] 5.3 Admin users: бейдж/тег «почта не подтверждена» при `emailVerified === false` (SC-ROLE-09); список без гостей (сервер)



- [x] 5.4 Guest warning: нет писем + риск потери доступа при смене сессии (SC-SUP-04); i18n RU



- [x] 5.5 Create/reply: после успеха clear + `resetValidation` (SC-SUP-24)



- [x] 5.6 Тред: видимый отступ между сообщениями (SC-SUP-25)



- [x] 5.7 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении починить



## 6. Meta — polish hints (if needed)



- [x] 6.1 При необходимости точечно обновить skills/AGENTS: create-ack mail, staff query filters, admin list non-anonymous + emailVerified badge



## 7. Client — reply UX (red empty + thread→form gap)



- [x] 7.1 Прочитать design D8/D12, SC-SUP-24/26; `SupportTicketPage.vue`, `SupportPage.vue`; skill `work-with-forms`



- [x] 7.2 Create/reply: после успеха clear → `await nextTick()` → `resetValidation`; `lazy-rules` на body (и create) inputs — без красного empty (SC-SUP-24)



- [x] 7.3 Тред: видимый отступ между блоком сообщений и полем «Ответ» (SC-SUP-26)



- [x] 7.4 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении починить



- [x] 7.5 При необходимости точечно обновить `work-with-forms`: lazy-rules + nextTick перед resetValidation после clear

