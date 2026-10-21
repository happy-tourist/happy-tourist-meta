## 0. Server / BE

- Нет задач. Контракт и runtime `happy-tourist-server` не меняются (см. proposal Out of scope / design «Server / BE»).

## 1. Shell background CSS + App wire (client)

- [x] 1.1 Перед работой: `design.md` (D1–D6), delta `specs/ui/branding/spec.md` (SC-BRAND-21), skills `work-with-styles` / `theme.md`, `App.vue` layout
- [x] 1.2 В `src/css/app.scss` добавить `.app-page-bg`: cover + center + `background-attachment: fixed`; light/dark JPG via `body.body--dark`; mobile `@media` max-width 599px; прозрачный canvas (`.q-page`, `.q-page-container` если нужно, `.app-breadcrumbs` / `--q-page-bg`); проверить CSS
- [x] 1.3 Повесить `app-page-bg` на `q-layout` в `App.vue` (все маршруты) и проверить class в шаблоне

## 2. Card-grid underlay (client)

- [x] 2.1 Перед работой: delta `specs/content/packs/spec.md` (SC-PACK-285); `.pack-card-grid` в `app.scss` (layout block ≠ shared token selector with `.peek-chrome`); design D5
- [x] 2.2 Добавить underlay только на `.pack-card-grid`: `rgba(255,255,255,0.78)` / dark `rgba(18,18,18,0.78)`, `padding: 0.75rem`, `border-radius: 8px`; без underlay на `.peek-chrome`; проверить стили

## 3. Tests + quality (client)

- [x] 3.1 Перед работой: `client-work-with-test`; аналоги source-match на pack-card chrome (`PackListCardTile.test.ts`) и App shell (`AppHeaderChrome.test.ts`); design D7–D8
- [x] 3.2 Vitest SC-BRAND-21 + SC-PACK-285: (a) `App.vue` / layout class `app-page-bg` — расширить `src/__tests__/AppHeaderChrome.test.ts`; (b) `app.scss` source-match — `background-attachment: fixed`, четыре `url(...pack-bg-...)`, `@media` max-width 599px / `body.body--dark`, transparent canvas правила, underlay rgba + padding/radius только на `.pack-card-grid` (не на `.peek-chrome`); (c) четыре JPG существуют под `src/assets/packs/` — `npm test` в client проходит
- [x] 3.3 `npm run lint` и `npm run typecheck` в client; починить регрессии
- [x] 3.4 Spot-check (ручной или smoke): Login / Lobby light+dark — арт виден, карточки/формы читаемы; Game — отверстия доски MAY показывать page bg (SC-THEME-07), tile fills/HUD без регрессии
