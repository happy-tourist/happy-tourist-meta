## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-222 | modified (compose forms → dialog; list tiles unchanged) |
| SC-PACK-223 | unchanged (carried in MODIFIED — task list tiles) |
| SC-PACK-224 | modified (task-dialog answer pool = chips, not playing-card picker) |
| SC-PACK-225 | unchanged (carried in MODIFIED — answer tile sizes) |
| SC-PACK-226 | unchanged (carried in MODIFIED — task tile size) |
| SC-PACK-227 | unchanged (carried in MODIFIED — dark theme contrast) |
| SC-PACK-256 | covered (`ContentPackAnswerCompose`) |
| SC-PACK-257 | covered (`ContentPackAnswerCompose`) |
| SC-PACK-258 | covered (`ContentPackTaskCompose`) |
| SC-PACK-259 | covered (`ContentPackTaskCompose`) |
| SC-PACK-260 | covered (`ContentPackTaskCompose` + `ContentPackTasksCsvHide`) |
| SC-PACK-261 | covered (`ContentPackTaskCompose` + `ContentFollowUp4`) |
| SC-PACK-262 | covered (`ContentPackTaskCompose`) |
| SC-PACK-263 | covered (`ContentPackTaskCompose`) |
| SC-PACK-264 | covered (server mocha + client vitest — pre-submit incomplete draft persist) |
| SC-PACK-265 | covered (server mocha + client vitest — leave/reload restores draft) |
| SC-PACK-266 | covered (server mocha + client vitest — delete draft from AddTaskSet top) |
| SC-PACK-267 | covered (server mocha — Cancel never-live → draft again) |
| SC-PACK-268 | covered (server mocha + client vitest — author-only drafts filter / neverLive draft) |
| SC-PACK-269 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDraftLifecycle.test.ts` — empty pack shell hidden) |
| SC-PACK-270 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDraftLifecycle.test.ts` / `content.draftLifecycle.test.ts` — first meaningful edit activates draft) |
| SC-PACK-271 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDirtySubmit.test.ts` / `ContentPackAddTaskSetDraft.test.ts` — Submit readiness survives reload) |
| SC-PACK-272 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDraftLifecycle.test.ts` / `ContentPackAddTaskSetDraft.test.ts` — author edit withdraws pending to draft) |
| SC-PACK-273 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDirtySubmit.test.ts` / `ContentPackAddTaskSetDraft.test.ts` — needs_revision dirty survives reload) |
| SC-PACK-274 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDraftLifecycle.test.ts` / `ContentPackAddTaskSetDraft.test.ts` — delete never-published pack/task-set from moderation) |
| SC-PACK-275 | covered (`{server}/test/zz-contentPacks.test.ts` + `{client}/src/pages/__tests__/ContentPackDraftLifecycle.test.ts` / `ContentPackAddTaskSetDraft.test.ts` — published entity remains non-deletable) |

Related: compose/slots SC-PACK-05…07 / 119; add-task-set SC-PACK-107/108; cancel→draft SC-PACK-175…179/196/197; neverLive SC-PACK-188…190; playing-card tiles SC-PACK-222…227; peek-slot chrome SC-PACK-237/238; CSV SC-PACK-213…220 / 235/236; main `content/packs`. Peek gameplay reuse — `game/board` SC-BOARD-49 (same change).

**Server / HTTP (revision):** compose UI (SC-PACK-256…263) remains client-only. Draft lifecycle (SC-PACK-264…275) changes pack/add-task-set persist, list/status, moderation transitions and discard on server. Staff-actionable queue contains clean submitted `pending` / `needs_revision` requests only (`hasUnsubmittedChanges=false`); an author edit of `pending` first withdraws that request to `draft`, while a dirty `needs_revision` keeps its author-facing status but leaves the actionable queue until resubmit.

## MODIFIED Requirements

### Requirement: Author Submit disabled until content is dirty

On pack creator / task-set-author editing surfaces that expose «На модерацию» (cards editor, task-set editor, add-task-set), Submit MUST represent **persisted unsubmitted author changes**, not merely a difference created during the current browser session. A successful author save that changes editable content MUST persist an unsubmitted-change signal; reopening or reloading the editor MUST preserve that signal. Submit MUST be enabled when that signal is present and minima plus existing lock rules allow, and MUST be disabled when no unsubmitted author changes exist. Successful Submit MUST clear the signal for the submitted revision.

