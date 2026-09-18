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
