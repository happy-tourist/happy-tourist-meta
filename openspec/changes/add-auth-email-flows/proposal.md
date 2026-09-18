## Why

Wave 1–2 закрыли smtp.bz, button-only confirm, forgot/cabinet и русский copy на серверных HTML Colyseus. Остаются UX-дыры: forgot говорит «если существует», хотя API уже отвечает «не найден»; ссылки confirm/reset ведут на `api.*` HTML; после сброса пароля пользователь остаётся на серверной форме. Нужны SPA-страницы на клиентском домене (`CLIENT_APP_URL`, prod: `https://happy-tourist.ru`) и JSON-эндпоинты без прокси `/auth/` на apex.

## What Changes

- Soft-подтверждение email **только по кнопке** в личном кабинете (письмо не уходит автоматически при регистрации); smtp.bz; ссылка **30 минут**; после отправки — диалог про почту **и Спам**.
- Кабинет: почта, кнопка подтверждения рядом, смена email; soft-verify (не блочит игру); модалка раз за сессию → кабинет; Google/legacy verified.
- **Confirm / reset UX на клиенте (SPA):** ссылки в письмах — `{CLIENT_APP_URL}/#/confirm-email?token=…` и `{CLIENT_APP_URL}/#/reset-password?token=…` (hash router), **не** `api.*` HTML Colyseus.
- **JSON API** на сервере для confirm и reset (тонкие `createEndpoint`); SPA не парсит HTML-redirect Colyseus.
- Confirm: открытие ссылки → SPA сразу вызывает JSON (без второй кнопки «Подтвердить») → при успехе → `#/lobby`.
- Reset: SPA-форма нового пароля → JSON → при успехе → `#/login` (войти с новым паролем).
- Forgot: при неизвестном email — явная RU-ошибка «аккаунт не найден» (не soft «если существует»); при успехе — «письмо отправлено… и Спам».
- Старые ссылки на `api.*/auth/confirm|reset` **не поддерживаем** (как будто их не было).
- `AUTH_BACKEND_URL` остаётся для Google OAuth callback; в письмах base = `CLIENT_APP_URL`.
- Русский human-facing copy (письма + SPA outcomes); skills/AGENTS; `.env.example` `connect.smtp.bz`.

## Capabilities

### New Capabilities

- `auth/email-verification`: soft-confirm из кабинета; SPA confirm + JSON; verified Google/legacy; RU copy.
- `auth/password-reset`: forgot + SPA reset + JSON; явный not-found; redirect login; RU copy.

### Modified Capabilities

- (нет — `auth/login` требования входа не меняются)

## Scope

- **Capability ID:** `auth/email-verification`, `auth/password-reset`
- **Пакеты:** client + server + meta skills/AGENTS
- **UX:** SPA confirm/reset guest routes; forgot not-found; post-send spam hints
- **Auth / HTTP:** Wave 1 endpoints + новые JSON confirm/reset; письма с client hash links
- **Почта:** smtp.bz; `connect.smtp.bz`
- **Язык:** RU human-facing в этом контуре

## Out of scope

- Автоотправка confirm при регистрации
- Resend / второй mail-провайдер / `MAIL_PROVIDER`
- Hard-gate лобби/комнат по `emailVerified`
- Смена имени / смена пароля из кабинета (отдельная волна)
- Политика сложности пароля / strength meter
- Nginx-прокси `/auth/` на apex (отклонён — только SPA)
- Поддержка старых писем со ссылками на `api.*` HTML
- Другие OAuth; удаление anonymous/Google/email login
- Правила игры, lobby listing, board sync
- Массовый перевод legacy UI вне auth-email

## Impact

- Client: guest pages confirm/reset; auth store JSON actions; forgot i18n not-found; skills.
- Server: JSON confirm/reset endpoints; link builder → `CLIENT_APP_URL` hash; можно убрать/не опираться на `writeConfirmSuccessHtml` / reset HTML как UX; тесты.
- Meta: skills auth/pages/localization — SPA + JSON канон.
- Ops: `CLIENT_APP_URL` = публичный client origin; `AUTH_BACKEND_URL` только OAuth/API origin.

## References

- Explore 2026-09-18: D1 not-found; D2 SPA без прокси; D3 reset→login; D4 JSON; D5 confirm→lobby; D6 нет fallback api.*
- Sibling AGENTS; skills `client-work-with-auth`, `work-with-pages`, `server-work-with-auth`, `work-with-routes`
- Main spec смежный: `openspec/specs/auth/login/spec.md`
