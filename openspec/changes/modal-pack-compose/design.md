## Context

См. `proposal.md` — Why. Пакет: **client** only (`../happy-tourist.github.io`).

Сейчас:

- `ContentPackEditorPage` — inline `q-form` ответа над `answer-card-grid`.
- `ContentPackTasksPage` / `ContentPackAddTaskSetPage` — inline compose задания (`compose-slot-row` + `slot-picker-grid` из `PackAnswerCardTile`) над списком `PackTaskTile`.
- Add-task-set дополнительно рисует `live-answer-card-grid` (read-only дубль тех же live cards).
- Peek на `GamePage`: `q-dialog` → difficulty → question → `.peek-slots` → `.peek-answers` как `q-chip`.

Store/API (`stores/content`) и cascade/minima не меняются — только оболочка compose UI.

### Server / HTTP (explicit non-touch)

- **Delta server: нет.** Модалки вызывают те же save/load paths, что и inline-формы сегодня (`stores/content` → `PUT/GET /api/content/packs/:id/draft`, add-task-set endpoints, staff-save при staff Edit).
- Product rules (cascade слотов, minima submit, ACL compose / add-only, CSV parse на client) остаются каноном main `openspec/specs/content/packs` (SC-PACK-04…07 / 107 / 119 / 213…220 и смежные) — этот change их не переопределяет.
- Colyseus rooms / schema / peek messages не затрагиваются (peek на `GamePage` — только визуальный эталон layout).

## Goals / Non-Goals

**Goals:**

- Формы ответа и задания только в `q-dialog`; над списками — кнопки добавления; Edit плитки → та же модалка.
- Модалка задания: layout как peek (слоты по центру, ниже chip-пул ответов).
- Модалка ответа: `content` + `description`, без preview плитки.
- Убрать верхний `live-answer-card-grid` на add-task-set.
- CSV (`PackTasksCsvControls`) остаётся на странице сверху списка заданий.

**Non-Goals:**

- Менять `GamePage` peek / server / CSV формат / правила ACL и moderation.
- Редизайн resting chrome плиток списка.
- Обязательный shared Vue-компонент с peek (допустимо визуально близко без выноса GamePage).

## Decisions

### D1 — Две модалки compose на страницах (или thin components)

- **Choice:** Вынести compose в `q-dialog` на существующих страницах; при дублировании Tasks/AddTaskSet — общий thin component (например `PackTaskComposeDialog`) с props form/slots/cards + emits save/cancel. Ответ — аналогично на Editor (локальный dialog или `PackAnswerComposeDialog`).
- **Why:** одна форма на create/edit; меньше вертикального шума на странице.
- **Alt:** всегда-visible form в `q-expansion` — отвергнуто (explore: нужна модалка).

### D2 — Task dialog layout ≈ peek

- **Choice:** Внутри dialog: inputs question + difficulty → **centered** slot row (reuse `.peek-slot-like` / SC-PACK-237 chrome; host row MUST center like GamePage `.peek-slots` — global `.peek-slot-like-row` today has no `justify-center`, so dialog row adds `justify-center` or equivalent) → пул ответов как compact chips (mirror GamePage `.peek-answers` / `q-chip`: label = `content`, click → fill next empty slot / disable or outline when already placed; description на chip не обязателен). Не `PackAnswerCardTile` grid и не `data-testid="slot-picker-grid"`. Add/remove slot controls рядом со слотами. Save/Cancel в footer dialog (`q-card-actions`).
- **Why:** продуктовое «как модалка выполнения».
- **Alt:** оставить playing-card picker в модалке — отвергнуто (explore).
- **Canon carve-out:** это **намеренно** сужает main SC-PACK-222/224 «slot picker = playing-card» и skill `work-with-styles/pack-cards.md` («не q-chip для slot pickers») — только для **answer pool внутри task compose dialog**. List grids и slot **chrome** (`.peek-slot-like`) без регрессии. Delta: MODIFIED Playing-card + SC-PACK-260; apply MUST обновить pack-cards / content-pages.

### D3 — Answer dialog: fields only

- **Choice:** `q-input` content + description; без `PackAnswerCardTile` preview в dialog.
- **Why:** explore решение #2.

### D4 — Remove add-task-set top live grid

