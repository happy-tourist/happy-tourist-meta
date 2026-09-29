## Context

See `proposal.md`. CSV helper + import modal + playing-card components уже поставлены (D1–D11). Follow-up explore 2026-09-29: contrast/sizes tiles; catalog + task-set card grids; staff ложный draft; map paint race + layout; Lobby crumb.

## Goals / Non-Goals

**Goals**

- Чистый parse/serialize CSV (`;`, без escape) + резолв слотов по точному `content`
- Import через Quasar-модалку; Export сразу download, disabled если пусто
- Append-only; гейт и block missing answers
- Playing-card tiles с **фиксированными** размерами и читаемым contrast light/dark
- Catalog packs + task-set lists как card grid 100×200 (star TL, status top, Edit right)
- Staff не видит ложный draft после прямого staff-save
- Map paint без потери клеток; поле ~game-board; палитра под полем с labels under tiles
- Breadcrumb Lobby → lobby route
- Vitest / mocha по новым SC

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

- **Choice:** Один helper + общий компонент/composable кнопок+диалога на `ContentPackAddTaskSetPage` и `ContentPackTasksPage`.
- **Why:** Одинаковый UX слотов; не дублировать parse.
- **Alternatives:** Только add-task-set — отклонено.

### D4 — Difficulty column; default 1

- **Choice:** Колонка 2 = difficulty; пусто / не `1|2|3` → `1`.
- **Why:** Difficulty обязателен в модели; автор может не указывать.
- **Alternatives:** Отдельный out-of-band default без колонки — отклонено (колонка нужна для round-trip export).

### D5 — Duplicates allowed; first content match for slots

- **Choice:** Дубликаты `content` при импорте ответов ок. Резолв слота: первый `answerCards.find(c => c.content === slotText)` (без trim/case-fold, кроме trim краёв ячейки CSV).
- **Why:** Explore: не усложнять; missing = block.
- **Note:** Trim ячеек — минимальная гигиена parse.

### D6′ — Quasar import modal with format + in-modal errors (revises D6)

- **Choice:** «Импорт» открывает `q-dialog` с примером формата и file pick; успех → apply + закрыть; ошибка → текст в модалке, draft не менять. Export сразу download; disabled если пусто.
- **Why:** Подсказка формата и ошибки в одном месте.
- **Supersedes:** D6 File picker dialog only.

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
| CSV UI | Shared import modal + Export на editor / `PackTasksCsvControls` |
| Answer/task tiles | Shared tiles; fixed px sizes; contrast light/dark |
| Pack/task-set cards | Catalog + live/editor task-set lists → card grid 100×200 |
| Map editor | `MapEditorPage` + `MapGridPreview`; save race; palette under |
| Breadcrumbs | `App.vue` Lobby crumb → lobby |
| Staff status | Server `withTaskSetModerationStatuses` / staff-save sync **или** client hide draft for staff |
| State | Локальный draft → autosave / staff-save / map quiet save |
| i18n | CSV + card a11y + map palette labels |
| Tests | Vitest / mocha по SC follow-up |

### D11 — Playing-card chrome everywhere

- **Choice:** Ответы и задания — скруглённые плитки в wrap-ряду; split halves; difficulty TL; pencil+delete TR when editable; Add form above grid; description scroll in lower half. Same chrome on picker / live / staff.
- **Why:** Explore C1–C9.
- **Follow-up sizes:** см. D12.

### D12 — Fixed tile pixel sizes (revises approximate sizing in D11)

- **Choice:** Answer **without** description: **100×200** px. Answer **with** description: **200×200** px with horizontal splitter. Task tile (question|slots): **200×200** px. Explicit `background` + `color` (and edit icon colors) so dark mode is not white-on-white (`--q-card-background` fallback `#fff` alone insufficient).
- **Why:** Explore 2026-09-29; contrast bug on dark theme.
- **Alternatives:** rem-only fluid sizing — отклонено для этого follow-up.

### D13 — Catalog packs + task-set lists as 100×200 cards

- **Choice:** Unified packs catalog and task-set lists (live pack + cards editor) render as wrap card grid **100×200**. Chrome: **status** top; **favorite star** top-left where favorites apply (catalog); **Edit** control **top-right only**; body = truncated title / task-set label. Row click / card body open semantics stay as today (catalog open pack; task-set enter/edit rules unchanged). Soft-unpublish / republish / other side actions remain reachable without losing star/edit (do not race navigation).
- **Why:** Explore D1 answers: catalog + task sets both 100×200; Edit only right.
- **Alternatives:** Keep q-list rows — отклонено.

### D14 — Staff must not see false draft after staff-save

- **Choice:** Moderator/admin after staff direct-save MUST NOT see author-facing `draft` / «needs moderation» on sets solely because working≠live. Prefer: after staff-save align working with live **or** when serializing statuses for staff without open author request, omit `draft` from working≠live fingerprint. Keep `pending` / `needs_revision` for real open author requests.
- **Why:** Staff edits publish immediately; stale working vs updated live currently marks draft for staff viewers.
- **Alternatives:** Only hide badge on client — weaker; sync working preferred if cheap.

### D15 — Map paint: block while saving + anti-stale apply

- **Choice:** While a quiet/staff map save is in flight, ignore further cell paints (or queue until unlock). Do **not** apply a save response that is older than the local revision the user has since painted (generation token / dirty-since-request). Goal: rapid second paint must not disappear when the first save returns.
- **Why:** Explore race: flushAutosave overwrote local with stale server grid.
- **Alternatives:** Only debounce longer — insufficient.

### D16 — Map editor layout: board-like field, palette under with labels under tiles

- **Choice:** Editor grid sized closer to in-game board (responsive large square, not tiny 280/360 side-panel). Paint tools as tiles **below** the map; **label under each tile** (start / task / finish). Seat selects and other chrome stay usable without returning palette to a side column as primary.
- **Why:** Explore; main spec already says bottom palette — align UI.
- **Alternatives:** Keep side palette — отклонено.

### D17 — Breadcrumb Lobby navigates to lobby

- **Choice:** Activating the Lobby crumb MUST navigate to the lobby route (`name: 'lobby'`). Use reliable router navigation (fix `to` and/or explicit push) so the crumb is not a dead label.
- **Why:** Explore: click on Lobby crumb did not enter lobby.
- **Capability:** `ui/branding` (extends breadcrumb path behavior).

## Risks / Trade-offs

- **[Risk] `;` в тексте** → format hint; no escape v1.
- **[Risk] Дубликаты content** → first match; accepted.
- **[Risk] Catalog cards 100×200 узкие** → truncated title; accepted start size.
- **[Risk] Staff sync working=live** → may discard author working if mis-applied; only after staff-save path / status omit for staff without open request.
- **[Risk] Blocking paint while save** → slight friction on fast paint; safer than lost cells.

## Migration Plan

1. Client (+ optional server status/sync): follow-up UX; vitest/mocha; lint/typecheck/build.
2. Deploy client (and server if status fix lands there).
3. Rollback: revert commits.

## Open Questions

None — D1–D5, D6′, D7–D17 закрыты в explore.
