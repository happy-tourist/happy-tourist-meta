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

Related: compose/slots SC-PACK-05…07 / 119; add-task-set SC-PACK-107; playing-card tiles SC-PACK-222…227 (list chrome; dialog pool carve-out below); peek-slot chrome SC-PACK-237/238; CSV SC-PACK-213…220 / 235/236; main `content/packs`. Peek gameplay remains `game/board`.

**Server / HTTP:** this delta is client UX only. Persist, cascade, ACL, add-task-set, and CSV product rules stay in main `content/packs` (no new route/payload/schema claims here).

## MODIFIED Requirements

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
