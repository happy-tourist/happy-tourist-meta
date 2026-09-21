## Why

Игрокам (включая гостей) нужен канал связи с командой: баг, предложение, отзыв, вопрос — без сокета и вне игровых комнат. Сейчас в продукте нет support/тикетов и ролей staff; письма smtp.bz уже есть для auth и могут уведомлять автора о смене статуса.

## What Changes

- Раздел поддержки: создание обращения (тема + текст), список своих, страница треда с ответами.
- Статусы: на рассмотрении, в работе, ожидает ответа (уточнение от staff — ждём пользователя), закрыто; автозакрытие через 3 суток без ответа в «ожидает ответа».
- Staff (модератор / админ): очередь всех обращений, взять в работу, ответить в публичном треде; только админ назначает роли.
- Первый админ — id из env (идемпотентно); письма автору при смене статуса (гость — предупреждение без писем).
- Ссылка «Поддержка» в хедере лобби; HTTP only (без Colyseus room).

## Capabilities

### New Capabilities

- `support/tickets`: обращения, тред, статусы, лимиты, письма автору, UI автора и staff-очереди.
- `support/roles`: роли `user` / `moderator` / `admin`, bootstrap admin из env, список пользователей и смена ролей (только admin).

### Modified Capabilities

- (нет — auth/login и lobby/rooms требования не меняются на уровне spec; навигация в лобби — часть `support/tickets`)

## Scope

- **Capability ID:** `support/tickets`, `support/roles`
- **Пакеты:** client + server (+ meta AGENTS/skills при необходимости)
- **Кто:** любой с JWT, в т.ч. anonymous-гость; полностью без сессии — нет
- **Темы:** проблема, предложение, отзыв, вопрос, другое
- **HTTP:** REST support API поверх существующего JWT middleware; комнаты Colyseus не затрагиваются
- **Почта:** существующий smtp.bz → письма только автору с email при смене статуса (включая закрытие и автозакрытие)
- **Роли:** поле роли на пользователе; `BOOTSTRAP_ADMIN_IDS`; UI назначения — только admin
- **UX:** форма + мои обращения + деталь; staff — все обращения + admin users; закрытый тред read-only; новое = новое обращение
- **Навигация:** пункт «Поддержка» в хедере лобби

## Out of scope

- Colyseus / WebSocket realtime для support
- Письма staff при новом тикете или ответе автора
- Внутренние (staff-only) заметки
- Тема «жалоба на игрока»
- Обращения без JWT (полностью неавторизованный посетитель)
- Contact email в форме для гостя (только предупреждение)
- Переоткрытие закрытого обращения
- Отдельный CMS / внешний helpdesk (Zendesk и т.п.)
- Встроенный Colyseus `db.moderation` / `colyseus_roles` как канон ролей
- Правила игры, lobby listing sync, board

## Impact

- Client: страницы support + admin users; HTTP через Pinia; i18n RU; ссылка из лобби.
- Server: таблицы тикетов/сообщений; поле роли; HTTP endpoints; reuse `sendEmail`; bootstrap env; lazy/periodic автозакрытие.
- Ops: `BOOTSTRAP_ADMIN_IDS` в server env (prod/local — свой id из той же БД).
- Meta: при необходимости обновить AGENTS (новый домен support).

## References

- Explore 2026-09-21: D1 anonymous-гость; D2 warn без mail; D3 roles; D4 awaiting = ответ пользователя; D5 rate limits; D6 mail on close; D7 env bootstrap id; no staff mail; closed = read-only; staff sees all; guest display «Гость»
- Sibling AGENTS: client/server — «no admin API» до этого change
- Почта: `openspec/specs/auth/email-verification`, `auth/password-reset` (паттерн smtp.bz + RU)
