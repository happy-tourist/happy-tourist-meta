## 1. Server — mailer + schema + button-only confirm + change email

- [x] 1.1 Прочитать `design.md` (D1–D6), specs `auth/email-verification` + `auth/password-reset`, skills `.agents/skills/server/server-work-with-auth/SKILL.md`, `work-with-config`, `work-with-database`, `work-with-env-deploy`, `work-with-routes`; сверить `../happy-tourist-server/src/config/auth.ts`, `src/db/schema.ts`, `.env.example`

- [x] 1.2 В `../happy-tourist-server`: установить `nodemailer` и `@types/nodemailer`; добавить `src/lib/mailer.ts` с `sendEmail` только smtp.bz (`SMTP_BZ_*`, `MAIL_FROM`); без Resend / `MAIL_PROVIDER`

- [x] 1.3 Документировать в `.env.example`: `SMTP_BZ_*`, `MAIL_FROM`, `AUTH_BACKEND_URL`, `CLIENT_APP_URL`; секреты не коммитить

- [x] 1.4 Добавить `emailVerified` (default false для новых); backfill существующих non-anonymous → `true`; register/login не падают на NOT NULL

- [x] 1.5 В `src/config/auth.ts`: **не** задавать `onSendEmailConfirmation`; задать `onEmailConfirmed` → verified true, `onForgotPassword` → `sendEmail`, `auth.backend_url` из env; Google path → verified true; helper сборки confirm link+html для HTTP send

- [x] 1.6 После успешного confirm — redirect на `CLIENT_APP_URL/#/lobby` (design D6)

- [x] 1.7 `POST /api/auth/send-email-confirmation` (`createEndpoint` + `auth.middleware()`): registered non-anonymous, cooldown 60s, confirm JWT `expiresIn: '30m'`, отправка confirm-письма; уже verified — предсказуемый отказ/no-op

- [x] 1.8 `POST /api/auth/email` (или `/api/profile/email`): смена email, unique check, `emailVerified = false`, **без** авто-send; обновить userdata/token так, чтобы клиент видел новый email

- [x] 1.9 Userdata отдаёт `email` + `emailVerified` клиенту

- [x] 1.10 Mocha (mock mailer): register **не** шлёт confirm; send-endpoint шлёт; cooldown; expired 30m link rejected; change-email сбрасывает verified и не шлёт; forgot шлёт; confirm callback ставит flag; `npm test` в server — починить при падении

## 2. Client — forgot, cabinet (confirm button + change email), modal

- [x] 2.1 Прочитать skills `.agents/skills/client/client-work-with-auth/SKILL.md`, `work-with-pages`, `work-with-forms`, `work-with-localization`, `colyseus-client`; `design.md` D7; `LoginPage`, `stores/auth.ts`, routes, `App.vue`

- [x] 2.2 Auth store: forgot; `sendEmailConfirmation` → HTTP send; `changeEmail` → HTTP; refresh userdata; loading/error как у существующих actions

- [x] 2.3 Login: ссылка forgot; **без** post-register hint «письмо уже ушло»; i18n

- [x] 2.4 Forgot-password page + guest route; banner feedback; возврат на Login (SC-RESET-04/05)

- [x] 2.5 Личный кабинет (requiresAuth): текущий email; если `!emailVerified` — кнопка подтверждения **рядом с почтой**; после успешного send — диалог «письмо отправлено, проверьте почту» (SC-EMAIL-06); UI смены email; навигация из App/lobby

- [x] 2.6 Модалка раз за session: текст что подтвердить можно **из личного кабинета** (+ переход в кабинет); anonymous без модалки (SC-EMAIL-08/09)

- [x] 2.7 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении починить

## 3. Meta skills sync

- [x] 3.1 Обновить server skills auth/env-deploy: smtp.bz, no auto `onSendEmailConfirmation`, send-confirm + change-email endpoints, `emailVerified`

- [x] 3.2 Обновить `client-work-with-auth`: cabinet confirm button + change email, modal → cabinet, forgot; без SPA confirm/reset pages и без auto mail на register

## 4. Russian human-facing copy + spam hint + smtp host docs

- [x] 4.1 Server: confirm email subject+HTML на русском (тот же смысл, что EN); subject в send-endpoint — RU (SC-EMAIL-13); проверить mock-mailer / ручной просмотр тела

- [x] 4.2 Server: `address-confirmation.html` (+ генерация в `auth.ts`) — заголовок/успех/ошибка на русском; маппинг EN query Colyseus → RU; expired/invalid → RU; redirect `#/lobby` сохранить (SC-EMAIL-02/11/12); `npm test` auth email — починить при падении

- [x] 4.3 Server: `reset-password-email.html` + subject forgot на русском; `reset-password-form.html` (+ success/error) на русском (SC-RESET-02/06)

- [x] 4.4 Client: i18n `confirmSentDialog` и `forgotSuccess` — проверить почту **и папку Спам** (SC-EMAIL-14, SC-RESET-05); `npm run lint` + `npm run typecheck`

- [x] 4.5 Meta: skills `server-work-with-auth`, `work-with-env-deploy`, `client-work-with-auth`, `work-with-localization` (+ AGENTS при необходимости) — канон human-facing RU для auth-email / нового в контуре

- [x] 4.6 Server `.env.example`: `SMTP_BZ_HOST=connect.smtp.bz` (+ комментарий портов 2525/587/465); убрать неверный `smtp.smtp.bz`

## 5. Wave 3 — SPA confirm/reset + JSON + forgot not-found

- [x] 5.1 Server: JSON `POST /api/auth/confirm-email` `{ token }` → JWT verify → `emailVerified`; ошибки expired/invalid предсказуемы; `npm test` (SC-EMAIL-02/12)

- [x] 5.2 Server: JSON `POST /api/auth/reset-password` `{ token, password }` → reset; one-time token semantics; `npm test` (SC-RESET-02)

- [x] 5.3 Server: ссылки в confirm/forgot письмах = `{CLIENT_APP_URL}/#/confirm-email?token=` и `#/reset-password?token=`; `AUTH_BACKEND_URL` не использовать как base mail links; mock-mailer assert host/path (SC-EMAIL-15, SC-RESET-08)

- [x] 5.4 Server: убрать опору на product UX серверных HTML confirm/reset (`writeConfirmSuccessHtml` / HTML forms) — не требуются; тесты не зависят от API HTML success pages

- [x] 5.5 Client: guest routes + pages ConfirmEmail / ResetPassword; confirm on mount auto JSON (без кнопки Confirm); успех → lobby; ошибка RU (SC-EMAIL-02/11/12)

- [x] 5.6 Client: ResetPassword форма → JSON; успех → login; i18n RU (SC-RESET-02/08)

- [x] 5.7 Client: forgot — success без «если существует»; map `email_not_found` → явный RU not-found (SC-RESET-05/07); lint + typecheck

- [x] 5.8 Meta: skills `client-work-with-auth`, `work-with-pages`, `server-work-with-auth`, `work-with-routes` (+ AGENTS) — SPA + JSON канон; нет «только Colyseus HTML»
