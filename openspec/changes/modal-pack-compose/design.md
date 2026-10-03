## Context

См. `proposal.md` — Why. Пакет: **client** (`../happy-tourist.github.io`).

Уже в runtime (apply блоков 1–3):

- `ContentPackEditorPage` — answer compose в `q-dialog` (fields only); add над `answer-card-grid`.
- Shared `PackTaskComposeDialog` на Tasks / AddTaskSet: question/difficulty → centered `.peek-slot-like` → chip pool; CSV на странице; верхний live-grid на AddTaskSet убран.
- Chip pool compose и peek на `GamePage` всё ещё гасят / подсвечивают карточку после первого размещения (`isCardPlaced` / `isAnswerPlaced`) — **снимаем** (revision).

Store/API (`stores/content`) и cascade/minima не меняются. Server `peekPlace` / `peekSubmit` уже допускают одинаковый `answerCardId` в разных слотах.

### Server / HTTP (explicit non-touch)

- **Delta server: нет.** Модалки compose и persist — те же пути. Peek messages / schema не меняются.
- Product rules cascade/minima/ACL/CSV — main `content/packs`. Correct peek order — main `game/board` SC-BOARD-43/44.

## Goals / Non-Goals

**Goals:**

- Формы ответа и задания только в `q-dialog`; над списками — кнопки добавления; Edit плитки → та же модалка.
- Модалка задания: layout как peek (слоты по центру, chip-пул ответов).
- Модалка ответа: `content` + `description`, без preview плитки.
- Убрать верхний `live-answer-card-grid` на add-task-set.
- CSV (`PackTasksCsvControls`) на странице сверху списка заданий.
- Compose + peek: одна карточка ответа MAY заполнять любое число слотов; без disable и без used-state chrome на chip.

**Non-Goals:**

- Менять server / CSV формат / ACL / moderation / правила правильности порядка слотов.
- Редизайн resting chrome плиток списка.
- Обязательный shared Vue-компонент compose↔peek.

## Decisions

### D1 — Две модалки compose на страницах (или thin components)

- **Choice:** Compose в `q-dialog`; Tasks/AddTaskSet — shared `PackTaskComposeDialog`; Editor answers — локальный dialog.
- **Why:** одна форма на create/edit; меньше вертикального шума.
- **Alt:** `q-expansion` — отвергнуто.

### D2 — Task dialog layout ≈ peek

- **Choice:** question + difficulty → **centered** slot row (`.peek-slot-like` + `justify-center`) → chip pool (`q-chip`, label = `content`). Не `PackAnswerCardTile` / не `slot-picker-grid`. Add/remove slot рядом со слотами; Save/Cancel в footer.
- **Why:** «как модалка выполнения».
- **Reuse (revision):** chip click → fill next empty slot; **не** disable и **не** outline/primary «already placed» — см. D8.
- **Canon carve-out:** answer pool в task compose dialog = chips (MODIFIED SC-PACK-222/224); list tiles без смены.

### D3 — Answer dialog: fields only

- **Choice:** `q-input` content + description; без preview плитки.
- **Why:** explore.

### D4 — Remove add-task-set top live grid

- **Choice:** Удалить `live-answer-card-grid` + hint / empty stub; live cards — chip-пул dialog + CSV context.
- **Why:** дубль не нужен; SC-PACK-107 add-only сохраняется.

### D5 — Page chrome: add button + CSV + list

- **Choice:** Tasks/AddTaskSet: `[+ Добавить вопрос]` → CSV (если не viewOnly) → `task-card-grid`. Editor: CSV → `[+ Добавить карточку]` → `answer-card-grid`.
- **Gates:** add/Edit compose только когда compose allowed.

### D6 — i18n / a11y

- **Choice:** reuse `content.*` / `game.*` keys; persist disabled + tooltip SC-PACK-119 на Save в dialog.

### D7 — Main-spec / skill supersede (compose shell)

- **Choice:** MODIFIED Playing-card SC-PACK-222…227 + ADDED 256…262; skills pack-cards / content-pages / content — dialog compose + chip-pool carve-out (уже в runtime/skills).

### D8 — Reusable answer cards in slots (compose + peek)

- **Choice:** Убрать client uniqueness: в `PackTaskComposeDialog` — `isCardPlaced` из disable/clickable/outline/color и early-return в `pickAnswer`; на `GamePage` peek — аналогично `isAnswerPlaced` / `:disable` / `:outline` / `:color` / early-return в `onPeekAnswerClick`. Chip визуально одинаковый независимо от того, стоит ли карточка в слотах (всегда outline или единый нейтральный вид без «consumed»). Клик по chip при наличии пустого слота снова кладёт ту же карточку; полный ряд слотов — no-op. Clear слота кликом по filled slot без изменений.
- **Why:** explore: «везде нет ограничения»; «не надо подсвечивать что уже используется»; иначе задания с повторяющимися слотами непроходимы в peek.
- **Alt:** soft highlight без disable — отвергнуто (продукт: без подсветки).
- **Server:** no-op (уже ok). Specs: SC-PACK-263 + SC-BOARD-49. Skills: `peek.md`, content-pages / pack-cards — убрать формулировки «disable when already placed».

## Risks / Trade-offs

- [Chip pool хуже показывает description] → Mitigation: label = content (как peek).
- [Дублирование Tasks vs AddTaskSet] → Mitigation: shared dialog (уже).
- [Регрессия тестов на unique chips] → Mitigation: vitest SC-PACK-263 / SC-BOARD-49; обновить ассерты на disable/outline.
- [Игрок не видит «сколько раз карточка уже стоит»] → Mitigation: смотреть на слоты (продуктовый выбор).

## Migration Plan

- Только client deploy; server не трогаем.
- Rollback: git revert client (compose dialogs + chip uniqueness).

## Technical prerequisites

- Нет новых npm deps / внешних сервисов.
- Нет новых server endpoints / schema / room messages.
- Explore: reuse везде + без used-state chrome — закрыто.

## Implementation touchpoints

- Client: `PackTaskComposeDialog.vue`, `GamePage.vue` (peek chips), уже существующие compose pages
- Skills: `.agents/skills/client/work-with-game-board/peek.md`, при drift — content-pages / pack-cards / content (убрать «disable when already placed»)
- Tests: vitest SC-PACK-263 (+ compose suites); SC-BOARD-49 (Game peek / board suites); lint/typecheck client
- Чеклист — `tasks.md` §4 (новые пункты); §1–3 уже `[x]`
