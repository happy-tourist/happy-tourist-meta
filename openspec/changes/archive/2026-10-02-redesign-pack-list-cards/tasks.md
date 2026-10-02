## 1. Server — list preview contract

- [x] 1.1 Прочитать `design.md` (Decisions 2–3), delta `specs/content/packs/spec.md` (SC-PACK-252/253) и skill `.agents/skills/server/work-with-routes` / `work-with-database`; найти `listCatalog` / `neverLiveAuthorTaskSets` в sibling `src/lib/content.ts` — verify: точка врезки ясна (enrichment в цикле `listCatalog` после выбора `revId`)
- [x] 1.2 Расширить `listCatalog`: на каждый pack — полный `taskSetsPreview` (id, 1-based ordinal, taskCount, inCatalog, neverLive?) из того же `revId`, что title/description; count-only / без slots-cards `loadRevisionPayload`; не резать до 4 на сервере; батч по revisionIds при возможности — verify: mocha SC-PACK-252 проходит (`npm test` в server, фильтр content packs)
- [x] 1.3 Soft-unpublished sets в preview; never-live ghost только автору (правила `retainedNeverLiveAuthorTaskSetCycle` / live append; cancelled тоже); slim ghost path без full slots — verify: mocha SC-PACK-253 проходит

## 2. Client — tile chrome + wire

- [x] 2.1 Прочитать Visual Spec в `design.md`, skill `.agents/skills/client/work-with-styles/pack-cards.md`, текущие `PackListCardTile` / `ContentCatalogPage` / `PackTaskSetCardTile` (icon mask pattern) — verify: готовность к правкам tile
- [x] 2.2 Обновить `PackListCardTile` под ~180×260: uppercase title, description, set-preview rows (max 4 + «ещё K»), divider над actions, outline+icon actions slot, revise red-outline badge **TR** (copy `ДОРАБОТАТЬ` via `taskSetCardBadge.needs_revision` / equal short key — не `statuses.needs_revision`), hover как task-set (без `--q-secondary`) — verify: vitest SC-PACK-228 / 249 / 251 (компонент или page)
- [x] 2.3 Типы store (`PackTaskSetPreview` + `ContentPackSummary.taskSetsPreview`) + `ContentCatalogPage`: передать preview, Material `edit`/`visibility_off`/`visibility` outline, whole-card open без навигации с set-row — verify: vitest SC-PACK-250 (muted set-row chrome superseded by §4 published-only; long card unpublish labels superseded by §5)
- [x] 2.4 i18n ключ `content.packCardSetsOverflow` = «ещё {k}»; обновить `pack-cards.md` под catalog Visual Spec (180×260 + preview) — verify: ключ есть в en-US; skill текст отражает новый chrome

## 3. Client / server verification

- [x] 3.1 Client: `npm run lint`, `npm run typecheck`, `npm test` (затронутые PackList / Catalog / list-cards) — verify: все зелёные
- [x] 3.2 Server: `npm test` (затронутые content list / packs) — verify: все зелёные

## 4. Follow-up — published-only + title.fg (client)

- [x] 4.1 В `PackListCardTile`: фильтр preview `inCatalog && !neverLive`; display ordinal 1…N среди опубликованных; overflow «ещё K» только по этому списку; убрать muted set-row chrome — verify: soft-unpub/neverLive не в DOM set-rows
- [x] 4.2 Цвета set label / count / icon → `title.fg` (Visual Spec Decision 12); обновить `pack-cards.md` — verify: нет отдельного muted grey для активных set-rows
- [x] 4.3 Vitest SC-PACK-249 (published-only overflow) + SC-PACK-254; `npm run lint` / `typecheck` / затронутые PackList tests — verify: все зелёные

## 5. Follow-up — shared chrome + short card actions (client)

- [x] 5.1 В `src/css/app.scss` на `.pack-card-grid` (light + `body.body--dark`): ввести `--pack-card-*` resting tokens (bg/fg/muted/border/border-hover/shadow/shadow-hover/splitter/action-h) со значениями = текущий task-set surface — verify: токены видны в DevTools / CSS на grid
- [x] 5.2 `PackTaskSetCardTile` + `PackListCardTile`: resting surface / hover / action outline / action-h / soft-unpub muted (`opacity: 0.72` + `border-style: dashed`) → `var(--pack-card-*)`; убрать расходящиеся list-only dark bg / `--pack-list-action-outline` — verify: visual no-op на task-set; list outline = border token; muted list имеет dashed
- [x] 5.3 `ContentCatalogPage` `#actions`: unpublish/republish labels → `content.taskSetCardUnpublish` / `content.taskSetCardRepublish` («Снять»/«Вернуть»); confirm/header длинные ключи не трогать — verify: vitest SC-PACK-250 (short copy)
- [x] 5.4 Vitest SC-PACK-255 + обновить `pack-cards.md` (shared tokens + short catalog actions); `npm run lint` / `typecheck` / затронутые PackList + PackTaskSet tests — verify: все зелёные
