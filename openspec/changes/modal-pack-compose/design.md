## Context

См. `proposal.md` — Why.

Уже в runtime (блоки 1–4):

- Answer/task compose в `q-dialog`; AddTaskSet без верхнего live-grid; CSV на странице.
- Compose + peek: reuse одной карточки ответа в нескольких слотах без used-state chrome.

Gap (revision): add-task-set правки живут в Vue local до Submit; `saveAddTaskSet` / `putAddTaskSet` есть, но страница не делает quiet autosave; GET восстанавливает только open или **cancelled** cycle; ghost — только pending|needs_revision|cancelled. `putAddTaskSet` гоняет submit-minima (≥2 tasks, filled slots) — мешает «любому изменению».

### Server / HTTP

- Compose UI / peek reuse — без room schema.
- **Pre-submit draft** — server + client: put/get/ghost/discard + quiet autosave + top delete.

## Goals / Non-Goals

**Goals:**

- Compose dialogs + chip reuse (уже).
- Add-task-set: quiet persist на любое dirty-изменение; неполный draft ок; minima только на Submit.
- Уход / reload → тот же draft; author neverLive + drafts filter; staff queue без draft.
- Top delete draft + confirm; Cancel never-live → снова draft.
- Один never-live цикл на автора на пак.

**Non-Goals:**

- Несколько параллельных never-live drafts одного автора.
- Soft-unpublish published set как «черновик».
- Новые peek room messages / CSV format / approve rules.

## Decisions

### D1 — Две модалки compose на страницах (или thin components)

- **Choice:** Compose в `q-dialog`; Tasks/AddTaskSet — shared `PackTaskComposeDialog`; Editor answers — локальный dialog.
- **Why:** одна форма на create/edit; меньше вертикального шума.
- **Alt:** `q-expansion` — отвергнуто.

### D2 — Task dialog layout ≈ peek

- **Choice:** question + difficulty → **centered** slot row → chip pool. Не `PackAnswerCardTile` / не `slot-picker-grid`.
- **Reuse:** chip click → fill next empty slot; без used-state — D8.
- **Canon carve-out:** dialog pool = chips (MODIFIED SC-PACK-222/224).

### D3 — Answer dialog: fields only

- **Choice:** `q-input` content + description; без preview плитки.

### D4 — Remove add-task-set top live grid

- **Choice:** Удалить `live-answer-card-grid` + hint / empty stub.

### D5 — Page chrome: add button + CSV + list

- **Choice:** Tasks/AddTaskSet: add-question → CSV → grid; Editor: CSV → add-card → grid.

### D6 — i18n / a11y

- **Choice:** reuse `content.*` / `game.*`; persist disabled + tooltip SC-PACK-119 на Save в dialog.

### D7 — Main-spec / skill supersede (compose shell)

- **Choice:** MODIFIED Playing-card + ADDED 256…262; skills dialog compose + chip-pool carve-out.

### D8 — Reusable answer cards in slots (compose + peek)

- **Choice:** Убрать `isCardPlaced` / `isAnswerPlaced` uniqueness chrome. Specs: SC-PACK-263 + SC-BOARD-49.

### D9 — Pre-submit never-live draft carrier (request status `draft`)

- **Choice:** Первый dirty `putAddTaskSet` без open pending/needs_revision: записать task_set-only revision и создать/обновить moderation request `type=task_set`, `status=draft` (author-facing draft; **не** staff-open). GET / neverLive ghosts / list drafts filter включают `draft` наравне с cancelled→draft mapping. Submit: `draft` → `pending` (reuse revision). Cancel open never-live: → `status=draft` (сохранить payload; читать legacy `cancelled` как draft для совместимости). Один цикл на автора на пак — как сейчас.
- **Why:** тот же author draft vocabulary «с первого изменения», без попадания в очередь; ближе к продуктовому «статус черновика», чем orphan `stagedRevisionId` без request.
- **Alt:** orphan stagedRevisionId only — отвергнуто (GET/ghost дырявые). Сразу `cancelled` на первом put — отвергнуто (ложная семантика cancel).
- **Put validation:** для draft put — **не** вызывать submit-minima (`validateOneTaskSet` / filled slots); достаточно ≥1 task set и валидных id ссылок где слот заполнен (пустые слоты ок). Submit сохраняет текущие minima.
- Specs: SC-PACK-264/265/267/268.

### D10 — Quiet autosave on AddTaskSet (client)

- **Choice:** Как `ContentPackEditorPage` / Tasks: debounce quiet `saveAddTaskSet` на любое dirty изменение local set. Не ждать Submit. После успешного put обновить `staged`/request ids в store; baseline dirty для Submit — отдельно (SC-PACK-234: Submit disabled пока pristine относительно loaded session; после restore draft Submit enabled только после новых правок **или** если baseline = restored draft и уже dirty vs empty — сохранить текущий fingerprint-after-load: после load draft Submit disabled until further edit, minima tooltip как сейчас).
- **Why:** explore D3/D4 — «любое изменение».
- Specs: SC-PACK-264/265.

### D11 — Top delete draft

- **Choice:** На `ContentPackAddTaskSetPage` сверху (над add/CSV/list) кнопка удаления черновика, видима когда есть retained never-live draft (`draft` / legacy cancelled / после load непустой draft без pending). Confirm → server discard (новый DELETE или POST discard add-task-set): удалить draft/cancelled never-live request + orphan revision при отсутствии refs; live pack/sets не трогать. Client: очистить local + ghost.
- **Why:** explore D2 — «сверху».
- **Alt:** только trash на ghost row — недостаточно (нужна кнопка сверху; ghost trash MAY later, не обязателен).
- Specs: SC-PACK-266.

## Risks / Trade-offs

- [Chip pool хуже показывает description] → Mitigation: label = content (как peek).
- [Неполный draft в БД] → Mitigation: staff queue не видит `draft`; Submit minima без изменений.
- [Legacy cancelled vs draft] → Mitigation: читать оба как author draft; новые cancel → `draft`.
- [Autosave гонки] → Mitigation: debounce + last-write; quiet flag без layout jump (SC-PACK-119).
- [Регрессия put minima tests] → Mitigation: mocha split put-draft vs submit.

## Migration Plan

- Deploy server (draft status + relax put + discard) затем client (autosave + delete).
- Rollback: revert server/client; legacy cancelled drafts остаются читаемыми.

## Technical prerequisites

- Нет новых npm deps / внешних сервисов.
- Нет новых Colyseus room messages.
- Explore D1–D4 закрыты: draft status earlier; top delete; any change; incomplete put ok.

## Implementation touchpoints

- Server: `lib/content.ts` (`putAddTaskSet`, `getAddTaskSet`, `submitAddTaskSet`, cancel, `retainedNeverLiveAuthorTaskSetCycle` / ghosts, list status); `app.config.ts` discard route; mocha zz-contentPacks
- Client: `ContentPackAddTaskSetPage.vue` (autosave + top delete); `stores/content.ts`; list/live neverLive already; vitest
- Skills: client content-pages / content store; server routes/structure/test — never-live draft pre-submit + discard
- Чеклист — `tasks.md` §5 (новые); §1–4 уже `[x]`
