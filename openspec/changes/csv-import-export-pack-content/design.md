## Context

See `proposal.md`. CSV helper + tiles + catalog cards + staff draft + map paint race + Lobby crumb уже поставлены (D1–D17). Follow-up explore 2026-09-29 вечер: убрать import modal/tooltip → framed CSV panel; пересмотреть размеры/split/actions плиток; description в каталоге; maps list cards; центрирование редактора карты; миниатюра selected map в Lobby.

## Goals / Non-Goals

**Goals**

- Чистый parse/serialize CSV (`;`, без escape) + резолв слотов по точному `content`
- Import/Export в **рамке** с format hint и inline-ошибкой; Import → сразу file picker; без tooltip / без `q-dialog`
- Append-only; гейт и block missing answers
- Playing-card tiles: **150×200** / **300×200**, высота **200**, вертикальный split, шрифт ~×2, слоты в ряд, actions снизу текстом
- Catalog + task-set cards **150×200**; catalog показывает truncated description; star TL; actions снизу
- Maps list как card grid (миниатюра сверху, capacity ниже)
- Map editor: весь столбец по центру; seat selects нормальной ширины
- Lobby create: selected map показывает миниатюру
- Staff без ложного draft; Lobby crumb; paint anti-stale (уже)
- Vitest / mocha по SC

**Non-Goals**

- Новые server routes / bulk upload CSV
- RFC4180 quoting, XLSX, JSON
- Replace/merge по id или upsert по тексту
- CSV import на staff / live-only
- Клик по телу answer/task → view modal
- Drag-paint; смена размерности карты

## Decisions

### D1 — Semicolon dialect without escaping

- **Choice:** `line.split(';')`; `;` внутри текста ломает колонки; без кавычек.
- **Why:** Простой авторский обмен; пользователь согласился на «`;` всегда разделитель».
- **Alternatives:** RFC4180 / `\;` — out of scope.

### D2 — Append only

- **Choice:** Импорт ответов → `answerCards.push(…)` с `newLocalId('card')`. Импорт заданий → `tasks.push(…)` текущего набора.
- **Why:** Replace с новыми id снёс бы слоты (cascade SC-PACK-07).
- **Alternatives:** Full replace / upsert по content — отклонены.

### D3 — Shared task CSV controls on both task surfaces

- **Choice:** Один helper + общий CSV-panel компонент на `ContentPackAddTaskSetPage` и `ContentPackTasksPage` (и answers на editor).
- **Why:** Одинаковый UX; не дублировать parse.
- **Alternatives:** Только add-task-set — отклонено.

### D4 — Difficulty column; default 1

- **Choice:** Колонка 2 = difficulty; пусто / не `1|2|3` → `1`.
- **Why:** Difficulty обязателен в модели; автор может не указывать.
- **Alternatives:** Out-of-band default без колонки — отклонено.

### D5 — Duplicates allowed; first content match for slots

- **Choice:** Дубликаты `content` при импорте ответов ок. Резолв слота: первый `answerCards.find(c => c.content === slotText)`.
- **Why:** Explore: не усложнять; missing = block.
- **Note:** Trim ячеек — минимальная гигиена parse.

### D6″ — Framed CSV panel; direct file picker; inline errors (revises D6′)

- **Choice:** Export + Import в общей визуальной **рамке**. Под кнопками — постоянный текст рекомендаций формата CSV (answers / tasks). **Нет** `q-tooltip` на Import. **Нет** `q-dialog` импорта. Клик Import (когда enabled) сразу открывает системный file picker. Ошибка импорта — текст **внутри рамки**; draft не менять. Успех — apply + сброс ошибки в рамке. Export сразу download; disabled если пусто.
- **Why:** Explore 2026-09-29 вечер: модалка и tooltip лишние; hint и ошибка рядом с кнопками.
- **Supersedes:** D6′ Quasar import modal.
- **Package:** client — `ContentPackEditorPage`, `PackTasksCsvControls`; удалить/не использовать `PackCsvImportDialog` как primary path.

### D7 — Whole-file reject only for missing answer texts

- **Choice:** Отсутствующий текст слота → reject всего файла + список недостающих. Пустые question/slot колонки — грузить.
- **Why:** Explore D7/D9.

### D8 — Answer context per surface

- **Choice:** Add-task-set: `liveCards`. TasksPage / creator set: `local.answerCards` draft. Гейт import: length === 0 → disable.
- **Why:** Уже так устроены picker’ы слотов.

### D9 — Export filenames

- **Choice:** Answers: от `pack.title` (sanitize). Tasks: `{title}-tasks-{n}.csv`.
- **Why:** Explore open thread 1.

### D10 — Layers / files

| Слой | Точка врезки |
|------|----------------|
| Helper | `src/lib/packContentCsv.ts` |
| CSV UI | Framed Export+Import + format hint + inline error (editor + `PackTasksCsvControls`) |
| Answer/task tiles | Shared tiles; 150/300×200; vertical split; bottom text actions |
| Pack/task-set cards | Catalog + task-set lists → 150×200; catalog description; actions bottom |
| Maps list | `MapsListPage` → card grid (mini top, capacity below) |
| Map editor | `MapEditorPage` centered column; seat select width |
| Lobby create | `LobbyPage` map `q-select` selected-item + option mini |
| Breadcrumbs | `App.vue` Lobby crumb → lobby |
| Staff status | Server omit false draft / twin clear (уже) |
| i18n | CSV hints + button labels (Edit/Delete словами) |
| Tests | Vitest / mocha по SC follow-up |

