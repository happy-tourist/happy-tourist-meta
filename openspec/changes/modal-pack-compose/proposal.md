## Why

На поверхностях редактирования пака и набора заданий формы создания/правки ответов и вопросов занимают место над списком и (на add-task-set) ещё дублируют live-карточки ответов сверху. Пользователю нужен список с кнопкой «добавить» и знакомая модалка задания, близкая к peek на игровом поле. Отдельно: при заполнении слотов задания (compose и peek) нельзя повторно выбрать уже поставленную карточку ответа и она подсвечивается как «использованная» — такого ограничения быть не должно: одна карточка может занимать любое число слотов.

## What Changes

- Inline-формы ответа и задания уезжают в модалки; над списком — кнопка добавления; Edit на плитке открывает ту же модалку.
- На add-task-set убирается верхний read-only дубль live-карточек ответов.
- Модалка задания: поля вопроса/сложности, слоты по центру, ниже пул ответов chips как в peek; модалка ответа — только поля без превью плитки.
- CSV остаётся на странице сверху списка заданий.
- Compose и peek: одна и та же карточка ответа MAY заполнять несколько (в т.ч. все) слотов; chip пула остаётся доступным без disable и без подсветки «уже используется».

## Scope

- **Capability ID:** `content/packs`, `game/board`
- **Пакет:** client (`happy-tourist.github.io`) — UX editor ответов, task-set editor, add-task-set; плюс peek-модалка на Game (выбор карточек в слоты)
- Поведение compose (слоты, cascade, minima, ACL, moderation, CSV import/export) без смены контракта API
- Peek: клиентский выбор чипов в слоты; server `peekPlace` / `peekSubmit` и порядок слотов **без** смены контракта (уникальность на server и так не требуется)
- **Server delta: none.** Persist/save/load и room peek messages остаются через существующие пути. Новый room/message/schema/route **не** вводится.

## Out of scope

- Server / HTTP / Colyseus protocol / schema fields (логика уникальности слотов на server не вводилась — не трогаем)
- Редизайн плиток списка (`PackAnswerCardTile` / `PackTaskTile` как карточки сетки)
- Новые поля ответа/задания, изменение правил модерации или CSV-формата
- Любые изменения payload draft / add-task-set / cascade на server
- Смена правил правильности peek (порядок слотов) — только снятие client-only uniqueness UI

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: UI compose ответов и заданий — модалки вместо inline-форм; add-task-set без верхнего дубля live-карточек; компоновка модалки задания приближена к peek; **MODIFIED** playing-card requirement (SC-PACK-222…227); **ADDED** reuse одной карточки ответа в нескольких слотах compose без used-state chrome (SC-PACK-263)
- `game/board`: peek answer chips — повторный выбор той же карточки в пустые слоты; без disable/подсветки «уже используется» (SC-BOARD-49)

## Impact

- Client: страницы редактора пака / набора заданий / add-task-set, `PackTaskComposeDialog`, Game peek UI, связанные vitest и skills content-pages / pack-cards / peek
- Server и room-контракт: без изменений (sibling `happy-tourist-server` не трогаем; persist уже в main `content/packs`; peek place/submit уже допускают одинаковые `answerCardId` в разных слотах)
- Main specs: delta `content/packs` (SC-PACK-256…263 + MODIFIED playing-card) и delta `game/board` (SC-BOARD-49)

## References

- Explore: модалки compose + peek-like layout; reuse одной карточки ответа в слотах (чат)
- `openspec/specs/content/packs/spec.md` (SC-PACK-107, compose/slots, playing-card tiles, CSV)
- `openspec/specs/game/board/spec.md` (peek modal SC-BOARD-43…46)
- Client AGENTS / skills: `work-with-pages/content-pages.md`, `work-with-game-board/peek.md`, `work-with-styles/pack-cards.md`
