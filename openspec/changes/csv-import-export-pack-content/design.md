## Context

See `proposal.md`. Client already edits packs via working draft: cards editor (`ContentPackEditorPage`), nested task-set editor (`ContentPackTasksPage`), add-task-set (`ContentPackAddTaskSetPage`) with `liveCards`. Persist через существующий `saveDraft` / add-task-set put — новых HTTP нет. Модель: `AnswerCard { content, description }`, `ContentTask { question, difficulty, slots→answerCardId }`, ids через `newLocalId`. CSV helper + кнопки import/export уже в коде (D1–D5, D7–D10); follow-up explore пересматривает D6 и добавляет playing-card chrome.

## Goals / Non-Goals

**Goals**

- Чистый parse/serialize CSV (`;`, без escape) + резолв слотов по точному `content`
- Import через Quasar-модалку (формат + file pick + ошибки внутри); Export сразу download, disabled если пусто
- Append-only мутация локального draft; гейт и block missing answers до mutate
- Единый playing-card presentation ответов/заданий на editor, picker, live, staff
- Vitest на helper + UI/smoke по новым SC

**Non-Goals**

- Новые server routes / bulk upload
- RFC4180 quoting, XLSX, JSON
- Replace/merge по id или upsert по тексту
- CSV import на staff / live-only (только визуал карт)
- Клик по телу карты → view modal (later)

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

- **Choice:** Дубликаты `content` при импорте ответов ок. Резолв слота: первый `answerCards.find(c => c.content === slotText)` (без trim/case-fold, кроме возможного trim краёв ячейки CSV как гигиена parse — зафиксировать в helper: trim каждой ячейки после split).
- **Why:** Explore: не усложнять; missing = block.
- **Note:** Trim ячеек — минимальная гигиена parse, не «умный» матч слов.

### D6′ — Quasar import modal with format + in-modal errors (revises D6)

- **Choice:** «Импорт» открывает `q-dialog` с кратким примером формата (answers: `ответ;абзац1;абзац2;…`; tasks: `вопрос;сложность;слот1;слот2;…`) и кнопкой выбора файла (native input внутри модалки). Успех → apply + закрыть модалку. Ошибка (missing answers / unreadable / empty) → текст **в модалке**, draft не менять; модалка остаётся открытой. Export **не** через модалку: сразу download; кнопка Export **disabled**, если соответствующий список пуст.
- **Why:** Автор ожидал подсказку формата и ошибки в одном месте; прежний D6 («только OS picker + banner») был ошибочно зафиксирован как отказ от «эссе правил».
- **Supersedes:** D6 File picker dialog only.
- **Alternatives:** Только tooltip / только page banner — отклонены.

### D7 — Whole-file reject only for missing answer texts

- **Choice:** Отсутствующий текст слота среди карточек → reject всего файла + список недостающих. Пустые question/slot колонки — всё же грузить (submit minima/empty slots как сейчас).
- **Why:** Explore D7/D9.

### D8 — Answer context per surface

- **Choice:** Add-task-set: `liveCards`. TasksPage / creator set: `local.answerCards` draft. Гейт import: length === 0 → disable.
- **Why:** Уже так устроены picker’ы слотов.

### D9 — Export filenames

- **Choice:** Answers: от `pack.title` (sanitize). Tasks: `{title}-tasks-{n}.csv` (n — индекс/номер набора в UI).
- **Why:** Explore open thread 1.

### D10 — Layers / files

| Слой | Точка врезки |
|------|----------------|
| Helper | `src/lib/packContentCsv.ts` — parseAnswers / serializeAnswers / parseTasks / serializeTasks / resolveSlots / downloadCsvText |
| CSV UI | Shared import modal + Export на `ContentPackEditorPage` (answers), `PackTasksCsvControls` (tasks); native file input внутри модалки |
| Card UI | Shared answer/task «playing card» components; wrap-grid на editor, slot picker, live, staff |
| State | Мутация локального `ref` draft на странице → существующий autosave/`saveDraft` / add-task-set save |
| i18n | `src/i18n` keys для CSV labels, format hints, gate, missing-answers, card a11y |
| Errors | Import errors в модалке; page `content.error` / banners для прочих store ошибок |
| Tests | Vitest helper (SC-PACK-210…218); modal + card chrome smoke где дешево |

**Server:** не трогать.

### D11 — Playing-card chrome everywhere

- **Choice:** Ответы и задания рендерятся как скруглённые плитки в `flex`/`grid` wrap-ряду. Карта с двумя половинами: **answers** — верх `content`, низ `description` (если description пуст — **прямоугольник** ~половина высоты, без пустой нижней половины). **tasks** — верх `question`, низ слоты (тексты ответов); **difficulty** бейдж **сверху слева**. На editable surfaces: **карандаш** и **корзина** **сверху справа**; правки только через карандаш (форма/диалог редактирования как сейчас по смыслу). Клик по телу карты для view-модалки — **out of scope** (later). Add-формы остаются **сверху**; сетка карт **ниже**. Длинное description: фиксированный потолок нижней половины + **scroll внутри**. Тот же chrome на slot picker, live view, staff moderation (read-only без edit/delete где не editable).
- **Why:** Explore C1–C9; визуально ближе к игральным картам; единообразие surfaces.
- **Alternatives:** Только editor list; квадрат без description; клик=edit — отклонены.

## Risks / Trade-offs

- **[Risk] `;` в тексте ответа/вопроса** → битые колонки; mitigation: format hint в import-модалке; no escape v1.
- **[Risk] Дубликаты content → неоднозначный слот** → first match; accepted.
- **[Risk] Import пустых tasks/slots** → пользователь не сдаст на модерацию; accepted.
- **[Risk] Export slot с пустым answerCardId** → пустое поле слота в CSV; serialize only filled content; empty slot → empty field.
- **[Risk] Очень длинные description на сетке** → scroll в половине; соседние карты в ряду могут иметь разную высоту при «примерно квадрате» с description — accepted.

## Migration Plan

1. Client: CSV modal UX + playing-card components на всех surfaces; vitest; lint/typecheck.
2. Deploy client only.
3. Rollback: revert client.

## Open Questions

None — D1–D5, D6′, D7–D11 закрыты в explore.
