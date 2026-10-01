## 1. Client — task-set summary card

- [x] 1.1 Прочитать `design.md`, delta `specs/content/packs/spec.md` (SC-PACK-229/239…242), skills `client/work-with-styles` + `pack-cards.md`, `client/work-with-pages/content-pages.md`, текущие `PackListCardTile.vue` / live+editor grids
- [x] 1.2 Добавить dedicated task-set summary tile (отдельно от catalog `PackListCardTile`): статус сверху, title без автора, rows count+difficulty dots, outline+icon actions снизу, ширина ~150–160 / высота с запасом, light/dark contrast, cascade-gap class; verify компонент монтируется и размеры/контраст в unit-тесте
- [x] 1.3 Подключить tile на live pack и cards editor task-set grids (сохранить click/open, soft-unpublish/republish/Edit ACL); catalog packs не менять; verify SC-PACK-229/130 сценарии в vitest list-cards / live+editor
- [x] 1.4 i18n: короткие бейджи (СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …) на карточках; labels actions с иконками; verify SC-PACK-240/241 в vitest

## 2. Client — убрать автора и соавторов

- [x] 2.1 Прочитать delta SC-PACK-135 и места `taskSetLabelFrom` / `coauthorLabels` (live, editor, staff hub headings, tasks header)
- [x] 2.2 Заменить лейблы на «Набор заданий {n}» без автора/соавторов на всех content-поверхностях; `coauthorLabels` при создании/save оставлять `[]`; verify SC-PACK-135 vitest (FollowUp5 / AuthorEditTake / list-cards)

## 3. Client — lobby labels

- [x] 3.1 Прочитать delta `specs/lobby/rooms/spec.md` (SC-LOBBY-24/26), `LobbyPage` create picker + room list metadata
- [x] 3.2 Убрать автора набора из create task-set options и из отображения task-set labels в listing; map author не трогать; verify SC-LOBBY-24/26 в `LobbyCreateWire` (и смежных) vitest

## 4. Client — verify gate

- [x] 4.1 Обновить/добавить vitest по Traceability SC-PACK-229/239…242/135/132 и SC-LOBBY-24/26; `npm test` в client — зелёный
- [x] 4.2 `npm run lint` и `npm run typecheck` в client — без ошибок по затронутым файлам

## 5. Server — optional coauthor silence

- [x] 5.1 Прочитать `design.md` Decision 3/7; при необходимости в persist task set всегда писать `coauthorLabels: []` (без миграции колонки); verify существующие mocha content packs не падают (`npm test` server на затронутых suite)
- [x] 5.2 skipped — UI-only — сервер не менялся; create/list не зависят от server copy author (`coauthorLabels: []` с клиента; lobby — ordinal `taskSetLabel`)

## 6. Meta skills touch-up

- [x] 6.1 Обновить `.agents/skills/client/work-with-styles/pack-cards.md` (и при необходимости content-pages/content store notes) под task-set summary chrome vs catalog 150×200; verify текст skills согласован с design

## 7. Client — mock polish (hash / colors / dividers / short actions)

- [x] 7.1 Прочитать `design.md` Decision 3/5/8 и delta SC-PACK-135/239/241/243 + SC-LOBBY-24/26; текущие `PackTaskSetCardTile.vue`, live/editor action labels, i18n `taskSetLabel` / `taskSetCard*` / `unpublish` / `republish`
- [x] 7.2 i18n: `taskSetLabel` → «Набор заданий #{n}» везде; total «Заданий:»; diff labels «Лёгкие:» / «Средние:» / «Сложные:»; card-only keys «Снять» и «Вернуть» (pack/catalog `unpublish` / confirms не трогать); verify строки в unit/i18n assertions
- [x] 7.3 `PackTaskSetCardTile`: leading Material `description` на total-row; divider после total; outline empty dots + filled green/amber/red; ослабить слишком dense actions; без нового hover-polish / без scale; verify SC-PACK-239/242/243 в `PackTaskSetCardTile` vitest
- [x] 7.4 Live + editor: soft-unpublish/republish на карточке → short keys «Снять» / «Вернуть»; verify SC-PACK-241 в list-cards / FollowUp vitest
- [x] 7.5 Lobby create/list уже на `taskSetLabel` — убедиться что `#{n}` виден в SC-LOBBY-24/26 vitest (`LobbyCreateWire` и смежные)
- [x] 7.6 Обновить `pack-cards.md` (+ localization/content notes при нужде): hash, цвета dots, dividers, short card actions, placeholder icon + reserved SVG names; `npm test` + `npm run lint` + `npm run typecheck` в client — зелёные по затронутому

