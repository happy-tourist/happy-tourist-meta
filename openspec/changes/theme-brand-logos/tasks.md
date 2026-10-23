## 0. Server

- Нет server-задач: room / schema / messages / HTTP / auth / DB не меняются; leave chrome только на client (см. `design.md` → Server scope).

## 1. Brand assets (client)

- [x] 1.1 Перед работой: прочитать `design.md` (D1), delta `specs/ui/branding/spec.md` (SC-BRAND-29); сверить файлы в `../happy-tourist.github.io/src/assets/brand/`
- [x] 1.2 Tight-crop `logo-dark-removebg-preview.png` / `logo-light-removebg-preview.png` → `logo-dark.png` / `logo-light.png`; удалить `logo.png` и оба `*-removebg-preview.png`; verify: в brand folder только theme wordmarks (+ любые несвязанные файлы), без маскота

## 2. Header chrome (client)

- [x] 2.1 Перед работой: skill `work-with-styles` / `work-with-pages` по необходимости; `design.md` D2–D4; текущий `src/App.vue` header logo + login/logout/leave
- [x] 2.2 Подключить theme-aware `brandLogoUrl` (`logo-dark.png` / `logo-light.png` imports + `$q.dark.isActive` per D2); CSS per D3: `.brand-logo` height `60px`, toolbar/content `min-height: 64px`, control vertical pad 2px; убрать `:label` у `header-logout`, `header-login`, `header-game-leave`; оставить icon + aria (`auth.logout` / `auth.login` / `game.leave`) + `flat round dense`; verify: `npm run typecheck` в client проходит для затронутых импортов

## 3. Account logout (client)

- [x] 3.1 Перед работой: `design.md` D4; `AccountPage.vue` logout; SC-BRAND-32
- [x] 3.2 Сделать session logout на Account icon-only (`icon="logout"` + `:aria-label="$t('auth.logout')"`, без `:label`; `flat` + `color="negative"`; `data-test-id="account-logout"`); verify: страница компилируется, контрол без текстового label в шаблоне

## 4. Tests + quality (client)

- [x] 4.1 Перед работой: skill `client-work-with-test`; Traceability SC-BRAND-29/30/31/32, SC-LEAVE-13; существующий `AppHeaderChrome.test.ts` (+ leave/account tests при наличии)
- [x] 4.2 Обновить `AppHeaderChrome.test.ts` под SC-BRAND-29/30/31 + SC-LEAVE-13 (D5); добавить vitest для Account logout SC-BRAND-32 (нового файла — ок); убрать ожидания visible label у toolbar login/logout/leave где мешают
- [x] 4.3 В `../happy-tourist.github.io`: `npm test`, `npm run lint`, `npm run typecheck` — все зелёные

## 5. Meta — skills / AGENTS (happy-tourist-meta)

- [x] 5.1 Перед работой: сверить факт диффа client с `design.md` D1–D5 и delta specs; список устаревших канонов: `logo.png` / высота **~44px** (40–48) / visible label у toolbar login|logout|`header-game-leave` / Account logout
- [x] 5.2 Обновить client skills под theme wordmarks (`logo-light.png` / `logo-dark.png`) + 60px + toolbar min-height 64 + icon-only session/leave/Account (`auth.login` / `auth.logout` / `game.leave` aria): как минимум `work-with-pages/shell.md`, `work-with-pages/SKILL.md`, `work-with-styles/theme.md`, `work-with-styles/SKILL.md`, `client-work-with-structure/SKILL.md`, `client-locate-change-points/SKILL.md`, `work-with-game-board/SKILL.md`, `work-with-localization/SKILL.md` (`game.leave` = aria-only для logo **и** right leave), `client-work-with-test` (+ `stores.md` при упоминании 44px / wordmark); verify: в skills нет `brand/logo.png` как header mark и нет «visible label on right-side Game leave»
- [x] 5.3 Синхронизировать discovery blurbs в `.agents/AGENTS.md` (и при необходимости root `AGENTS.md`), где ещё «logo ~44px» / старый маскот; обновить `docs/projects-map.md` ключ `client.assets` (`brand/logo.png` → theme wordmarks `logo-light.png` / `logo-dark.png`, высота 60px); sibling client `AGENTS.md` — только если там явно устаревший brand chrome
