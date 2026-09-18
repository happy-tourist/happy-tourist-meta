## Context

См. `proposal.md` (Why / Scope) и delta specs `auth/email-verification`, `auth/password-reset`.

Сейчас:

- Client: Login (email/password + Google + anonymous) через Pinia auth; JWT `colyseus-auth-token`; guards `requiresAuth` / `guest`; личного кабинета нет; forgot/confirm UX нет.
- Server: `defineServer({ database: db })` → `@colyseus/auth` `/auth/*`; `src/config/auth.ts` только Google OAuth; mailer нет; в `users` нет флага verified.
- Colyseus 0.18: если задан `onSendEmailConfirmation`, письмо уходит **автоматически при register** — для этого change **не задаём** этот колбэк (отправка только с нашего HTTP по кнопке). Ссылки confirm/reset — на **API** HTML; клиентских `confirmEmail` нет.
- Prod: client `https://happy-tourist.github.io`, API `https://api.happy-tourist.ru`; From-домен `happy-tourist.ru`.

Пакеты: **server** + **client**. Чеклист — в `tasks.md`.

## Goals / Non-Goals

**Goals:**

- `sendEmail` через smtp.bz; forgot + `onEmailConfirmed`; **send-confirm только по кнопке** в кабинете.
- Смена email в кабинете (сброс verified).
- Soft UX: модалка → «подтвердите в личном кабинете»; Google/legacy = verified.
- Reset/confirm HTML на API; redirect после confirm на `#/lobby`.

**Non-Goals:**

- Auto-send при register / при смене email.
- Resend SaaS / второй провайдер.
- Hard-gate лобби/комнат/рейтинга.
- Смена имени/пароля; password complexity; SPA confirm/reset pages.

## Decisions

### D1: Только smtp.bz + nodemailer

- **Выбор:** `src/lib/mailer.ts` → `sendEmail(to, subject, html)`; deps `nodemailer` + `@types/nodemailer`.
- **Env:** `SMTP_BZ_HOST` / `PORT` / `USER` / `PASS`, `MAIL_FROM` (`Happy Tourist <noreply@happy-tourist.ru>`).
- **Prerequisite (ops):** DNS/SPF(+DKIM smtp.bz) для `happy-tourist.ru`; секреты только VPS / local env, не CI app secrets.

### D2: Не вешать `onSendEmailConfirmation` (нет авто-письма)

- **Выбор:** **не** назначать `auth.settings.onSendEmailConfirmation`, чтобы register не слал почту.
- **Назначить:** `onEmailConfirmed` (flag → true), `onForgotPassword` → `sendEmail`; `onResetPassword` из AuthService.
- **Отправка confirm:** только наш endpoint (D5), который сам собирает link (JWT + `auth.backend_url` + `/auth/confirm-email?token=`) и HTML (шаблон Colyseus или свой короткий HTML с `[LINK]`).
- **Почему:** продукт — только по кнопке.
- **Альтернатива:** задать колбэк и глушить send — хрупко; лучше не регистрировать auto-hook.

### D3: Флаг `emailVerified` в `users`

- Default `false` для новых email-регистраций; `onEmailConfirmed` → `true`.
- Google path → `true`; legacy backfill non-anonymous → `true`.
- Смена email → `emailVerified = false` (без авто-send).
- Userdata/JWT: клиент читает `emailVerified` (+ email) из Pinia auth user / userdata refresh.

### D4: Soft verify — без gate в `MyRoom.onAuth`

Без изменений относительно soft-решения.

### D5: Send-confirm + change-email — `createEndpoint`

- **`POST /api/auth/send-email-confirmation`:** `auth.middleware()`; registered non-anonymous; если уже verified — no-op/400; cooldown 60s in-memory по user id; confirm JWT с `expiresIn: '30m'` (payload минимум `{ email }`, как у reset); `sendEmail` с confirm link.
- **`POST /api/auth/email` (или `/api/profile/email`):** body `{ email }`; unique check; update email; `emailVerified = false`; опционально новый JWT/userdata refresh чтобы клиент сразу видел новый адрес; **не** слать письмо автоматически.
- Паттерн как `/api/theme`; не Express Router.

### D6: Confirm/reset UI = серверные HTML + redirect lobby

Как раньше: `CLIENT_APP_URL/#/lobby` после success; `AUTH_BACKEND_URL` → `auth.backend_url`.

### D7: Client UX

- **Login:** ссылка forgot; **нет** post-register «письмо уже ушло».
- **Forgot:** page + `sendPasswordResetEmail`.
- **Кабинет:** показать email; если `!emailVerified` — кнопка подтверждения **рядом с почтой**; после успешного send — **диалог** «письмо отправлено, проверьте почту»; форма/действие смены email; навигация из App/lobby.
- **Модалка (session reminder):** раз за sessionStorage; текст: подтвердить можно **из личного кабинета** (+ CTA на кабинет); не утверждать, что письмо уже отправлено.
- I/O в Pinia auth store.

### D8: Тесты

- Mocha + mock mailer: register **не** вызывает send; button endpoint вызывает; cooldown; confirm token `expiresIn: 30m`; change email сбрасывает verified и не шлёт mail; forgot шлёт; `onEmailConfirmed` ставит flag.
- Client: lint + typecheck.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Пользователи не найдут кнопку | Модалка явно указывает на кабинет + CTA |
| Смена email на чужой адрес | Unique constraint; confirm снова по кнопке; soft (играть можно) |
| Дублирование сборки confirm-link вне Colyseus register | Один helper рядом с mailer/auth config |
| smtp.bz / DNS | Ops; From = `@happy-tourist.ru` |

## Migration Plan

1. Колонка `email_verified` + backfill legacy = true.
2. Server (mailer, endpoints, callbacks без `onSendEmailConfirmation`) → client Pages.
3. Rollback: отключить SMTP/endpoints; login не ломается.

## Open Questions

(нет — TTL confirm 30m; cooldown 60s; success-диалог после send.)
