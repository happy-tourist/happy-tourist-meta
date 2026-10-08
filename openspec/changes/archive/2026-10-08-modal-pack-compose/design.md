## Context

См. `proposal.md` — Why.

Реализовано (все блоки `tasks.md` §1–8):

- Answer/task compose в `q-dialog`; AddTaskSet без верхнего live-grid; CSV на странице.
- Compose + peek: reuse одной карточки ответа в нескольких слотах без used-state chrome.
- Never-live add-task-set quiet draft persist / restore / discard.
- Persisted author lifecycle: draft activation (hidden empty shell), `hasUnsubmittedChanges` Submit gate, pending→draft on edit, dirty `needs_revision` вне staff-actionable queue, authoritative `canHardDelete` для never-published pack/cycle.

### Server / HTTP

- Compose UI / peek reuse — без room schema.
- Draft lifecycle — server + client: activation, persisted unsubmitted-change signal, pending withdrawal on edit, delete any never-published status.

## Goals / Non-Goals

**Goals:**

- Compose dialogs + chip reuse (уже).
- Add-task-set: quiet persist на любое dirty-изменение; неполный draft ок; minima только на Submit.
- Уход / reload → тот же draft; author neverLive + drafts filter; staff queue без draft.
- После reload Submit отражает persisted unsubmitted work, а не текущую browser-сессию.
- Авторская правка `pending` снимает запрос в `draft`; `needs_revision` сохраняет author-facing статус и thread, получает dirty-признак и до resubmit исключается из staff-actionable queue.
- Empty pack shell скрыт до первого сохранённого изменения; top delete доступен для never-published draft/pending/needs_revision.
- Один never-live цикл на автора на пак.

**Non-Goals:**

- Несколько параллельных never-live drafts одного автора.
- Soft-unpublish published set как «черновик».
- Новые peek room messages / CSV format / approve rules.
- Hard-delete когда-либо опубликованных pack/task-set; author re-edit сохраняет предыдущую live-версию.

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

- **Choice:** Как `ContentPackEditorPage` / Tasks: debounce quiet `saveAddTaskSet` на любое dirty изменение local set. Не ждать Submit. После успешного put обновить `staged`/request ids и server-provided `hasUnsubmittedChanges` в store. Autosave fingerprint остаётся локальным механизмом дедупликации put, но **не** определяет Submit readiness.
- **Why:** explore D3/D4 — «любое изменение».
- Specs: SC-PACK-264/265/271.

### D11 — Top delete never-published work

- **Choice:** На editor/add-task-set сверху кнопка удаления, пока целевой pack/task-set ни разу не был live, независимо от `draft` / `pending` / `needs_revision` (legacy cancelled читается как draft). `GET /api/content/packs/:id/draft` и `GET /api/content/packs/:id/add-task-set` возвращают server-derived `canHardDelete` текущего target; client не выводит право из одного status. Confirm переиспользует существующие client actions/routes (`deleteUnpublishedPack` → `POST /api/content/pack/delete`, `discardAddTaskSetDraft` → `POST …/add-task-set/discard`), расширенные на этот lifecycle: server повторно проверяет право и hard-delete target + его request/messages/revision refs; parent live pack и опубликованные sibling sets не трогать. Для add-task-set target = единственный retained never-live cycle автора; если revision содержит несколько never-live sets этого цикла, discard удаляет их вместе. Для когда-либо опубликованной сущности кнопку не показывать даже при новом author draft.
- **Why:** продуктовый критерий удаления — «ещё не опубликовано», а не текущий moderation status.
- **Alt:** только trash на ghost row — недостаточно (нужна кнопка сверху; ghost trash MAY later, не обязателен).
- Specs: SC-PACK-266/274/275.

### D12 — Draft activation hides untouched pack shell

- **Choice:** `createPack` может создать DB shell и открыть editor, но shell получает persisted `content_packs.draft_activated=false` и не попадает в unified list/drafts filter. Первый author save после семантического изменения относительно сохранённого editor payload (metadata/card/task) атомарно ставит `draft_activated=true`; identical retry marker не меняет. Далее status = `draft` и запись видна только автору.
- **Why:** ID нужен до вложенных edits, но сам переход Create→Editor не является пользовательским черновиком.
- **Alt:** создавать pack только при первой карточке — усложняет routing/child ids; GC на leave — ненадёжно при crash/закрытии вкладки.
- **Migration:** существующие строки созданы до появления marker и не позволяют надёжно отличить untouched shell от meaningful metadata-only draft (`createPack` уже сохраняет непустой title). Поэтому backfill всех существующих packs = `true` (не скрывать пользовательскую работу); `false` ставится только новым shells после deploy. Live/open/non-empty work тем самым также остаётся видимым.
- Specs: SC-PACK-269/270.

### D13 — Persisted unsubmitted-change signal