If the author changes content while its request is `pending`, the system MUST withdraw that request from the staff-open queue and expose the work as author `draft` with unsubmitted changes; the author MUST explicitly Submit again. If staff has marked the request `needs_revision`, that author-facing status and thread MUST remain available while the author edits, but the persisted unsubmitted-change signal MUST become true so Submit remains enabled after reload. A dirty `needs_revision` request MUST NOT remain staff-actionable or be approvable until the author successfully resubmits it; resubmit MUST set it to clean `pending`. This supersedes the generic rule that an unchanged needs-revision request may be approved without resubmit: only `hasUnsubmittedChanges=false` remains eligible for that behavior. Staff direct-edit surfaces without Submit are unaffected.

The server MUST persist the unsubmitted-change signal independently for each current author edit target, including pack working copies, re-edits of an author’s published task set, and add-task-set cycles. Editor and add-task-set responses MUST return `hasUnsubmittedChanges` for that target; list responses that expose author status MUST derive status/filter behavior from the same server state. The first semantic task-set save MUST establish or reuse an author `draft` cycle. Save MUST set the signal only after a semantic content mutation is persisted, not for an identical retry. Submit MUST clear it atomically with the request transition and MUST reuse the target’s retained cycle rather than create a parallel one. Migration MUST preserve retained pre-submit task-set drafts (including legacy cancelled rows that retain never-live payload) as dirty, while existing staff-actionable `pending` / `needs_revision` requests MUST start clean because pre-deploy dirty state cannot be reconstructed reliably.

#### Scenario [SC-PACK-234]: Submit disabled on open without edits

- **GIVEN** a verified author opens pack Edit (or add-task-set / task-set editor) with content that already meets submit minima
- **AND** the persisted content has no unsubmitted author changes
- **WHEN** the client renders «На модерацию»
- **THEN** the control is disabled
- **AND GIVEN** the author changes and saves at least one editable field in the working copy
- **WHEN** minima and lock rules still allow submit
- **THEN** «На модерацию» is enabled

#### Scenario [SC-PACK-271]: Draft Submit readiness survives reload

- **GIVEN** author U has a persisted draft with unsubmitted changes that meets submit minima
- **WHEN** U reloads or later reopens the editor
- **THEN** the content and its unsubmitted-change signal are restored
- **AND** «На модерацию» is enabled without requiring another edit in that browser session

#### Scenario [SC-PACK-272]: Editing pending work withdraws it to draft

- **GIVEN** author U has a `pending` pack or never-live task-set request
- **WHEN** U changes and saves editable content
- **THEN** the request is removed from the staff-open queue
- **AND** U sees the affected work as `draft`
- **AND** its unsubmitted-change signal is true
- **AND** U MUST Submit again before staff can moderate the changed content

#### Scenario [SC-PACK-273]: Needs-revision changes remain ready after reload

- **GIVEN** staff returned author U’s request with status `needs_revision`
- **WHEN** U changes and saves editable content and then reloads the editor
- **THEN** the author-facing status remains `needs_revision`
- **AND** the persisted unsubmitted-change signal remains true
- **AND** the request is absent from the staff-actionable queue and staff approve is rejected while that signal is true
- **AND** «На модерацию» is enabled when submit minima and lock rules allow
- **AND WHEN** U successfully submits again
- **THEN** the request becomes clean `pending` and returns to the staff-actionable queue

### Requirement: Playing-card presentation for answer cards and tasks

