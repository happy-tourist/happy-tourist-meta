## Context

См. `proposal.md` (Why / Scope). Сейчас:

- Client: Login — email/password + anonymous через Pinia `auth` (`register` / `login` / `loginAnonymously`); JWT в `colyseus-auth-token`; guards `requiresAuth` / `guest` + `whenReady()`.
- Server: `defineServer({ database: db })` монтирует `@colyseus/auth` `/auth/*`; user store SQLite; `MyRoom.onAuth` → `JWT.verify`. OAuth-провайдеры не зарегистрированы.
- Встроенный `AuthService.oauthCallback` (`@colyseus/database`) уже умеет: upgrade anonymous по `upgradingToken`, find-by-email, create user — отдельный custom `onCallback` для MVP не нужен.
- Deploy: client GitHub Pages, server отдельный origin; OAuth callback всегда на **server** URL.

Пакеты: **server** + **client** (cross-package). Meta: skills после кода.

## Goals / Non-Goals

**Goals:**

- Зарегистрировать OAuth provider `google` на сервере (env secrets).
- Одна кнопка на Login → `client.auth.signInWithProvider('google')` через Pinia auth store.
- Опереться на встроенный OAuth callback + существующий JWT/room gate / token persistence.
- Email/password и guest без регрессии.

**Non-Goals:**

- Другие провайдеры, UI account linking, колонка `google_id`.
- Кастомный `onOAuthProviderCallback` (кроме follow-up для `displayName`, если понадобится).
- Менять `MyRoom.onAuth` или схему обязательных user defaults ради Google.

Чеклист реализации — в `tasks.md`, не дублировать здесь.

## Decisions

### D1: Штатный Colyseus OAuth, не свой Passport/Google ID token flow

- **Выбор:** `auth.oauth.addProvider('google', { key, secret, scope })` + client `signInWithProvider('google')` (popup → `/auth/provider/google` → callback).
- **Почему:** уже в `@colyseus/auth` / SDK; тот же JWT, что email/anonymous.
- **Альтернатива:** Google Identity Services + свой `/auth/google` — отклонена (дублирует auth layer).

### D2: Не писать свой `onCallback` в MVP

- **Выбор:** оставить `database: db` → `onOAuthProviderCallback` из AuthService (email match / create / anon upgrade).
- **Почему:** покрывает SC-AUTH-01..03 без миграции схемы.
- **Альтернатива:** custom upsert по `google_id` — out of scope proposal.
- **Следствие:** `displayName` может остаться пустым; UI уже fallback на email / «Игрок».

### D3: Одна кнопка = register + login

- **Выбор:** один control на Login; нет отдельного «Зарегистрироваться через Google».
- **Почему:** OAuth не разделяет эти шаги; продукт «один клик».

### D4: Секреты и redirect на server origin

- **Выбор:** env `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`; Google Console redirect  
  `https://<api-host>/auth/provider/google/callback` (dev: `http://localhost:2567/...`).
- **Почему:** callback обрабатывает Colyseus на API, не GitHub Pages.
- **Prerequisite / blocker:** без Console clients и URI кнопка не заработает в runtime; local smoke требует `.env.development` с валидными ключами.
- **Альтернатива:** захардкодить key в репо — запрещено.

### D5: Точка врезки server — отдельный auth config module

- **Выбор:** `src/config/auth.ts` (или аналог) с `addProvider`, импорт из `app.config.ts` при загрузке модуля (до listen).
- **Почему:** не раздувать express-hook; явная регистрация провайдера рядом с auth concerns.
- **Альтернатива:** вызов внутри `express(app)` — хуже по читаемости, не нужен request context.

### D6: Client I/O только в Pinia auth store

- **Выбор:** action вроде `loginWithGoogle` → `client.auth.signInWithProvider('google')`; LoginPage только вызывает store + `goAfterLogin` / показывает `auth.error`.
- **Почему:** канон client skills (Colyseus I/O не в page).
- **i18n:** для новой строки кнопки — ключ в `src/i18n/en-US` (даже если соседний copy пока hardcoded RU).

## Точки врезки

### Server (`../happy-tourist-server`)

| Место | Что сделать |
|-------|-------------|
| `.env.example` (+ local/prod env) | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`; документировать redirect URI |
| `src/config/auth.ts` (новый) | `auth.oauth.addProvider('google', { key, secret, scope: ['email','profile'] })` |
| `src/app.config.ts` | side-effect import config auth; `database: db` без смены |
| `src/db/schema.ts` | без изменений в MVP |
| `src/rooms/MyRoom.ts` | без изменений (`JWT.verify` уже достаточен) |
| Tests | полный Google OAuth в mocha нереалистичен без мока Grant; опционально assert что provider зарегистрирован / health не ломается; SC без автотеста — `pending` в Traceability specs, не отдельный manual-пункт в tasks |

### Client (`../happy-tourist.github.io`)

| Место | Что сделать |
|-------|-------------|
| `src/stores/auth.ts` | `loginWithGoogle`: loading/error + `signInWithProvider('google')`; export в return |
| `src/pages/LoginPage.vue` | одна кнопка Google; success → существующий redirect после login; error → существующий banner |
| `src/i18n/en-US/` | ключ лейбла кнопки |
| Router / guards / boot | без изменений |

### Meta (после кода)

| Место | Что сделать |
|-------|-------------|
| `.agents/skills/client/client-work-with-auth/SKILL.md` | Google / `signInWithProvider` / one-click |
| `.agents/skills/server/server-work-with-auth/SKILL.md` | `addProvider('google')`, env, callback URL, reuse AuthService callback |
| Sibling AGENTS.md (по необходимости) | строка: email + anonymous + Google |

Слои client: page → store → `client` из boot.

## Risks / Trade-offs

- **[Popup blocked / cancelled]** → Mitigation: error в store + banner; email/guest остаются (SC-AUTH-05).
- **[Wrong redirect URI]** → Colyseus help / OAuth fail → Mitigation: документировать URI в `.env.example` и tasks; отдельные clients для dev/prod.
- **[No email in Google profile]** → AuthService может создать user с `email: null` → Mitigation: scope `email`+`profile`; edge case принять для MVP.
- **[displayName empty]** → Mitigation: существующий fallback displayName; follow-up wrap callback вне MVP.
- **[Cross-origin popup]** Pages ↔ API → Mitigation: CORS уже credentials + Authorization; smoke на prod Pages обязателен.
- **[Secrets missing in CI/VPS]** → Mitigation: env на VPS; без ключей Google path недоступен, email/guest работают.

## Migration Plan

1. Создать Google OAuth clients (dev/prod) и redirect URI на server.
2. Задеплоить server с `addProvider` + env secrets.
3. Задеплоить client с кнопкой Google.
4. Smoke: first login, reload, join `checkers`, email/guest regression.
5. Rollback: убрать кнопку client и/или provider registration server; email/anonymous не зависят от Google.

## Open Questions

- Нужны ли отдельные Google OAuth clients для staging vs production, или один client с несколькими redirect URI — на усмотрение ops при apply (на поведение spec не влияет).