- **Choice:** Server — source of truth для `hasUnsubmittedChanges`. Pack creator working-copy lifecycle хранит marker на `content_packs`; task-set author cycle (existing own-set re-edit и add-task-set) — на `content_moderation_requests`, чтобы не смешивать независимые author targets одного pack. Первый semantic save task-set автора без request создаёт `status=draft` carrier сразу (аналог D9), а не оставляет marker только в общем `workingRevisionId`. Author save ставит marker `true` только при семантическом изменении; identical retry его не меняет. Successful Submit ставит `false`; staff `needs_revision` оставляет `false` до первой author save; Cancel→draft ставит `true`. GET/list/editor/add-task-set payloads возвращают/выводят marker текущего target. Client `canSubmit` = marker ∧ minima ∧ ACL/lock gates, не session fingerprint.
- **Migration:** новые NOT NULL marker columns получают safe defaults. Existing never-published pack без moderation request получает pack marker `true`. Existing retained task-set requests в author-draft состояниях (`draft`, а legacy `cancelled` — только когда его payload сохраняется как never-live cycle) получают request marker `true`, чтобы уже сохранённый pre-submit draft не стал pristine после deploy. Existing staff-actionable `pending` / `needs_revision` мигрируются clean (`false`) для сохранения очереди. До deploy невозможно надёжно восстановить факт старой несабмиченной правки поверх `pending` / `needs_revision`; это rollout limitation, после deploy все save/submit transitions marker-aware.
- **Why:** `moderationStatus` недостаточен: `needs_revision` должен сохраняться до resubmit, но readiness после reload всё равно должна быть известна. Session baseline теряет смысл при reload.
- **Alt:** сравнивать timestamps — гонки и ложные dirty; сравнивать mutable request revision с собой — невозможно, если author save изменяет ту же revision.
- Specs: MODIFIED SC-PACK-234 + SC-PACK-271/273.

### D14 — Author edit of pending withdraws to draft

- **Choice:** При первом author save поверх `pending` server в одной SQLite transaction записывает payload, переводит request в `draft`, очищает staff take и ставит `hasUnsubmittedChanges=true`; staff action после перехода отклоняется, потому что request больше не open. `needs_revision` при author save сохраняет author-facing status/thread, но transaction очищает staff take, ставит dirty-marker и делает request не staff-actionable. Staff queue и approve принимают `pending|needs_revision` только при `hasUnsubmittedChanges=false`. Resubmit валидирует minima, переиспользует тот же retained `draft`/`needs_revision` request (включая pack request, который текущий `getOpenModerationRequest` не находит в `draft`) и атомарно делает clean `pending`, не создавая параллельный cycle.
- **Why:** авторский `draft` означает «не отправлено на модерацию»; нельзя позволять staff approve изменяемый snapshot параллельно с author changes. Та же защита нужна dirty `needs_revision`, иначе текущий approve-without-resubmit контракт опубликует несабмиченные изменения.
- **Alt:** держать pending snapshot и отдельный draft — отвергнуто как два конкурирующих состояния одного цикла.
- Specs: SC-PACK-272/273.

## Risks / Trade-offs

- [Chip pool хуже показывает description] → Mitigation: label = content (как peek).
- [Неполный draft в БД] → Mitigation: staff queue не видит `draft`; Submit minima без изменений.
- [Legacy cancelled vs draft] → Mitigation: читать оба как author draft; новые cancel → `draft`.
- [Autosave гонки] → Mitigation: debounce + last-write; quiet flag без layout jump (SC-PACK-119).
- [Регрессия put minima tests] → Mitigation: mocha split put-draft vs submit.
- [Staff держит take в момент author save pending] → Mitigation: status transition + take clear атомарны; staff action повторно проверяет open status.
- [Staff держит take в момент author save needs_revision] → Mitigation: dirty transition очищает take; queue/approve проверяют `hasUnsubmittedChanges=false`.
- [Existing empty shells неотличимы от metadata-only drafts] → Mitigation: conservative backfill existing packs как activated; скрывать только новые shells с явным marker.

## Migration Plan

- Deploy server (DB markers + payload/status/discard contract) затем client (server-driven Submit gate + editable pending/delete chrome).
- Backfill draft activation / dirty markers до включения новых list filters; existing packs активировать консервативно, retained task-set `draft` / legacy retained `cancelled` считать dirty, а existing staff-actionable `pending` / `needs_revision` оставить clean как rollout baseline.
- Rollback: client может игнорировать новые поля; server defaults сохраняют текущие status payloads, но schema columns не удалять.

## Technical prerequisites

- Нет новых npm deps / внешних сервисов.
- Нет новых Colyseus room messages.
- Explore decisions закрыты: author draft = не отправлено; pending edit withdraws; needs_revision сохраняет status; delete до первой публикации; meaningful save activates draft.

## Implementation touchpoints

- Server: `db/schema.ts` + content migration; `lib/content.ts` create/list/save/submit/needs-revision/cancel/delete and add-task-set cycle; thin existing routes; mocha `zz-contentPacks`
- Client: pack editor + add-task-set/task-set pages; `stores/content.ts` payload types/status transitions; `lib/editorDirty.ts` только для autosave/session dedupe где применимо; vitest
- Skills: client content pages/store/test + server routes/structure/database/test — lifecycle marker/status/delete canon
- Чеклист — `tasks.md` §1–8 все `[x]`
