## 0. Server — no implementation (reuse only)

- [x] 0.1 Перед работой: `design.md` «Server contract»; delta `specs/auth/login/spec.md` Related; убедиться, что change **не** добавляет room messages / HTTP routes / schema / DB / auth API
- [x] 0.2 Подтвердить reuse as-is (без правок `happy-tourist-server`): `@colyseus/auth` anonymous + upgrade; `tourist` `MyRoom.onAuth`; `GET /api/content/packs` + `GET /api/content/maps` с `auth.middleware()`; create gates `registered_user_required` для anonymous — без расширения прав гостя

## 1. Client — silent guest + Login upgrade

- [x] 1.1 Перед работой: `design.md` (D1–D3, D5); delta `specs/auth/login/spec.md` (SC-AUTH-11…15); skills `client-work-with-auth`, `work-with-pages`; файлы `src/router/index.ts`, `src/stores/auth.ts`, `src/pages/LoginPage.vue`
- [x] 1.2 В router `beforeEach`: при `requiresAuth` без сессии и без `requiresStaff`/`requiresAdmin` — `loginAnonymously` и остаться на целевом маршруте; при сбое anonymous → Login + существующий auth error (SC-AUTH-11b); staff/admin без сессии → Login с `redirect` на цель (без anonymous); на **Login** bounce в lobby только для registered (`!anonymous`); `forgot-password` и прочие `meta.guest` кроме Login — bounce любого authenticated в т.ч. anonymous (SC-AUTH-11…13; design D1–D3)
- [x] 1.3 LoginPage: скрыть «Войти как гость» если `auth.user?.anonymous`; сохранить `goAfterLogin` + `query.redirect` (SC-AUTH-14/15)
- [x] 1.4 Vitest: guard silent guest / silent-guest fail→Login / staff→Login / anonymous на Login / hide guest button / redirect после upgrade (SC-AUTH-11…15, 11b); `npm test` по затронутым тестам в client

## 2. Client — header login для гостя

- [x] 2.1 Перед работой: delta `specs/ui/branding/spec.md` (SC-BRAND-13/19/22/24); `design.md` D4; `App.vue` + `AppHeaderChrome.test.ts`; skill `work-with-pages` / branding chrome
- [x] 2.2 Anonymous off-Game и вне `AUTH_ROUTE_NAMES` (login/forgot/confirm/reset): иконка `login` вместо logout (wide + burger) + i18n aria (`auth.login` / «Войти»); account по-прежнему скрыт для guest; клик → Login с `redirect=fullPath` без logout; на auth-экранах ни logout ни login-icon; registered logout без регрессии (SC-BRAND-13/19/22/24; design D4)
- [x] 2.3 Vitest `AppHeaderChrome` (и смежные): guest login icon / no logout; registered logout; `npm test` по этим тестам

## 3. Client — скрыть identity-фильтры packs + maps

- [x] 3.1 Перед работой: delta `specs/content/packs/spec.md` (SC-PACK-153/286), `specs/content/maps/spec.md` (SC-MAP-69); `design.md` D6; `ContentCatalogPage.vue`, `MapsListPage.vue`; существующие filter-тесты
- [x] 3.2 Packs: identity-фильтры только при `!isGuest` (`v-if`, не disable); **all** остаётся видимым (SC-PACK-153/286)
- [x] 3.3 Maps: то же для moderation/drafts/mine; **all** остаётся видимым (SC-MAP-69)
- [x] 3.4 Vitest packs + maps filter chrome для guest; `npm test` по затронутым тестам

## 4. Client — phantom scroll с breadcrumbs

- [x] 4.1 Перед работой: delta `specs/ui/branding/spec.md` (SC-BRAND-23); `design.md` D7; `App.vue` breadcrumbs + `src/css/app.scss` / Quasar page min-height
- [x] 4.2 Исправить раскладку так, чтобы при коротком контенте packs/maps с крошками не было лишнего page-scroll; lobby без регрессии (SC-BRAND-23)
- [x] 4.3 Vitest или layout assertion на отсутствие принудительного overflow при коротком контенте с crumbs (SC-BRAND-23); при невозможности стабильного DOM-assert — source/layout unit check согласованный с design D7

## 5. Client — verify

- [x] 5.1 Перед работой: убедиться, что блоки 1–4 закрыты по SC из delta Traceability
- [x] 5.2 В sibling client: `npm run lint`, `npm run typecheck`, `npm test` — зелёные (починить падения в scope change)

## 6. Meta — skills sync

- [x] 6.1 Перед работой: delta specs + `design.md` D1–D7; текущие `.agents/skills/client/client-work-with-auth/SKILL.md`, `work-with-pages/SKILL.md`, `work-with-pages/shell.md`, `work-with-stores/content.md`; при необходимости краткая строка в `.agents/AGENTS.md`
- [x] 6.2 Обновить `client-work-with-auth`: silent `loginAnonymously` на ordinary `requiresAuth`; staff/admin без сессии → Login + `redirect`; Login bounce только для registered; `forgot-password` и прочие `meta.guest` — bounce любого authenticated; checklist/Do не оставлять «requiresAuth → /login» как единственный путь
- [x] 6.3 Обновить `work-with-pages` (+ `shell.md`): header login вместо logout для anonymous; account omitted for guest; Login без session/login-icon; guard snippet как в 6.2
- [x] 6.4 Обновить `work-with-stores/content.md`: identity-фильтры packs/maps для guest **скрыты** (`v-if`), не disabled
