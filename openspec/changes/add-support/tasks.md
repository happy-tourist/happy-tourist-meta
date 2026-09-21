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