Wherever the client shows a **list** of pack **answer cards** or **tasks** (cards editor grid, task list grid, live pack/tasks view, staff moderation views), each item MUST be rendered as a rounded playing-card tile in a wrapping row. An answer card tile MUST use a **vertical** splitter: content on one side and description on the other when description is present; when description is empty the tile MUST be **150×200** without an empty description half; when description is present the tile MUST be **300×200**. A task tile MUST be **300×200** with a **vertical** splitter between question and slots; filled slot answers MUST appear **side by side** as filled slot chips (not a vertical stack of lines); task difficulty MUST appear at the **top-left**. Content/question/slot type MUST be roughly twice the previous body size so tiles remain readable at these widths. Tile background and foreground MUST remain readable in both light and dark themes. On surfaces where the item is editable, Edit and Delete MUST appear as **full-width text buttons stacked at the bottom** of the tile (not icon buttons in a corner). Changing content MUST be available through Edit, not by treating a body click as edit. Overflow text MUST scroll inside its half rather than unbounded grow of the row.

**Compose shell (this change):** create/edit of answer cards and tasks MUST happen in a dialog (see ADDED requirements SC-PACK-256…260), not as an always-visible inline form above the list. When compose is allowed, the surface MUST expose an **add control** above the corresponding list (not the full form). When compose is not allowed (read-only / live view-only / cards locked for `task_set_author`), the add control MUST be hidden and Edit on tiles MUST NOT open a compose dialog.

**Task compose answer pool carve-out:** inside the task compose dialog, the selectable answer pool MUST be compact chips comparable to the game-board peek answer pool and MUST NOT be a playing-card tile grid. List surfaces and any non-dialog answer/task grids keep playing-card chrome unchanged.

#### Scenario [SC-PACK-222]: Answer cards render as vertically split playing-card tiles

- **GIVEN** a surface that lists answer cards including one with description and one without
- **WHEN** the user views the list
- **THEN** each card is a rounded tile in a wrapping row
- **AND** the card with description shows content and description side by side with a vertical splitter
- **AND** the card without description is a 150×200 rectangle without an empty description half
- **AND** long description text scrolls inside its half
- **AND** create/edit fields for those cards are not an always-visible inline form above the list (dialog compose per SC-PACK-256/257)

#### Scenario [SC-PACK-223]: Tasks render as vertically split tiles with horizontal slot chips

- **GIVEN** a surface that lists tasks with difficulty and filled slots
- **WHEN** the user views the list
- **THEN** each task is a rounded 300×200 tile with the question and slots on either side of a vertical splitter
- **AND** filled slot answers appear side by side as chips
- **AND** difficulty is shown at the top-left of the tile

#### Scenario [SC-PACK-224]: Editable tiles expose bottom full-width text Edit and Delete

- **GIVEN** an editable cards or tasks editor and a read-only live or staff view of the same kind of items
- **WHEN** the user compares those surfaces
- **THEN** editable tiles show Edit and Delete as stacked full-width text buttons at the bottom
- **AND** live and staff views use the same playing-card chrome without Edit/Delete when not editable
- **AND** answer/task **list** grids keep playing-card chrome
- **AND** the task compose dialog answer pool uses compact chips (not playing-card tiles) per SC-PACK-260

#### Scenario [SC-PACK-225]: Answer tiles use fixed 150×200 or 300×200 sizes

- **GIVEN** a surface listing an answer card without description and an answer card with description
- **WHEN** the user views those tiles
- **THEN** the card without description is 150 CSS pixels wide and 200 CSS pixels tall
- **AND** the card with description is 300 CSS pixels wide and 200 CSS pixels tall with a vertical splitter between content and description

#### Scenario [SC-PACK-226]: Task tiles are 300×200

- **GIVEN** a surface listing tasks
- **WHEN** the user views a task tile
- **THEN** the tile is 300 CSS pixels wide and 200 CSS pixels tall

#### Scenario [SC-PACK-227]: Answer and task tiles stay readable in dark theme

- **GIVEN** the application is in dark theme
- **WHEN** the user views answer and task playing-card tiles including editable Edit controls
- **THEN** tile text and edit controls remain visible against the tile background
- **AND** the tile MUST NOT present light text on an unresolved white card background

## ADDED Requirements

### Requirement: Empty pack shell stays hidden until meaningful author work

Creating a pack MAY persist a technical shell needed to open its editor, but that untouched shell MUST NOT appear in the unified packs list or drafts filter for its creator. The first saved author content mutation after opening the editor — including changing pack metadata, adding an answer card, or adding a task — MUST activate an author-only `draft`. An identical save/retry MUST NOT activate the shell. That draft MUST then appear to its creator in the unified list and drafts filter and MUST remain hidden from other non-staff users. Leaving the untouched editor MUST NOT create visible author work. Existing pre-marker never-published packs MUST be conservatively migrated as activated so rollout cannot hide possibly meaningful historical work; the hidden-shell guarantee applies to packs created with the new marker.

