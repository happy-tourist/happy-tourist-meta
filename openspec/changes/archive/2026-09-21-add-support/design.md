## Context

См. `proposal.md` и delta specs `support/tickets`, `support/roles`.

v1 уже в runtime: JWT support HTTP, `ht_role`, таблицы тикетов/сообщений, staff/admin API, mail on status change, client pages. Этот revision — polish по feedback после первого прогона.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`). Чеклист — `tasks.md` (секции 4+; UX reply — секция 7).

## Goals / Non-Goals

**Goals:**

- HTTP CRUD тикетов + тред; статусы; лимиты; автозакрытие 3 суток.
- Письма автору: **ack при create** + при смене статуса (не anonymous).
- Роли + bootstrap; admin list **без anonymous**; бейдж неподтверждённой почты.
- Staff queue filters (topic + status open/closed/all).
- Client UX: guest warn (mail + session); form clear без ложного error-state; spacing сообщений **и** тред→поле ответа.
- Без Colyseus room для support.

**Non-Goals:**

- Staff email notify; private notes; reopen; unauthenticated tickets; guest DB purge/TTL; Colyseus moderation roles.

## Decisions

### D1: Transport = HTTP only

- `client.http` + `createEndpoint` + `auth.middleware()`.
- Refresh on navigate; optional manual refresh. Realtime out of scope.

### D2: Data model (SQLite / Drizzle)

- `htRole` / `ht_role` on users (not JS name `role`).
- `support_tickets`, `support_messages` as shipped.
- Topics / statuses unchanged.

### D3: HTTP surface

Author: create / list own / detail / message / close.

Staff:

- `GET /api/support/staff/tickets` — query filters: `topic` (optional enum | omit=all), `status` (`open` | `closed` | `all`; default **`open`** = not `closed`). Keep response cap (~50) **after** filter.
- take / status / messages as shipped (closed terminal — no reopen).

Admin:

- `GET /api/admin/users` — **exclude** `anonymous === true`; include `emailVerified` (and existing id/email/role/displayName). Google / email-registered included.
- `POST /api/admin/users/:id/role` `{ role }`.

### D4: Email — status change **and** create ack

- Reuse `sendEmail`.
- **Create ack:** after successful ticket create, if author non-anonymous with email → RU mail «обращение получено» + link `#/support/<id>`. Distinct copy from status-change / auto-close.
- **Status change:** take, awaiting, close manual, auto-close, author close (unchanged).
- Skip anonymous / no email. No staff notification emails.

### D5: Auto-close

- 3 × 24h from `awaitingSince`; lazy + ~1h interval (unchanged).

### D6: Rate limits

- 5 creates/day; ≤3 open; 30 msg/hour/ticket (unchanged).

### D7: Roles + bootstrap

- `BOOTSTRAP_ADMIN_IDS`; userdata `role` (unchanged).

### D8: Client structure

- Existing Support* / AdminUsers pages + store.
- Guest banner: no email **and** without the same session may not see replies / history (i18n RU).
- Staff page: topic filter (default all) + status filter (default open).
- Admin page: list from filtered API; badge/chip when `emailVerified === false`.
- Forms create/reply: after successful submit clear fields; **must not** leave empty field in Quasar error/red state. Mere `resetValidation()` right after `v-model = ''` is insufficient (rules re-fire on model update). Use: `lazy-rules` on body (and create) inputs + clear → `await nextTick()` → `q-form.resetValidation()` (same pattern on `SupportPage` create and `SupportTicketPage` reply).
- Ticket thread: visible spacing between messages / author line and body; **and** visible gap between the message list block and the reply composer when the form is shown (Quasar spacing / margin — not flush borders).
- Closed read-only + CTA new ticket.

### D9: Auth vs guest

- `requiresAuth`; anonymous OK for support routes (unchanged).

### D10: Skills / AGENTS

- Already updated for v1; polish: mention staff query filters, create-ack mail, admin non-anonymous list if skills list endpoints.
- UX fix: if `work-with-forms` mentions support create/reply, note `lazy-rules` + `nextTick` before `resetValidation` after clear.

### D11: No guest purge (explore)

- Anonymous rows may remain in SQLite; product fix = hide from admin list. TTL delete — out of scope.

### D12: Reply form UX regression (explore 2026-09-21)

- Observed: after successful reply, empty «Ответ» stays red; thread card sits flush against reply input.
- Fix scope: client only (`SupportTicketPage`, optionally `SupportPage` create for the same race); no API/schema change.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Create + status emails feel noisy | Distinct ack copy; only one mail on create |
| Filtered staff list empties with cap | Cap after filter; defaults open + all topics |
| Admin hides guests but tickets still from guests | Staff queue still shows guest tickets as «Гость» |
| Unverified badge without server field | API already can expose `emailVerified` from users row |
| `resetValidation` alone leaves red empty | `lazy-rules` + `nextTick` before reset |

## Migration Plan

1. Deploy server (filters + create mail + admin filter) → client polish → client reply UX fix (section 7).
2. No new env vars.
3. Rollback: independent sibling deploys.

## Open Questions

(нет — explore polish + UX reply закрыты: D1 hide guests + unverified badge; D2 create ack; Google in admin list; no purge; D12 Quasar clear/reset + thread→form gap.)