- **Choice:** Удалить блок `live-answer-card-grid` + caption `addTaskSetLiveCardsHint` (+ empty-cards stub над compose); live cards остаются источником для chip-пула в dialog и для CSV answer context.
- **Why:** дубль не нужен; SC-PACK-107 add-only сохраняется (карточки по-прежнему не редактируются).

### D5 — Page chrome: add button + CSV + list

- **Choice:** Порядок на Tasks/AddTaskSet: `[+ Добавить вопрос]` → `PackTasksCsvControls` (если не viewOnly) → `task-card-grid`. На Editor answers: CSV frame (как сейчас) → `[+ Добавить карточку]` → `answer-card-grid` (CSV не в dialog).
- **Gates:** add control + dialog open только когда compose allowed — Tasks/AddTaskSet: не `viewOnly` / не `readOnly` lock; Editor answers: не `cardsReadOnly` (в т.ч. `task_set_author`). Иначе add скрыт; Edit на плитке не открывает compose.
- **Why:** CSV «сверху списка»; add control тоже над списком; без disabled inline-формы как сейчас на Editor.

### D6 — i18n / a11y

- **Choice:** Новые/переиспользовать ключи `content.*` для add-question / dialog titles (`addCard` / `addTask` / `editCard` / `editTask` / `save*` / `cancelEdit*` уже есть — dialog title может reuse или короткий `content.*DialogTitle`); persist disabled + tooltip SC-PACK-119 остаётся на Save в dialog.
- **Why:** без прыжка layout на странице (форма больше не на странице).

### D7 — Main-spec / skill supersede (implementability)

- **Choice:** Delta несёт **MODIFIED** полный requirement `Playing-card presentation…` (SC-PACK-222…227): (1) create/edit fields в dialog, над списком только add control; (2) task-dialog answer pool = chips, list tiles без смены; сценарии 223/225/226/227 переносятся без смены смысла (MODIFIED заменяет весь блок). Skills `pack-cards` / `content-pages` / `content` обновлены в align-meta; apply §3.1 только подтверждает.
- **Why:** без полного MODIFIED `openspec validate`/archive отвергает delta (drop sibling scenarios); без carve-out apply конфликтует с main «forms above grid» + «slot picker playing-card».

## Risks / Trade-offs

- [Chip pool хуже показывает description карточки] → Mitigation: label chip = content; description не обязателен для slot pick (как в peek).
- [Дублирование markup Tasks vs AddTaskSet] → Mitigation: D1 shared dialog component.
- [Регрессия тестов на `slot-picker-grid` / inline form] → Mitigation: обновить vitest на dialog testids + SC-PACK-256…262.
- [Skill pack-cards / content.md противоречат chip pool] → Mitigation: D7 + skills уже обновлены в align-meta (carve-out); tasks §3.1 — подтвердить / дотянуть при drift.
- [`.peek-slot-like-row` без center] → Mitigation: D2 — в dialog явно `justify-center` (не менять global row для list tiles).

## Migration Plan

- Только client deploy; server не трогаем.
- Rollback: вернуть inline forms (git revert client).

## Technical prerequisites

- Нет новых npm deps / внешних сервисов.
- Нет новых server endpoints / schema fields / room messages.
- Explore решения закрыты (scope поверхностей, chips, CSV, no preview, Edit = same modal).

## Implementation touchpoints

- Client pages: `ContentPackEditorPage.vue`, `ContentPackTasksPage.vue`, `ContentPackAddTaskSetPage.vue`
- Optional components: `PackTaskComposeDialog.vue`, `PackAnswerComposeDialog.vue` under `src/components/`
- Styles: reuse `.peek-slot-like*`; chip pool styles — mirror GamePage peek answers (shared utility in `app.scss` if needed, without changing GamePage behavior)
- i18n: `src/i18n` / content locale keys
- Skills (meta, обновлены в align-meta; apply §3.1 verify): `.agents/skills/client/work-with-pages/content-pages.md`, `work-with-styles/pack-cards.md`, `work-with-stores/content.md` — compose = dialog + chip-pool carve-out
- Tests: vitest page/component suites for SC-PACK-256…262; регрессия SC-PACK-237 slot chrome inside dialog
- Чеклист apply — `tasks.md`
