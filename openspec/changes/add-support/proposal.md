## Why

Игрокам (включая гостей) нужен канал связи с командой: баг, предложение, отзыв, вопрос — без сокета и вне игровых комнат. После первой поставки support: в admin-списке мешают anonymous-гости; при создании тикета нет ack-письма; staff-очередь без фильтров; мелкие UX-дефекты формы и треда.

## What Changes

- Раздел поддержки: создание обращения (тема + текст), список своих, страница треда с ответами.
- Статусы: на рассмотрении, в работе, ожидает ответа, закрыто; автозакрытие через 3 суток в «ожидает ответа».
- Staff: очередь с фильтрами тема/статус; взять в работу, ответить; только админ назначает роли.
- Первый админ — id из env; письма автору при **создании** (ack) и при смене статуса; гость — предупреждение (нет писем + потеря сессии).
- Admin users: только не-гости (email/Google); бейдж «почта не подтверждена».
- Ссылка «Поддержка» в хедере лобби; HTTP only.

## Capabilities

### New Capabilities

- `support/tickets`: обращения, тред, статусы, лимиты, письма автору (create + status), UI автора и staff-очереди с фильтрами.
- `support/roles`: роли `user` / `moderator` / `admin`, bootstrap admin из env, список пользователей (без anonymous) и смена ролей (только admin).

### Modified Capabilities

- (нет отдельных main-capability вне этого change до sync)

## Scope

- **Capability ID:** `support/tickets`, `support/roles`
- **Пакеты:** client + server (+ meta AGENTS/skills при необходимости)
- **Кто:** любой с JWT, в т.ч. anonymous-гость; полностью без сессии — нет
- **Темы:** проблема, предложение, отзыв, вопрос, другое
- **HTTP:** REST support API + JWT; комнаты Colyseus не затрагиваются
- **Почта:** smtp.bz → автору с email при **создании** (получили) и при **смене статуса**; anonymous — без писем
- **Роли / admin list:** `BOOTSTRAP_ADMIN_IDS`; admin UI — только зарегистрированные (не anonymous), вкл. Google; индикация неподтверждённой почты
- **Staff queue:** фильтр по теме (default все) и статусу open|closed|all (default open = не closed)
- **UX polish:** сброс валидации формы после успешной отправки; отступ между сообщениями в треде; усиленное guest-предупреждение
- **Навигация:** «Поддержка» в хедере лобби

## Out of scope

- Colyseus / WebSocket realtime для support
- Письма staff при новом тикете или ответе автора
- Внутренние (staff-only) заметки
- Тема «жалоба на игрока»
- Обращения без JWT
- Contact email в форме для гостя
- Переоткрытие закрытого обращения
- Отдельный CMS / внешний helpdesk
- Colyseus `db.moderation` / `colyseus_roles` как канон ролей
- **Purge / TTL-удаление anonymous-строк из БД** (гости просто не в admin list)
- Правила игры, lobby listing sync, board

## Impact

- Client: support + staff filters + admin users (без гостей, бейдж verify); i18n; UX формы/треда.
- Server: create ack email; admin list filter; staff list query filters; тесты.
- Ops: `BOOTSTRAP_ADMIN_IDS` без изменений смысла.
- Meta: при необходимости точечно skills/AGENTS.

## References

- Explore 2026-09-21 (v1): anonymous-гость; warn; roles; awaiting; limits; mail on status; env bootstrap; no staff mail; closed read-only
- Explore polish 2026-09-21: D1 admin = non-anonymous + unverified badge (no guest purge); D2 mail on create ack; staff filters; form resetValidation; thread spacing; guest warn session loss
- Sibling AGENTS: client/server Support domain
- Почта: `openspec/specs/auth/email-verification`, `auth/password-reset`
