## Why

На поверхностях редактирования пака и набора заданий формы создания/правки ответов и вопросов занимают место над списком и (на add-task-set) ещё дублируют live-карточки ответов сверху. Пользователю нужен список с кнопкой «добавить» и знакомая модалка задания, близкая к peek на игровом поле. Отдельно: при заполнении слотов задания (compose и peek) нельзя повторно выбрать уже поставленную карточку ответа и она подсвечивается как «использованная» — такого ограничения быть не должно: одна карточка может занимать любое число слотов. Ещё: у автора add-task-set черновик сейчас появляется только после Cancel уже отправленной модерации; если уйти со страницы до Submit — работа теряется. Нужен тот же author-facing черновик **с первого изменения**, с удалением сверху и видимостью только автору.

## What Changes

- Inline-формы ответа и задания уезжают в модалки; над списком — кнопка добавления; Edit на плитке открывает ту же модалку.
- На add-task-set убирается верхний read-only дубль live-карточек ответов.
- Модалка задания: поля вопроса/сложности, слоты по центру, ниже пул ответов chips как в peek; модалка ответа — только поля без превью плитки.
- CSV остаётся на странице сверху списка заданий.
- Compose и peek: одна и та же карточка ответа MAY заполнять несколько (в т.ч. все) слотов; chip пула остаётся доступным без disable и без подсветки «уже используется».
- Add-task-set: любое изменение тихо сохраняет **неполный** never-live черновик (minima только на Submit); уход без модерации сохраняет draft; кнопка удаления сверху; Cancel непубликованного цикла снова даёт draft; в списке/фильтре черновиков — только автору.

## Scope

- **Capability ID:** `content/packs`, `game/board`
- **Пакет:** client (`happy-tourist.github.io`) — UX editor ответов, task-set editor, add-task-set; peek на Game; quiet autosave + delete draft на AddTaskSet; list draft visibility для автора
- **Пакет:** server (`happy-tourist-server`) — persist pre-submit never-live draft (relax put validation vs submit), GET/ghost restore для draft, discard/delete draft, Cancel→draft без смены staff-open semantics
- Compose UI / peek chip reuse без смены room peek protocol
- Один never-live add-task-set цикл на автора на пак (как сейчас)

## Out of scope

- Colyseus room/message/schema для peek (уникальность слотов на server и так не требуется)
- Редизайн плиток списка (`PackAnswerCardTile` / `PackTaskTile`)
- Новые поля ответа/задания, изменение правил approve/needs_revision или CSV-формата
- Несколько параллельных never-live drafts одного автора на одном паке
- Soft-unpublish published set → «черновик» (это unpublished, не add-task-set draft)

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: UI compose модалки; chip reuse compose (SC-PACK-256…263); **ADDED** pre-submit never-live draft persist / restore / delete / author list draft (SC-PACK-264…268); **MODIFIED** add-task-set put vs submit minima
- `game/board`: peek answer chips reuse без used-state (SC-BOARD-49) — уже в этом change

## Impact

- Client: compose pages / dialogs, Game peek, `ContentPackAddTaskSetPage` autosave + top delete, store `saveAddTaskSet` / discard, list draft filter, vitest + skills
- Server: `putAddTaskSet` / `getAddTaskSet` / never-live ghost / cancel / new discard path; mocha
- Main specs: delta `content/packs` (+ уже существующие compose/peek deltas) и `game/board` SC-BOARD-49

## References

- Explore (чат): add-task-set draft с первого изменения; delete сверху; неполный put; Cancel→draft; list только автору
- `openspec/specs/content/packs/spec.md` (SC-PACK-107/108, cancel→draft 175…179/196/197, neverLive 188…190)
- Client/server skills: content-pages, content store, server work-with-routes / structure
