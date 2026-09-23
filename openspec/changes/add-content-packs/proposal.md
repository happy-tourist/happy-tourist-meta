## Why

Dual answers/tasks уже в runtime, но автору и staff плохо видно статус модерации (особенно reject + комментарий), интерфейс дёргается от строки «Сохранение…», submit доступен когда нечего слать, а staff hub разворачивает задания вместо списка как у автора. Нужен UX-polish страниц **набора карточек** / заданий и модерации без смены домена.

## What Changes

- Клиент: убрать верхнюю строку autosave; loading на кнопках; submit answers/tasks disabled без dirty / minima; tooltip на создание заданий; нельзя добавить вопрос без выбранного ответа в слоте.
- Статусы на страницах карточек и заданий: на модерации / отклонено (нужна доработка) / одобрено; тред сообщений **внизу каждой** страницы (answers и tasks отдельно).
- Метки «нужна модерация» на **изменённых** task set (server-side, переживает F5); submit tasks по-прежнему **весь** список заданий пака.
- Staff: hub как у автора (карточки + список task set с пометками) → клик на страницу заданий; approve/reject на странице карточек и на странице заданий.
- Lock: пока answers pending у A — **доотправка answers** только у A; **другие** не сабмитят tasks (таски опираются на ответы). Автор A по-прежнему может сабмитить tasks при своём answers pending (порядок staff: tasks → answers).
- Copy: страница ответов называется **«Набор карточек»** (title/description там).
- Каталог по-прежнему только одобренное; статусы модерации — в editor, не в каталоге.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: editor/staff UX polish; status + embedded threads; per-task-set needs-moderation marks; pending-author-only answers resubmit; block others’ tasks submit while answers pending; rename cards page

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client + server (+ meta AGENTS/skills при необходимости)
- Сохранить dual submit, dirty-answers lock, collection-first, catalog after answers approve, staff answers-only queue
- Server: `tasksDirty` / per-set dirty flags vs last tasks snapshot; draft GET отдаёт статусы заявок (pending|rejected|approved cycle) + threads для UI; enforce «чужой не сабмитит tasks при answers pending»
- Client: карточки / задания / staff nested page; i18n «Набор карточек»

## Out of scope

- Room↔pack, peek runtime, media, hard delete, transfer ownership
- Отдельная staff-очередь «только tasks»
- Partial submit только изменённых task sets (submit остаётся паком)
- Смена порядка staff approve (tasks → answers)

## Impact

- Client: ContentPack* / ContentStaff* pages, Pinia content, i18n
- Server: draft/submit/moderation payloads + тесты SC-PACK
- Meta: точечно skills/AGENTS hints

## References

- Explore 2026-09-23: UX jitter, status visibility, staff list→tasks page, Q1–Q5b decisions
- Prior dual flow: `add-content-packs` sections 1–6 (implemented)
- Карта путей: `docs/projects-map.md`
