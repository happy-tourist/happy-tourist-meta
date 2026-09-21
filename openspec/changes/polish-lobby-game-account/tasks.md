## 1. Server — password policy + register name

- [x] 1.1 Прочитать `design.md` (D6–D9), delta `specs/auth/login/spec.md`, `specs/auth/password-reset/spec.md`, skills `server-work-with-auth`, `work-with-database`, `work-with-routes`, `server-locate-change-points`
- [x] 1.2 Добавить shared server password-policy helper (≥8, `[a-z]`, `[A-Z]`, `[0-9]`, `[^A-Za-z0-9]`) и подключить reject на register path + JSON reset; покрыть mocha сценарии SC-AUTH-08 / SC-RESET-09 (или эквивалентные assert в существующих auth tests) — `npm test` в server проходит для затронутых тестов
- [x] 1.3 Persist register `name` → `users.displayName` (хук/post-register); userdata/JWT отдают имя так, что клиент видит displayName — тест SC-AUTH-10; `npm test`
- [x] 1.4 `POST` update displayName (`createEndpoint`, auth): trim, min 1, reject empty — тест SC-PROFILE-01/05; `npm test`
- [x] 1.5 `POST` change-password: current + new, `Hash.verify`, set hash, **bumpTokenVersion**, reject bad current / weak new — тесты SC-PROFILE-02/04; Google без password credential не принимает change — `npm test`
- [x] 1.6 На JSON reset при успехе bump token version (выровнять с change-password) — тест/assert; `npm test`

## 2. Client — auth deps + forms + AccountPage

- [x] 2.1 Прочитать delta `specs/auth/profile/spec.md`, `design.md` D7–D10, skills `client-work-with-auth`, `work-with-forms`, `work-with-pages`, `work-with-localization`, `client-locate-change-points`
- [x] 2.2 Установить в client `@zxcvbn-ts/core`, `@zxcvbn-ts/language-common`, `@zxcvbn-ts/language-ru`; shared client policy helper + strength meter UI (цвета + чеклист; Submit только по policy) — используется на register/reset/change
- [x] 2.3 Login register: policy rules + meter (SC-AUTH-08/09); login sign-in без complexity; i18n RU — `npm run lint` / `npm run typecheck` в client
- [x] 2.4 ResetPasswordPage: та же policy + meter (SC-RESET-09); lint/typecheck
- [x] 2.5 Auth store: actions update displayName + changePassword + logout после успешной смены; `displayName` предпочитает persisted name; register передаёт/отражает имя (SC-AUTH-10)
- [x] 2.6 AccountPage: форма имени; форма смены пароля (скрыть для Google/anonymous — SC-PROFILE-03); сохранить email-verify Wave 1; logout; soft-verify без hard-gate (SC-PROFILE-06) — lint/typecheck

## 3. Client — lobby join + stale rooms

- [x] 3.1 Прочитать delta `specs/lobby/rooms/spec.md`, `design.md` D2–D3, skill `work-with-lobby`, store `game.ts` subscribe/leave
- [x] 3.2 LobbyPage: busy-lock join (early-return, disable rows / loading) — SC-LOBBY-19; lint/typecheck
- [x] 3.3 game store: clear `rooms` + не показывать stale после leave/resubscribe until fresh `rooms` snapshot — SC-LOBBY-20; lint/typecheck

## 4. Client — board busy + presence reserve

- [x] 4.1 Прочитать delta `specs/game/board/spec.md`, `specs/game/presence/spec.md`, `design.md` D4–D5, skills `work-with-game-board`, `work-with-pages`
- [x] 4.2 GamePage: continuous `isBoardBusy` (intent lock до send; gap move→trap; grille hold); strip под lock; say доступен — SC-BOARD-27/32; lint/typecheck
- [x] 4.3 GamePage: seated всегда резервирует top presence row height без empty avatars — SC-PRESENCE-26; lint/typecheck

## 5. Verify packages

- [x] 5.1 Server: полный `npm test` в `happy-tourist-server` — зелёный
- [x] 5.2 Client: `npm run lint` и `npm run typecheck` в `happy-tourist.github.io` — зелёные
- [x] 5.3 При необходимости обновить meta skills/AGENTS краткими канонами (auth policy, lobby join busy, board-busy continuous, account profile) без дублирования specs