## 8. Client — layout polish (pale dividers / columns / fill / slim actions)

- [x] 8.1 Прочитать `design.md` Decision 5/9 + Visual Spec и delta SC-PACK-239/241/244; текущие `PackTaskSetCardTile.vue` + live/editor action `q-btn`
- [x] 8.2 `PackTaskSetCardTile`: бледные dividers после total, **между** каждыми difficulty rows и над actions; колонка lead = ширина 3 dots + icon по центру; labels на одной левой вертикали; counts справа; vertical fill (без badge — пусто только top); verify SC-PACK-239/244 в vitest
- [x] 8.3 Live + editor: slim outline actions ~28–32 CSS px (`dense` и/или min-height); несколько actions стопкой как сейчас; verify SC-PACK-241 slim в list-cards / tile tests
- [x] 8.4 Обновить `pack-cards.md` (+ notes): pale multi-row dividers, column grid, fill, slim actions; `npm test` + `npm run lint` + `npm run typecheck` в client — зелёные по затронутому

## 9. Client — spacing + themed hover (Visual Spec)

- [x] 9.1 Прочитать `design.md` Decision 10 + Visual Spec (точные CSS px) и delta SC-PACK-245/246; текущие `PackTaskSetCardTile.vue` gaps / `:hover`
- [x] 9.2 `PackTaskSetCardTile`: увеличить внутренние отступы title→stats и air вокруг pale dividers / stats-rows по Visual Spec (grid gutter списков **не** трогать); min-height под resting ~206+; verify SC-PACK-245 в vitest
- [x] 9.3 Hover без scale: light — тёмная рамка + soft lift shadow; dark — светло-серая рамка (~`#bdbdbd` / Visual Spec), не `--q-secondary`; иконки/dots без перекраса; verify SC-PACK-246 в vitest
- [x] 9.4 Обновить `pack-cards.md` (+ notes): spacing tokens + themed hover; `npm test` + `npm run lint` + `npm run typecheck` в client — зелёные по затронутому

## 10. Client — spacing retune + wire custom SVG icons

- [x] 10.1 Прочитать `design.md` Decision 11/12 + Visual Spec (retune + **Цвета** + **Icon ink / sizes**) и delta SC-PACK-247/248; текущие `PackTaskSetCardTile.vue` tokens / Material placeholders; assets в `src/assets/content/task-set-*.svg`
- [x] 10.2 Spacing retune: status→title ~6–8 CSS px; `--pack-ts-title-gap` 14→7; `--pack-ts-divider-air` 5→2–3 (grid gutter **не** трогать); verify SC-PACK-247 в vitest
- [x] 10.3 Wire SVG: total-row `task-set-card-tasks.svg` (14px); badges revise/pending/unpublished/draft (~12px) на live+editor; ink через `currentColor` / CSS mask по Visual Spec (muted/pending hex); убрать Material placeholders; verify SC-PACK-248 + color assertions в vitest
- [x] 10.4 Обновить `pack-cards.md` (+ content-pages/localization notes при нужде): retune tokens + SVG paths/map + color/ink table; `npm test` + `npm run lint` + `npm run typecheck` в client — зелёные по затронутому