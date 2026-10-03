## Why

На поверхностях редактирования пака и набора заданий формы создания/правки ответов и вопросов занимают место над списком и (на add-task-set) ещё дублируют live-карточки ответов сверху. Пользователю нужен список с кнопкой «добавить» и знакомая модалка задания, близкая к peek на игровом поле.

## What Changes

- Inline-формы ответа и задания уезжают в модалки; над списком — кнопка добавления; Edit на плитке открывает ту же модалку.
- На add-task-set убирается верхний read-only дубль live-карточек ответов.
- Модалка задания: поля вопроса/сложности, слоты по центру, ниже пул ответов chips как в peek; модалка ответа — только поля без превью плитки.
- CSV остаётся на странице сверху списка заданий.

## Scope

- **Capability ID:** `content/packs`
- **Пакет:** client (`happy-tourist.github.io`) — UX editor ответов, task-set editor, add-task-set
- Поведение compose (слоты, cascade, minima, ACL, moderation, CSV import/export) без смены контракта API
- **Server delta: none.** Persist/save/load остаются через существующий client `stores/content` и уже задокументированный HTTP `content/packs` в main spec (`GET/PUT …/draft`, `…/add-task-set`, staff-save и т.д.). Новый room/message/schema/route **не** вводится; Colyseus не затрагивается.

## Out of scope

- Server / HTTP / Colyseus / peek на `GamePage` (игровая модалка не меняется)
- Редизайн плиток списка (`PackAnswerCardTile` / `PackTaskTile` как карточки сетки)
- Новые поля ответа/задания, изменение правил модерации или CSV-формата
- Любые изменения payload draft / add-task-set / cascade на server

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: UI compose ответов и заданий — модалки вместо inline-форм; add-task-set без верхнего дубля live-карточек; компоновка модалки задания приближена к peek; **MODIFIED** playing-card requirement (SC-PACK-222…227): над списком add control (не inline form); answer pool в task dialog = chips (list tiles без смены)

## Impact

- Client: страницы редактора пака / набора заданий / add-task-set, связанные vitest и skills content-pages / pack-cards
- Server и room-контракт: без изменений (sibling `happy-tourist-server` не трогаем; контракт persist уже в main `content/packs` + `/api/content/packs*`)
- Main spec `openspec/specs/content/packs` — delta UX (SC-PACK-256…262) + MODIFIED полный playing-card requirement (SC-PACK-222…227, carve-out в 222/224); product persist/ACL/cascade — без delta

## References

- Explore: модалки compose + peek-like layout (чат)
- `openspec/specs/content/packs/spec.md` (SC-PACK-107, compose/slots, playing-card tiles, CSV)
- `openspec/specs/game/board/spec.md` (peek modal — эталон компоновки, не меняется)
- Client AGENTS / skills: `work-with-pages/content-pages.md`, `work-with-game-board/peek.md`, `work-with-styles/pack-cards.md`