#### Scenario [SC-PACK-269]: Untouched new pack does not appear in author catalog

- **GIVEN** verified author U creates a pack shell and opens its editor
- **WHEN** U makes no editor change and leaves
- **THEN** the shell does not appear in U’s unified packs list or drafts filter
- **AND** it does not appear to other non-staff users

#### Scenario [SC-PACK-270]: First meaningful edit activates author draft

- **GIVEN** verified author U has opened an untouched new pack shell
- **WHEN** U saves a metadata change, adds an answer card, or adds a task
- **THEN** the work is quietly persisted with author-facing status `draft`
- **AND** it appears only to U in the unified packs list and drafts filter
- **AND** leaving and reopening restores that draft

### Requirement: Answer card compose uses a dialog

On the pack cards editing surface, creating or editing an answer card MUST happen in a dialog, not in an always-visible inline form above the answer-card list. When compose is allowed, the surface MUST expose an add control above the answer-card list that opens an empty create dialog; when compose is not allowed (`cardsReadOnly` / locked), the add control MUST be hidden. Activating Edit on an answer-card tile MUST open the same dialog prefilled for that card only when compose is allowed. The dialog MUST present the answer content and optional description fields only and MUST NOT show a playing-card preview of the answer. Cancel or successful save MUST close the dialog; persistence and cascade rules for answer cards remain unchanged.

#### Scenario [SC-PACK-256]: Add answer opens dialog above the list

- **GIVEN** a verified editor on the pack cards editing surface with compose allowed
- **WHEN** the editor activates the add-answer control above the answer-card list
- **THEN** a dialog opens for creating an answer card
- **AND** the page does not show a persistent inline answer create/edit form above the list

#### Scenario [SC-PACK-257]: Edit answer opens the same dialog without preview

- **GIVEN** a verified editor viewing editable answer-card tiles on the pack cards editing surface
- **WHEN** the editor activates Edit on an answer-card tile
- **THEN** the same answer dialog opens prefilled with that card’s content and description
- **AND** the dialog does not show a playing-card preview of the answer
- **AND** saving or cancelling closes the dialog without leaving an inline form on the page

### Requirement: Task compose uses a peek-like dialog

On task-set editing surfaces (creator/staff task-set editor and post-publish add-task-set), creating or editing a task MUST happen in a dialog, not in an always-visible inline compose form above the task list. When compose is allowed, the surface MUST expose an add-question control above the task list that opens an empty create dialog; when compose is not allowed (`viewOnly` / `readOnly`), the add control MUST be hidden. Activating Edit on a task tile MUST open the same dialog prefilled for that task only when compose is allowed. Inside the dialog the client MUST present editable question and difficulty, then a **centered** ordered slot row (peek-comparable slot chrome, row centered like the game-board peek slot row), then below it a pool of answer choices rendered as compact chips comparable to the shared peek answer pool on the game board (not as a playing-card tile grid). Slot fill/clear/add/remove behavior and draft persistence rules remain as today. Cancel or successful save MUST close the dialog.

#### Scenario [SC-PACK-258]: Add question opens dialog above the task list

- **GIVEN** a verified editor on a task-set editing surface with compose allowed
- **WHEN** the editor activates the add-question control above the task list
- **THEN** a dialog opens for creating a task
- **AND** the page does not show a persistent inline task create/edit form above the list

#### Scenario [SC-PACK-259]: Edit task opens the same dialog

- **GIVEN** a verified editor viewing editable task tiles on a task-set editing surface
- **WHEN** the editor activates Edit on a task tile
- **THEN** the same task dialog opens prefilled with that task’s question, difficulty, and slots
- **AND** saving or cancelling closes the dialog without leaving an inline form on the page

#### Scenario [SC-PACK-260]: Task dialog matches peek slot and answer-pool layout

