## Why

После регистрации по email сервер не умел подтверждать адрес и не умел «забыли пароль»: не было почтового транспорта и колбэков `@colyseus/auth`. Wave 1 закрыла flows; сейчас human-facing тексты писем и серверных HTML всё ещё на английском, а игроку нужен русский продукт (включая подсказку про папку «Спам»).

## What Changes

- Soft-подтверждение email **только по кнопке** в личном кабинете (письмо не уходит автоматически при регистрации); клик по ссылке подтверждает адрес на стороне API (smtp.bz); ссылка действует **30 минут**; после успешной отправки — диалог «письмо отправлено, проверьте почту **и папку Спам**».
- В кабинете: текущая почта, кнопка подтверждения рядом с почтой если ещё не подтверждена, возможность **сменить email** (после смены снова неподтверждён).
- Soft-verify: незавершённое подтверждение не блокирует лобби и игру; модалка раз за сессию напоминает подтвердить **из личного кабинета**.
- Google-аккаунты и уже существующие email-пользователи считаются подтверждёнными.
- Сценарий «забыли пароль» (письмо + сброс через штатные HTML-эндпоинты Colyseus); на Login — ссылка на запрос сброса; feedback forgot тоже упоминает проверку спама.
- **Русский human-facing copy** для всего нового в этом контуре: confirm/reset письма (subject + body), HTML confirm (успех / истекший или невалидный токен), HTML reset-форма и reset-письмо; клиентские строки отправки писем — на русском (каталог i18n может оставаться `en-US` технически).
- После успешного confirm — редирект на клиентское лобби (`CLIENT_APP_URL/#/lobby`).
- Канон в skills/AGENTS: новое, что читает человек в auth-email контуре — по-русски.
- Ops docs: `.env.example` — хост smtp.bz `connect.smtp.bz` (не несуществующий `smtp.smtp.bz`).

## Capabilities

### New Capabilities

- `auth/email-verification`: soft-подтверждение по кнопке в кабинете, смена email, модалка → кабинет, verified для Google и существующих аккаунтов; RU copy писем/HTML confirm.
- `auth/password-reset`: запрос сброса пароля по email и установка нового пароля по ссылке; RU copy письма и HTML формы.

### Modified Capabilities

- (нет — поведение Google/email/anonymous login из `auth/login` не меняется на уровне требований входа)

## Scope

- **Capability ID:** `auth/email-verification`, `auth/password-reset`
- **Пакеты:** client (`happy-tourist.github.io`) и server (`happy-tourist-server`) + meta skills/AGENTS / `.env.example`
- **UX:** Login forgot; кабинет; session-модалка; диалоги/баннеры после **любой** отправки письма — проверить почту и **Спам**; anonymous без verify-модалки
- **Auth / HTTP:** как Wave 1 (button-only confirm, forgot, change-email, TTL 30m, cooldown 60s) + русский copy на серверных шаблонах/хелперах
- **Почта:** smtp.bz; From на `happy-tourist.ru`; host в примере env — `connect.smtp.bz`
- **Язык:** только русский для human-facing в этом контуре; полный аудит старого UI вне auth-email — out of scope

## Out of scope

- Автоотправка confirm при регистрации
- Resend и любой второй mail-провайдер / `MAIL_PROVIDER`
- Hard-gate лобби/комнат / рейтинга / разделов по `emailVerified` (волна позже)
- Смена имени и смена пароля из кабинета (Wave 2)
- Политика сложности пароля и strength meter (Wave 3)
- Брендированные SPA-страницы confirm/reset вместо серверных HTML Colyseus (D4-B позже)
- Другие OAuth-провайдеры, удаление anonymous/Google/email login
- Правила игры, lobby listing, board sync
- Массовый перевод всего legacy UI вне поверхностей auth-email этого change

## Impact

- Client: i18n строки confirm/forgot success (+ спам); skills localization/auth.
- Server: RU HTML/письма confirm+reset; маппинг EN query от Colyseus → RU на странице; `.env.example` host.
- Meta: skills + AGENTS — канон human-facing RU для нового в auth-email.
- Ops: VPS `SMTP_BZ_HOST=connect.smtp.bz`; `CLIENT_APP_URL` для редиректа после confirm.

## References

- Explore: RU copy; spam hint; update current change; D1 reset HTML RU; D2 только новое auth-email + канон skills
- Sibling: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
- Skills: `client-work-with-auth`, `work-with-localization`, `server-work-with-auth`, `work-with-env-deploy`
- Main spec (смежный): `openspec/specs/auth/login/spec.md`
