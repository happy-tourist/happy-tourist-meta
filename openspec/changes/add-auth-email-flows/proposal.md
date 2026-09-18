## Why

После регистрации по email сервер не умеет подтверждать адрес и не умеет «забыли пароль»: в проекте нет почтового транспорта и колбэков `@colyseus/auth`. Игрок не может подтвердить адрес из кабинета и восстановить доступ при утере пароля.

## What Changes

- Soft-подтверждение email **только по кнопке** в личном кабинете (письмо не уходит автоматически при регистрации); клик по ссылке подтверждает адрес на стороне API (smtp.bz); ссылка действует **30 минут**; после успешной отправки — диалог «письмо отправлено, проверьте почту».
- В кабинете: текущая почта, кнопка подтверждения рядом с почтой если ещё не подтверждена, возможность **сменить email** (после смены снова неподтверждён).
- Soft-verify: незавершённое подтверждение не блокирует лобби и игру; модалка раз за сессию напоминает подтвердить **из личного кабинета**.
- Google-аккаунты и уже существующие email-пользователи считаются подтверждёнными.
- Добавляется сценарий «забыли пароль» (письмо + сброс через штатные HTML-эндпоинты Colyseus); на Login — ссылка на запрос сброса.

## Capabilities

### New Capabilities

- `auth/email-verification`: soft-подтверждение по кнопке в кабинете, смена email, модалка → кабинет, verified для Google и существующих аккаунтов.
- `auth/password-reset`: запрос сброса пароля по email и установка нового пароля по ссылке из письма.

### Modified Capabilities

- (нет — поведение Google/email/anonymous login из `auth/login` не меняется на уровне требований входа)

## Scope

- **Capability ID:** `auth/email-verification`, `auth/password-reset`
- **Пакеты:** client (`happy-tourist.github.io`) и server (`happy-tourist-server`)
- **UX:** Login (ссылка forgot; **без** авто-письма и без «проверьте почту» как обязательного post-register mail hint); личный кабинет (email + кнопка подтверждения если не verified + смена email); модалка soft-verify раз за сессию («подтвердите из личного кабинета»); anonymous без модалки
- **Auth / HTTP:** `onEmailConfirmed` + `onForgotPassword`; штатные `/auth/confirm-email` и `/auth/reset-password` (серверные HTML); **не** включать auto-send на register (`onSendEmailConfirmation` не использовать для автоотправки); authenticated HTTP: отправка confirm-письма по кнопке (токен ссылки TTL 30 мин, cooldown 60 с) + смена email; client: success-диалог после отправки
- **Пользователи:** поле статуса подтверждения email; миграция/default для уже существующих рядов = confirmed; Google path = confirmed; смена email сбрасывает verified
- **Почта:** один транспорт smtp.bz; From на `happy-tourist.ru`
- После успешного confirm — редирект на клиентскую «главную» (лобби), не на Login

## Out of scope

- Автоотправка confirm при регистрации
- Resend и любой второй mail-провайдер / `MAIL_PROVIDER`
- Hard-gate лобби/комнат / рейтинга / разделов по `emailVerified` (волна позже)
- Смена имени и смена пароля из кабинета (Wave 2)
- Политика сложности пароля и strength meter (Wave 3)
- Брендированные SPA-страницы confirm/reset вместо серверных HTML Colyseus (D4-B позже)
- Другие OAuth-провайдеры, удаление anonymous/Google/email login
- Правила игры, lobby listing, board sync

## Impact

- Client: Login forgot; кабинет (email, confirm button, change email); session-модалка → кабинет; forgot/send-confirm/change-email через auth/HTTP.
- Server: mailer smtp.bz; confirm/forgot callbacks без auto-send на register; user field verified; HTTP send-confirm + change-email; env SMTP на VPS; `CLIENT_APP_URL` для редиректа после confirm.
- Ops: DNS/доставляемость для `happy-tourist.ru` через smtp.bz (вне кода).
- Deploy: новые ключи только в `.env` на сервере (не в GitHub Actions app secrets).

## References

- Explore + update: Wave 1; soft; smtp.bz only; confirm **только по кнопке**; смена email в кабинете; модалка → кабинет; D4-A серверные HTML; From `noreply@happy-tourist.ru`
- Sibling: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
- Skills: `client-work-with-auth`, `server-work-with-auth`, `work-with-env-deploy` (client/server), `work-with-routes`, `work-with-database`
- Docs Colyseus Auth: confirm/forgot callbacks (auto register send **не** используем)
- Main spec (смежный): `openspec/specs/auth/login/spec.md`
