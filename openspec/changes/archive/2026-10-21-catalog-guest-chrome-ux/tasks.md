## 0. Server — no implementation (reuse only)

- [x] 0.1 Перед работой: `design.md` «Server contract»; убедиться, что change **не** добавляет room messages / HTTP routes / schema / DB / auth API / filter query params
- [x] 0.2 Подтвердить reuse as-is (без правок `happy-tourist-server`): `listCatalog` / `listMaps` уже staff-only soft-unpublished; `getLivePack` доступен anonymous для in-catalog; create/add-task-set `requireVerifiedEditor` → `registered_user_required` для anonymous (SC-PACK-165 server mocha без изменений)

## 1. Packs list — guest filters + СНЯТО

- [x] 1.1 Перед работой: `design.md` (D1/D2, точки врезки ContentCatalog); delta `specs/content/packs/spec.md` (SC-PACK-286/289); skills `work-with-pages` / `client-work-with-test`; текущий `ContentCatalogPage.vue` + `ContentUnifiedList.test.ts`
- [x] 1.2 Скрыть весь filter chrome для anonymous (`v-if="!isGuest"` на ряд фильтров, не пустой row); исключить soft-unpublished (`hasLive && inCatalog === false`) из списка для non-staff (defense-in-depth к `listCatalog`); staff без изменений
- [x] 1.3 Vitest: обновить SC-PACK-286 (нет «Все»); добавить SC-PACK-289; `npm test` (затронутые) + при падении починить

## 2. Maps list — guest filters + СНЯТО

- [x] 2.1 Перед работой: delta `specs/content/maps/spec.md` (SC-MAP-69/70); `design.md` D1/D2; `MapsListPage.vue` + `ContentMaps.test.ts`
- [x] 2.2 Скрыть весь filter chrome для anonymous (`v-if="!isGuest"` на ряд фильтров, не пустой row); исключить soft-unpublished (`hasLive && inCatalog === false`) из списка для non-staff (defense-in-depth к `listMaps`); staff без изменений
- [x] 2.3 Vitest: обновить SC-MAP-69; добавить SC-MAP-70; `npm test` (затронутые)

## 3. Pack detail — add-task-set gate + view-only task-set

- [x] 3.1 Перед работой: delta packs SC-PACK-165/287/288/290/291; `design.md` D3/D6/D8; `ContentPackPage.vue` gate pattern; `ContentPackTasksPage.vue` `checkGate` / `liveViewMode`; `ContentPackAcl.test.ts` / related
- [x] 3.2 Показать «Добавить набор заданий» anonymous **и** unverified на in-catalog live (`showAddTaskSet`); клик → login / verify gate via `ensureEligible` (D3, как create); не открывать editor
- [x] 3.3 Live drill-in published task set (`view=live`): skip `checkGate` on boot for anonymous **и** unverified (D8); keep view-only (SC-PACK-288/291); mutate paths still call `checkGate`; vitest CTA+gate (287/290) и view-only без gate (288/291); revise SC-PACK-165 ACL test (guest **sees** CTA, gate on click / reject beyond gate)
- [x] 3.4 `npm test` (затронутые)

## 4. Catalog page chrome (packs + maps)

- [x] 4.1 Перед работой: `design.md` Visual Spec + D4/D5/D7; delta `specs/ui/branding/spec.md` (SC-BRAND-25/26); `work-with-styles` / `work-with-pages`
- [x] 4.2 Подтянуть title/subtitle/create band и токены (header/page/create) к Visual Spec на списках Наборы и Карты; без wordmark; без смены map capacity rows
- [x] 4.3 Vitest SC-BRAND-25/26 (или расширение существующих list chrome тестов); `npm test` (затронутые)

## 5. Client quality gate

- [x] 5.1 Перед работой: краткий просмотр Traceability в трёх delta specs — все pending SC закрыты тестами из §1–4
- [x] 5.2 Из `../happy-tourist.github.io`: `npm run lint`, `npm run typecheck`, `npm test` — починить регрессии этого change

## 6. Meta skills sync (после UI)

- [x] 6.1 Обновить client skills под новый SC-PACK-286 / SC-MAP-69 (весь filter row скрыт для guest, не «Все» остаётся): `.agents/skills/client/work-with-pages/content-pages.md`, `work-with-stores/content.md`, `work-with-stores/maps.md`, `client-work-with-test/stores.md`
- [x] 6.2 В тех же skills кратко: client omit soft-unpublished list rows для `!isStaff` (`hasLive && inCatalog === false`, D2); anonymous+unverified add-task-set CTA→login/verify gate (D3 / SC-PACK-287/290) + live view-only без `checkGate` на `view=live` (D8 / SC-PACK-288/291)

## 7. Follow-up — soft-unpublished task sets staff-only (D6 revise / SC-PACK-132/292)

- [x] 7.1 Перед работой: delta packs SC-PACK-132/292; `design.md` D6; `ContentPackPage.vue` task-set grid; тесты SoftUnpublish / ContentPackAcl / related asserting muted СНЯТО for non-staff
- [x] 7.2 На live pack detail: для `!isStaff` не рендерить soft-unpublished task sets (`inCatalog === false`); staff оставляют muted «СНЯТО» + Edit/Вернуть; ordinals среди видимых; deep-link non-enterable unchanged
- [x] 7.3 Vitest: revise SC-PACK-132 (omit for non-staff; staff keep card); add SC-PACK-292; `npm test` (затронутые)

## 8. Follow-up — create = lobby primary + translucent catalog underlay (D9/D10)

- [x] 8.1 Перед работой: delta branding SC-BRAND-27/28; design D9/D10; `ContentCatalogPage.vue` / `MapsListPage.vue` create; `app.scss` `.catalog-list-create` + solid `.catalog-list-page .pack-card-grid`
- [x] 8.2 Убрать `unelevated` и custom `.catalog-list-create` sizing — create как Lobby (`color="primary" icon="add"`); title band layout сохранить
- [x] 8.3 Удалить solid cream/navy override на catalog `.pack-card-grid` — снова SC-PACK-285 `rgba(…,0.78)`; обновить тесты SC-BRAND-27/28 / underlay assertions; `npm test` (затронутые)

## 9. Follow-up — translucent game HUD (D11 / SC-PRESENCE-34)

- [x] 9.1 Перед работой: delta `specs/game/presence/spec.md` SC-PRESENCE-34; `GamePage.vue` `.game-hud` CSS (сейчас solid `#fff` / `#121212`)
- [x] 9.2 Сделать sticky bottom `.game-hud` translucent: light `rgba(255,255,255,0.78)`, dark `rgba(18,18,18,0.78)`; top presence row не трогать
- [x] 9.3 Vitest / source-match assertion для SC-PRESENCE-34 (или расширение Game chrome теста); `npm test` (затронутые)

## 10. Follow-up quality + skills

- [x] 10.1 Traceability: SC-PACK-132/292, SC-BRAND-27/28, SC-PRESENCE-34 → covered; `npm run lint`, `typecheck`, `npm test` из client — починить регрессии follow-up
- [x] 10.2 Skills: content-pages / pack-cards / board (presence HUD) — staff-only soft-unpub task sets; create=lobby; translucent underlay + `.game-hud` alpha 0.78