### D11 — Playing-card chrome everywhere

- **Choice:** Ответы и задания — скруглённые плитки в wrap-ряду на editor / picker / live / staff. Add form above grid.
- **Follow-up layout:** см. D12′ / D18.

### D12′ — Tile sizes 150/300, vertical split, larger type (revises D12)

- **Choice:** Answer **без** description: **150×200**. Answer **с** description: **300×200** с **вертикальным** разделителем (content | description). Task: **300×200**, вертикальный split (question | slots). Высота всегда **200**. Размер шрифта content/question/slots ~**×2** относительно прежних ~0.8–0.9rem. Explicit light/dark bg + fg (contrast).
- **Why:** Explore 2026-09-29 вечер.
- **Supersedes:** D12 100/200 horizontal split.

### D13′ — Catalog + task-set cards 150×200 + description + bottom actions (revises D13)

- **Choice:** Catalog и task-set lists — wrap grid **150×200**. Status **сверху**. Favorite star **слева сверху** (catalog). **Все action-кнопки** карты (Edit, Unpublish, Republish, …) — **внизу**, **словами**, друг под другом, на всю ширину карточки. Справа сверху пусто. Catalog body: truncated title + truncated **description**. Task-set body: truncated label. Open/navigation semantics без изменений.
- **Why:** Explore Q2/Q5/Q6.
- **Supersedes:** D13 Edit top-right; 100×200.

### D14 — Staff must not see false draft after staff-save

- **Choice:** Omit `draft` for staff without open author request; clear twin working after staff-save when appropriate. Keep `pending` / `needs_revision`.
- **Status:** реализовано в change.

### D15 — Map paint: block while saving + anti-stale apply

- **Choice:** Block paint while save in flight; skip stale save apply via local revision.
- **Status:** реализовано (+ flushAutosave wait).

### D16′ — Map editor centered column + usable seat selects (extends D16)

- **Choice:** Под полем — palette with labels under tiles (уже). Весь столбец редактора (поле + палитра + селекты seats + chrome) **центрировать** горизонтально на странице как игровое поле. Селекты players / tourists — **достаточная ширина** (не схлопнутые `col-6` в узком `max-width: 360px` без запаса).
- **Why:** Explore Q7 + «селекты схлопнулись».
- **Package:** client `MapEditorPage.vue`.

### D17 — Breadcrumb Lobby navigates to lobby

- **Choice:** Lobby crumb → `name: 'lobby'`.
- **Status:** реализовано.

### D18 — Bottom full-width text actions on answer/task tiles

- **Choice:** Убрать round icon Edit/Delete из углов. Когда editable — внизу плитки две (или более) кнопки **словами** (`Редактировать` / `Удалить`), stacked, **width 100%** плитки. Difficulty / status badges остаются сверху.
- **Why:** Explore Q2/Q6.
- **Package:** `PackAnswerCardTile`, `PackTaskTile` (+ list cards per D13′).

### D19 — Task slots as horizontal filled chips

- **Choice:** В правой половине task tile слоты — **в ряд** (wrap), визуально как заполненные slot-чипы, не вертикальный список строк.
- **Why:** Explore.
- **Package:** `PackTaskTile`.

### D20 — Maps list as cards

- **Choice:** `MapsListPage` — wrap card grid вместо `q-list` rows. Карточка: сверху **миниатюра** grid; ниже `players×tourists` (и author/status по необходимости); action-кнопки снизу словами на всю ширину (как D13′).
- **Why:** Explore «набор карт тоже в виде карт».
- **Package:** client `MapsListPage` (+ shared map list card component если нужно).

### D21 — Lobby create map select shows selected miniature

- **Choice:** В create-game `q-select` карты: option list уже с mini preview; **selected-item** (закрытое состояние) MUST тоже показывать миниатюру выбранной карты рядом с label (не только текст).
- **Why:** Explore.
- **Capability:** `lobby/rooms`.
- **Package:** client `LobbyPage.vue`.

## Risks / Trade-offs

- **[Risk] `;` в тексте** → format hint в рамке; no escape v1.
- **[Risk] 150×200 + bottom buttons** → меньше места для title/description; truncate + clamp.
- **[Risk] Font ×2 на узкой половине** → overflow/scroll внутри половины.
- **[Risk] Staff sync working=live** → только staff-save path (уже).
- **[Risk] Blocking paint while save** → slight friction (уже принято).

## Migration Plan

1. Client UX follow-up + vitest; lint/typecheck/build.
2. Deploy client (server already has D14).
3. Rollback: revert commits.

## Open Questions

None — Q1–Q7 закрыты (высота 200; все actions вниз; Import сразу picker; fold в этот change; star TL; badges top; editor column centered).
