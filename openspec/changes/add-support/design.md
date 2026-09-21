## Context

См. `proposal.md` и delta specs `support/tickets`, `support/roles`.

Сейчас: JWT auth (registered + anonymous), HTTP `createEndpoint` + `auth.middleware()`, smtp.bz `sendEmail`, SQLite `colyseus_users` без `role`, нет support-таблиц и admin UI. AGENTS: «no separate admin API».

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`). Meta: при необходимости AGENTS Business Entities.

Чеклист реализации — `tasks.md` (не дублировать здесь).

## Goals / Non-Goals

**Goals:**

- HTTP CRUD тикетов + публичный тред; статусы; лимиты; автозакрытие 3 суток в `awaiting_response`.
- Письма автору при смене статуса через существующий mailer (не anonymous).
- Роли `user` | `moderator` | `admin`; `BOOTSTRAP_ADMIN_IDS`; admin users UI.
- Client: support pages + staff queue + admin users; ссылка в хедере лобби.
- Без Colyseus room/WS для support.

**Non-Goals:**

- Staff email notify; private notes; reopen; unauthenticated tickets; Colyseus `db.moderation` roles.

## Decisions

### D1: Transport = HTTP only

- Все операции через `client.http` + server `createEndpoint` + `auth.middleware()`.
- Не создавать Colyseus room для support (нагрузка рядом с `tourist`/`lobby` не нужна).
- Refresh: load on navigate; optional manual refresh. Realtime — out of scope.

### D2: Data model (SQLite / Drizzle)

- Extend `colyseus_users` with `role` text NOT NULL default `user` (`user` | `moderator` | `admin`) — same pattern as other custom columns with `.default(...)`.
- New tables (имена apply могут уточнить, смысл фиксирован):
  - `support_tickets`: id, authorUserId, topic, status, createdAt, updatedAt, awaitingSince (nullable; set when entering `awaiting_response`, clear otherwise).
  - `support_messages`: id, ticketId, authorUserId, body, createdAt, authorKind (`user` | `staff`).
- Topics: `problem` | `suggestion` | `feedback` | `question` | `other`.
- Statuses: `under_review` | `in_progress` | `awaiting_response` | `closed`.

### D3: HTTP surface (contract sketch)

Author (any JWT):

- `POST /api/support/tickets` `{ topic, body }` → create + first message
- `GET /api/support/tickets` → own list
- `GET /api/support/tickets/:id` → detail + messages (owner or staff)
- `POST /api/support/tickets/:id/messages` `{ body }` → author reply (open only); if was `awaiting_response` → `in_progress`
- `POST /api/support/tickets/:id/close` → author close

Staff (moderator|admin):

- `GET /api/support/staff/tickets` → all tickets (filter query optional)
- `POST /api/support/tickets/:id/take` → `under_review` → `in_progress`
- `POST /api/support/tickets/:id/status` `{ status }` → staff status changes (`awaiting_response` | `closed` | … with validation)
- `POST /api/support/tickets/:id/messages` as staff (same path; role decides authorKind) **или** отдельный staff message endpoint — apply выбирает один стиль, поведение: staff message + optional status

Admin only:

- `GET /api/admin/users` → list id, email, anonymous, role, displayName
- `POST /api/admin/users/:id/role` `{ role }` (POST, not PATCH — same style as other support mutations; SDK/test harness)

Reject unauthenticated; reject staff/admin actions for insufficient role (`403`).

### D4: Email on status change

- Reuse `src/lib/mailer.ts` `sendEmail`.
- Trigger when status changes (take, awaiting, close manual, auto-close, author close).
- Skip if author `anonymous` or no email.
- RU subjects/bodies; auto-close: отдельный смысл («не получен ответ в течение 3 суток»).
- Link: `{CLIENT_APP_URL}/#/support/<id>` (hash router).
- No staff notification emails.

### D5: Auto-close

- Clock: 3 × 24h from `awaitingSince` without author message after that timestamp.
- Implementation: lazy evaluate on GET list/detail + `setInterval` ~1h on server boot (no external cron).
- On auto-close: set `closed`, send mail if eligible (D4).

### D6: Rate limits

- 5 creates / user / UTC day; max 3 non-`closed` tickets; 30 messages / hour / ticket / user.
- Enforce server-side; client may mirror hints.

### D7: Roles + bootstrap

- Env `BOOTSTRAP_ADMIN_IDS` comma-separated user ids (same DB as runtime).
- On listen/startup: for each id, `UPDATE … SET role='admin'` idempotent (env wins demotion).
- Document in `.env.example`.
- JWT userdata SHOULD expose `role` for client gating (extend userdata / profile read path used today); client MUST NOT trust role alone for security — server enforces.

### D8: Client structure

- Routes (`requiresAuth`): support list/create, support detail, staff tickets (moderator+), admin users (admin only).
- Pages under `src/pages/` (e.g. SupportPage, SupportTicketPage, SupportStaffPage, AdminUsersPage — exact names in apply).
- Pinia store for support HTTP (не размазывать `client.http` по UI) — follow `work-with-stores` / `colyseus-client`.
- Lobby header: Support link (authenticated, incl. anonymous) — LobbyPage / App header as fits existing layout; explore: «в хедере лобби».
- Guest create: banner warning no email notify.
- Closed: UI read-only; CTA to create new.
- i18n RU for topics/statuses/warnings/errors; `q-form` rules; errors via store + `q-banner`.
- Display name for anonymous in thread: «Гость».

### D9: Auth vs guest

- `requiresAuth` for support routes (same as lobby) — anonymous OK.
- Unauthenticated → login redirect (existing guard).

### D10: Skills / AGENTS

- Update sibling AGENTS Business Entities: Support + roles.
- Prefer existing skills (`work-with-routes`, `work-with-database`, `client-work-with-structure`, `work-with-pages`, `work-with-forms`); new topic skill только если apply упрётся — не блокер proposal.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Bootstrap id from wrong DB (local vs prod) | Document: take JWT id on same env as env file |
| Enumerate users via admin list | Admin-only; accepted |
| Lazy auto-close delay if nobody opens ticket | Periodic interval backup |
| Role only in DB, stale JWT | Refresh userdata on app load / after role change; server checks DB role on each request |
| SQLite growth of messages | Fine for v1; pagination later if needed |

## Migration Plan

1. Server: schema + endpoints + bootstrap + tests → deploy with empty `BOOTSTRAP_ADMIN_IDS` then set owner id.
2. Client: pages + store + lobby link.
3. Ops: set `BOOTSTRAP_ADMIN_IDS` on VPS; restart PM2.
4. Rollback: feature flags not required; revert deploys independently (client without API fails soft with banner).

## Open Questions

(нет — explore закрыт; apply defaults выше.)
