## 1. Client — CSV helper

- [x] 1.1 Прочитать `design.md` (D1–D10), delta `specs/content/packs/spec.md` (SC-PACK-210…218); skills `client-locate-change-points`, `work-with-test` — зафиксировать API helper
- [x] 1.2 Добавить `src/lib/packContentCsv.ts`: serialize/parse answers (`content;paragraph…` → description через `\n`), serialize/parse tasks (`question;difficulty;slot…`, difficulty default `1`), resolve slots по точному `content` + collect missing texts; `;` split без escape; trim ячеек
- [x] 1.3 Vitest на helper: round-trip answers/tasks, append-ready parse, missing answers list, invalid/empty difficulty → 1 (SC-PACK-210/211/213/214/215/218 ядро); `npm test` в client

## 2. Client — answers import/export on cards editor

- [x] 2.1 Skills `work-with-pages`, `work-with-stores` (+ content), `work-with-forms`, `work-with-localization`, `client-work-with-errors`; страница cards editor
- [x] 2.2 Export: Blob download текущего `local.answerCards`, имя от pack title (SC-PACK-210)
- [x] 2.3 Import: file dialog → parse → append cards с `newLocalId`; disable при `cardsReadOnly`; успех закрывает диалог; ошибка через banner/dialog без partial apply (SC-PACK-211/212/218)
- [x] 2.4 i18n подписи кнопок/ошибок; vitest page/composable smoke где практично; `npm test`

## 3. Client — tasks import/export on both task surfaces

- [x] 3.1 Skills `work-with-pages`, `work-with-stores`, `work-with-localization`, `client-work-with-errors`; `ContentPackAddTaskSetPage` + `ContentPackTasksPage` (shared control per design D3)
- [x] 3.2 Export текущего набора: слоты как content текста карточек; имя `{title}-tasks-{n}.csv` (SC-PACK-213/217)
- [x] 3.3 Import: гейт disable при пустом answer context (`liveCards` / draft cards); missing slot texts → reject + list; иначе append tasks со слотами; difficulty default 1; read-only соблюсти (SC-PACK-214/215/216/217/218)
- [x] 3.4 i18n + vitest на гейт/missing/append (SC-PACK-214…217); `npm test`

## 4. Client — verify package

- [x] 4.1 `npm run lint`, `npm run typecheck`, `npm test`, `npm run build` в `../happy-tourist.github.io`

## 5. Client — CSV import modal (D6′)

- [x] 5.1 Skills `work-with-pages`, `work-with-localization`, `client-work-with-errors`, `work-with-test`; design D6′; SC-PACK-218/219/220/221
- [x] 5.2 Answers: Import → `q-dialog` с примером формата `ответ;абзац1;абзац2;…` + выбор файла; ошибка внутри модалки; успех закрывает; Export disabled при 0 cards (SC-PACK-218/219/221)
- [x] 5.3 Tasks (`PackTasksCsvControls`): та же модалка с примером `вопрос;сложность;слот…`; Export disabled при 0 tasks; убрать page-banner как primary path ошибок импорта (SC-PACK-218/220/221)
- [x] 5.4 i18n format hints; vitest modal smoke; `npm test`

## 6. Client — playing-card chrome (D11)

- [x] 6.1 Skills `work-with-pages`, `work-with-styles`, `client-work-with-structure`, `work-with-localization`, `work-with-test`; design D11; SC-PACK-222…224
- [x] 6.2 Shared answer-card tile: rounded, split content|description; empty description → shorter rectangle; description overflow → scroll; pencil+delete top-right when editable
- [x] 6.3 Shared task tile: rounded, split question|slots; difficulty top-left; pencil+delete top-right when editable
- [x] 6.4 Wire tiles on editor lists (Add form stays above grid), slot picker, live pack/tasks views, staff moderation views (SC-PACK-222…224)
- [x] 6.5 Vitest component/page smoke где практично; `npm test`

## 7. Client — verify package (follow-up)

- [x] 7.1 `npm run lint`, `npm run typecheck`, `npm test`, `npm run build` в `../happy-tourist.github.io`

## 8. Client — tile sizes + dark contrast (D12)

- [x] 8.1 Skills `work-with-styles`, `work-with-pages`, `work-with-test`; design D12; SC-PACK-225/226/227
- [x] 8.2 `PackAnswerCardTile`: 100×200 without description; 200×200 with description + splitter; explicit light/dark bg + text + edit icon colors
- [x] 8.3 `PackTaskTile`: 200×200; same contrast rules
- [x] 8.4 Vitest size/contrast smoke; `npm test`

## 9. Client — catalog + task-set cards 100×200 (D13)

- [x] 9.1 Skills `work-with-pages`, `work-with-structure`, `work-with-localization`, `work-with-test`; design D13; SC-PACK-228/229
- [x] 9.2 Catalog packs: card grid 100×200 — status top, star top-left, Edit top-right only, truncated title
- [x] 9.3 Live + editor task-set lists: same 100×200 card chrome (status top, Edit right, truncated label); preserve soft-unpublish / open rules
- [x] 9.4 Vitest page smoke; `npm test`

## 10. Server/Client — staff false draft (D14)

- [x] 10.1 Skills `server-work-with-structure` / `work-with-routes` / `work-with-database` as needed + client content skills; design D14; SC-PACK-230
- [x] 10.2 After staff-save: sync working↔live **or** omit draft status for staff without open author request; keep pending/needs_revision
- [x] 10.3 Mocha and/or vitest SC-PACK-230; `npm test` in touched repo(s)

## 11. Client — map paint race + editor layout (D15–D16)

- [x] 11.1 Skills `work-with-pages`, `work-with-stores` (+ maps), `work-with-styles`, `work-with-test`; design D15–D16; SC-MAP-53/54
- [x] 11.2 Block paint / anti-stale save apply so rapid paints are not lost (SC-MAP-53)
- [x] 11.3 Board-comparable editor field; palette tiles under map with labels under tiles (SC-MAP-54)
- [x] 11.4 Vitest; `npm test`

## 12. Client — Lobby breadcrumb (D17)

- [x] 12.1 Skills `work-with-pages`, `work-with-test`; design D17; SC-BRAND-20
- [x] 12.2 Lobby crumb navigates to lobby from packs/maps/moderation trails
- [x] 12.3 Vitest App breadcrumbs; `npm test`

## 13. Verify packages (follow-up 2)

- [x] 13.1 Client: `npm run lint`, `npm run typecheck`, `npm test`, `npm run build`
- [x] 13.2 Server (если трогали D14): `npm test`, `npm run build`
