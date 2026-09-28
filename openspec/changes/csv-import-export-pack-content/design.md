## Context

See `proposal.md`. Client already edits packs via working draft: cards editor (`ContentPackEditorPage`), nested task-set editor (`ContentPackTasksPage`), add-task-set (`ContentPackAddTaskSetPage`) with `liveCards`. Persist через существующий `saveDraft` / add-task-set put — новых HTTP нет. Модель: `AnswerCard { content, description }`, `ContentTask { question, difficulty, slots→answerCardId }`, ids через `newLocalId`. Explore closed: D1–D9.

## Goals / Non-Goals

**Goals**

- Чистый parse/serialize CSV (`;`, без escape) + резолв слотов по точному `content`
- Кнопки Import/Export на трёх поверхностях; общий UX file-dialog
- Append-only мутация локального draft; гейт и block missing answers до mutate
- Vitest на helper (+ тонкие UI/store сценарии по SC)

**Non-Goals**

- Новые server routes / bulk upload
- RFC4180 quoting, XLSX, JSON
- Replace/merge по id или upsert по тексту
- Staff moderation / live-only CSV

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

### D6 — File picker dialog only

- **Choice:** Import → диалог выбора файла; успех → закрыть + apply; ошибка (missing answers / unreadable) → сообщение, draft не менять.
- **Why:** Пользователь отказался от отдельной «модалки правил»; ошибки достаточно.
- **Alternatives:** Модалка с эссе правил — отложена.

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
| Helper | `src/lib/packContentCsv.ts` — parseAnswers / serializeAnswers / parseTasks / serializeTasks / resolveSlots |
| UI | Кнопки + `q-file`/native file input dialog на `ContentPackEditorPage` (answers), `ContentPackAddTaskSetPage` + `ContentPackTasksPage` (tasks); опционально shared component |
| State | Мутация локального `ref` draft на странице → существующий autosave/`saveDraft` / add-task-set save |
| i18n | `src/i18n` keys для labels, gate tooltip, missing-answers error |
| Errors | Store `content.error` + `q-banner` и/или локальный error в диалоге импорта |
| Tests | Vitest helper (SC-PACK-210…218 core); page smoke по skill `work-with-test` где дешево |

**Server:** не трогать.

## Risks / Trade-offs

- **[Risk] `;` в тексте ответа/вопроса** → битые колонки; mitigation: document in UI caption/tooltip; no escape v1.
- **[Risk] Дубликаты content → неоднозначный слот** → first match; accepted.
- **[Risk] Import пустых tasks/slots** → пользователь не сдаст на модерацию; accepted.
- **[Risk] Export slot с пустым answerCardId** → пустое поле слота в CSV; import потом может создать empty slot или требовать текст — serialize only filled content; empty slot → empty field.

## Migration Plan

1. Client helper + i18n + buttons на трёх страницах; vitest; lint/typecheck.
2. Deploy client only.
3. Rollback: revert client.

## Open Questions

None — D1–D9 закрыты в explore; BOM/Excel later при столкновении.