- **GIVEN** a verified editor has the task compose dialog open with available answer cards
- **WHEN** the dialog is shown
- **THEN** question and difficulty controls appear above a centered ordered slot row
- **AND** below the slots the answer choices appear as compact chips comparable to the game peek answer pool
- **AND** the dialog MUST NOT use a playing-card tile grid as the answer picker

### Requirement: Add-task-set does not duplicate live answer cards above compose

On the post-publish add-task-set surface, the client MUST NOT show a separate read-only grid of live answer cards above the task list or above the add-question control. Live answer cards remain available only as the answer pool inside the task compose dialog (and as the answer context for task CSV). Existing add-only rules for non-staff (no editing of live cards or existing sets) remain unchanged.

#### Scenario [SC-PACK-261]: Add-task-set has no top live answer grid

- **GIVEN** a verified non-staff user opens add-task-set for a published pack that has live answer cards
- **WHEN** the page renders
- **THEN** there is no separate read-only live answer-card grid above the task list
- **AND** the user can still fill task slots from live cards inside the task compose dialog

### Requirement: Task CSV stays above the task list outside the dialog

On task-set editing surfaces that expose task CSV import/export, those controls MUST remain on the page above the task list (subject to existing live view-only hide rules) and MUST NOT move into the task compose dialog. The add-question control MAY appear above the CSV frame; both MUST stay outside the dialog and above the task list when compose and CSV are allowed.

#### Scenario [SC-PACK-262]: CSV remains above the task list

- **GIVEN** a verified editor on a task-set editing surface where task CSV is allowed
- **WHEN** the page renders with the task list
- **THEN** the framed task CSV controls appear above the task list
- **AND** opening the task compose dialog does not relocate those CSV controls into the dialog

### Requirement: Task compose allows the same answer card in multiple slots

On task compose surfaces (task compose dialog on task-set editor and add-task-set), filling answer slots MUST allow the same answer card to be selected into any number of slots, including every slot of the task. After a card is placed in one slot, its chip in the answer pool MUST remain selectable for other empty slots. The client MUST NOT disable that chip solely because it is already used in a slot, and MUST NOT present a distinct «already used» highlight or filled-vs-outline state that marks the card as consumed. Clicking a filled slot MAY still clear that slot. Persistence and cascade rules for slot `answerCardId` references remain unchanged (multiple slots MAY reference the same card id).

#### Scenario [SC-PACK-263]: Same answer card fills multiple compose slots without used chrome

- **GIVEN** a verified editor has the task compose dialog open with at least one answer card and two or more empty slots
- **WHEN** the editor places that same answer card into more than one slot
- **THEN** each of those slots shows that card as its answer
- **AND** the answer-pool chip for that card remains available for further empty slots
- **AND** the chip is not shown in a distinct already-used disabled or highlighted consumed state solely because it appears in a slot

### Requirement: Add-task-set persists an author draft on any change before moderation

On the post-publish add-task-set surface, any author edit that changes the in-progress task set (add/remove/edit a task or slot binding, including incomplete content) MUST quietly persist a **never-live** working draft for that author. Persist MUST succeed even when submit minima are not met (fewer than two tasks and/or empty slots). Submit for moderation MUST continue to enforce existing minima and filled-slot rules. Leaving the page without Submit MUST NOT discard that draft. Reloading add-task-set or returning later MUST restore the retained tasks and slots. The draft MUST appear to the set author as author-facing **draft** with the same never-live ghost visibility rules as after Cancel (live pack set list + unified packs drafts filter for that author only). Other users and staff on live MUST NOT see that never-live draft row. Staff moderation queue MUST NOT list a not-yet-submitted draft. At most one never-live add-task-set cycle per author per pack MUST be retained (same exclusivity as today).

#### Scenario [SC-PACK-264]: Incomplete add-task-set change persists as draft

- **GIVEN** a verified author on add-task-set for in-catalog pack P with no open pending/needs_revision task_set request of their own
- **WHEN** the author adds or changes at least one task (even with empty slots or fewer than two tasks)
- **THEN** the system persists that content as the author’s never-live draft
- **AND** author-facing status for that work is draft
- **AND** Submit remains disabled until existing minima and filled slots are met

