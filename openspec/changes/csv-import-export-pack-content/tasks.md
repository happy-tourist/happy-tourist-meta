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
