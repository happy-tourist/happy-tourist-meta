## 1. Server — Google OAuth provider

- [x] 1.1 Прочитать `design.md` (D1–D5), `specs/auth/login/spec.md` (SC-AUTH-01…03, SC-AUTH-07), skill `.agents/skills/server/server-work-with-auth/SKILL.md` и текущий `../happy-tourist-server/src/app.config.ts` / `.env.example` — зафиксировать точку `addProvider` и env-имена
- [x] 1.2 В Google Cloud Console создать OAuth Web client(s); добавить Authorized redirect URI `http://localhost:2567/auth/provider/google/callback` (dev) и prod callback на API-хост; сохранить Client ID/Secret для env (не в git)
- [x] 1.3 Добавить `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` в `../happy-tourist-server/.env.example` (и локальный `.env.development`) с комментарием про redirect URI; убедиться, что значения читаются при старте
- [x] 1.4 Создать `../happy-tourist-server/src/config/auth.ts` с `auth.oauth.addProvider('google', { key, secret, scope: ['email','profile'] })`; импортировать из `app.config.ts` (side-effect); **не** переопределять `onOAuthProviderCallback` (D2)
- [x] 1.5 В `../happy-tourist-server`: `npm test` (регрессия существующих room/auth тестов); при падении — починить до перехода к client

## 2. Client — one-click Google button

- [x] 2.1 Прочитать skill `.agents/skills/client/client-work-with-auth/SKILL.md`, `../happy-tourist.github.io/src/stores/auth.ts`, `../happy-tourist.github.io/src/pages/LoginPage.vue`, design D3/D6 и SC-AUTH-01…06
- [x] 2.2 В auth store добавить действие Google sign-in через `client.auth.signInWithProvider('google')` (loading/error как у остальных actions); Colyseus I/O остаётся в store
- [x] 2.3 На LoginPage: одна кнопка Google; success → существующий post-login redirect; failure/cancel → `auth.error` + существующий banner (SC-AUTH-05); email/password и guest не убирать (SC-AUTH-06)
- [x] 2.4 Добавить i18n-ключ лейбла кнопки в `../happy-tourist.github.io/src/i18n/en-US/` и использовать на кнопке
- [x] 2.5 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 3. Meta + validate

- [x] 3.1 Обновить `.agents/skills/client/client-work-with-auth/SKILL.md` и `.agents/skills/server/server-work-with-auth/SKILL.md` (Google provider, one-click, env, callback URL, reuse built-in OAuth callback); при необходимости одна строка в sibling AGENTS.md
- [x] 3.2 Из корня meta: `openspec validate add-google-auth` (и при готовности к merge — `/opsx-sync` / archive по отдельной просьбе)