#### Scenario [SC-PACK-265]: Leave without Submit restores the same draft

- **GIVEN** author U has a persisted never-live add-task-set draft on pack P that was never submitted (or was Cancelled back to draft)
- **WHEN** U leaves the add-task-set page and later opens add-task-set Edit for P again
- **THEN** the draft includes the same task questions and slot references that were last persisted
- **AND** the draft MUST NOT be an empty task-set list solely because no open pending request exists

#### Scenario [SC-PACK-268]: Author-only drafts filter and never-live draft row

- **GIVEN** author U has a never-live add-task-set draft on in-catalog pack P and viewer V ≠ U
- **WHEN** U opens the unified packs list with the drafts filter and the live pack task-set list
- **THEN** U can reach that draft (list draft mark and/or never-live set row with draft status)
- **AND WHEN** V opens the same list/live surfaces
- **THEN** V MUST NOT see U’s never-live draft row or treat P as draft solely because of U’s work

### Requirement: Author may delete never-published work from its edit surface

While an author-owned pack or task set has never been published, its edit surface MUST expose a delete control at the top regardless of whether its moderation status is `draft`, `pending`, or `needs_revision`. The pack editor (`GET /api/content/packs/:id/draft`) and add-task-set (`GET /api/content/packs/:id/add-task-set`) payloads MUST return authoritative `canHardDelete` for the current target, and the existing pack-delete / add-task-set-discard endpoints MUST enforce the same condition rather than trusting client status. Confirming delete MUST remove that never-published entity, its moderation request/messages and retained revision payload so it disappears from the author list/ghost rows and from the staff queue. For add-task-set, the target is the current author’s single retained never-live cycle; if its retained payload contains multiple never-live sets, delete removes that cycle’s never-live sets together. Deleting a never-live task-set cycle MUST NOT remove its published parent pack or any published sibling task sets. Once the target pack or task set has been published at least once, hard delete MUST NOT be available during later draft/pending/needs_revision edits; the last approved live version remains until a later approval replaces it.

#### Scenario [SC-PACK-266]: Top delete removes never-live draft

- **GIVEN** author U has a never-live add-task-set draft on pack P and is on the add-task-set page
- **WHEN** U activates the top delete-draft control and confirms
- **THEN** the never-live draft is removed
- **AND** U no longer sees that draft ghost/row
- **AND** opening add-task-set again starts from an empty set (no restored prior draft)

#### Scenario [SC-PACK-274]: Author deletes never-published work from moderation

- **GIVEN** author U has either a never-published pack or a never-live task-set cycle with status `pending` or `needs_revision`
- **AND** the editor payload reports `canHardDelete=true`
- **WHEN** U confirms delete from the top of its edit surface
- **THEN** the target entity/cycle, retained payload and moderation request are removed
- **AND** moderation messages belonging to that request are removed
- **AND** the request no longer appears in the staff queue
- **AND IF** the target is a task-set cycle on a published parent pack
- **THEN** that parent pack and its existing published sibling task sets are unchanged

#### Scenario [SC-PACK-275]: Previously published work cannot be hard-deleted

- **GIVEN** a pack or task set has been published at least once and now has author draft, pending, or needs-revision edits
- **WHEN** the author opens its edit surface
- **THEN** hard delete of that pack or task set is unavailable
- **AND** the server reports `canHardDelete=false` and rejects a direct hard-delete attempt
- **AND** other users continue to see the last approved live version until new changes are approved

### Requirement: Cancelling never-live moderation returns the work to draft

When staff or the set author cancels an open never-live add-task-set moderation request that has not been approved into live, the retained payload MUST remain available as author-facing **draft** (same restore/ghost rules as SC-PACK-196/197 and the pre-submit draft requirement). The author MUST be able to amend, re-submit, or delete that draft.

#### Scenario [SC-PACK-267]: Cancel never-live open request yields draft again

- **GIVEN** author U submitted a never-live add-task-set on pack P and the request is pending or needs_revision
- **WHEN** U or staff (after take) cancels that request
- **THEN** U sees the work as draft (never-live ghost / drafts filter)
- **AND** GET add-task-set restores the retained tasks and slots
- **AND** the staff open queue no longer lists that request
