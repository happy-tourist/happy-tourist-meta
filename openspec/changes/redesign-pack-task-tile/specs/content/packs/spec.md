## Traceability

| SC-ID | Status |
|-------|--------|
| SC-PACK-223 | modified (task tile portrait + header + filled slots; **keep historic scenario title**) |
| SC-PACK-224 | modified (task Edit/Delete → outline+icon like task-set; **keep historic scenario title**) |
| SC-PACK-226 | modified (portrait W&lt;H, not 300×200; **keep historic scenario title**) |
| SC-PACK-127 | modified (filled only; empty omitted; description secondary MUST when present) |
| SC-PACK-238 | modified (carve-out lifted — filled peek-like mini-cards on tile) |
| SC-PACK-277 | modified (carve-out text removed for list tiles) |
| SC-PACK-276 | modified (compose dots ≈ badge height) |
| SC-PACK-260 | modified (compose width + Save chrome; no Cancel) |
| SC-PACK-278 | added (task tile: 24px question, centered raised slots, scroll) |
| SC-PACK-279 | added (compose dialog width = 4 mini-cards + pad + ≈50px slack; **keep historic title sense**) |
| SC-PACK-280 | added (compose difficulty dots hit-target ≈ badge height) |
| SC-PACK-281 | added (compose Save = peekSubmit chrome; Cancel omitted; dismiss backdrop/Esc) |

Related: game peek chrome stays as shipped (`game/board`) except list tile **reuses** peek question 24px and compose Save **reuses** peekSubmit chrome; answer playing-cards SC-PACK-222/225 unchanged; compose slot/pool parity SC-PACK-237/263 unchanged. **Apply scope:** client list task tile + task compose dialog only — no server/HTTP/schema.

## MODIFIED Requirements

### Requirement: Playing-card presentation for answer cards and tasks

Wherever the client shows a **list** of pack **answer cards** or **tasks** (cards editor grid, task list grid, live pack/tasks view, staff moderation views), each item MUST be rendered as a rounded tile in a wrapping row. An answer card tile MUST use a **vertical** splitter: content on one side and description on the other when description is present; when description is empty the tile MUST be **150×200** without an empty description half; when description is present the tile MUST be **300×200**. A **task** tile MUST use **portrait** proportions (width strictly less than height), resting surface chrome in the same product family as pack task-set summary cards (shared pack-card surface tokens), and MUST NOT use the former fixed **300×200** wide playing-card with a vertical question|slots splitter. Task difficulty MUST appear as **three difficulty dots at the top-left** plus a **word-only** difficulty badge **horizontally centered** in the tile header (same product sense as the game peek / task compose header — not a digit-only outline badge). The question MUST appear below the header at approximately the **same font-size as the game peek question** (24 CSS px). **Filled** answer slots MUST appear as portrait mini-cards comparable to peek/compose filled slots (answer **content** primary and optional answer **description** secondary when present); empty slots MUST NOT be rendered on the list tile. Filled slots MUST be **horizontally centered** in the tile and MUST NOT sit flush against the actions divider — there MUST be visible air between the slots row and the actions section (or the tile bottom when not editable). When more filled slots than fit in one row, the slot row MUST scroll **horizontally**. Tile background and foreground MUST remain readable in both light and dark themes. On surfaces where a **task** item is editable, Edit and Delete MUST appear as **full-width stacked outline buttons with icons** at the bottom of the tile (same product chrome as pack task-set summary card actions — not flat text-only and not corner icon-only). Delete MAY use a negative/red tint. Answer-card editable tiles keep their existing action chrome. Changing content MUST be available through Edit, not by treating a body click as edit. Overflow question text MUST scroll inside the question region rather than unbounded grow of the row.

**Compose shell:** create/edit of answer cards and tasks MUST happen in a dialog (see SC-PACK-256…260), not as an always-visible inline form above the list. When compose is allowed, the surface MUST expose an **add control** above the corresponding list (not the full form). When compose is not allowed (read-only / live view-only / cards locked for `task_set_author`), the add control MUST be hidden and Edit on tiles MUST NOT open a compose dialog.

**Task compose answer pool carve-out:** inside the task compose dialog, the selectable answer pool MUST be mini-cards comparable to the game-board peek answer pool and MUST NOT be a playing-card tile grid. Answer-card **list** grids keep playing-card chrome unchanged.

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
- **THEN** each task is a rounded portrait tile (width less than height) with pack-card family surface chrome (not a 300×200 vertical question|slots splitter)
- **AND** difficulty dots sit at the top-left and a word-only difficulty badge is horizontally centered in the header
- **AND** the question appears below the header at approximately 24 CSS pixels (peek question parity)
- **AND** filled slots appear as portrait answer mini-cards in a horizontally centered row (scroll when needed)
- **AND** there is visible air between the slots row and the actions divider (or tile bottom when not editable)
- **AND** empty slots are not shown on the tile
- **AND** when a task has no filled slots, the tile omits the slots row (no muted empty-slot placeholder copy in the tile body)


#### Scenario [SC-PACK-224]: Editable tiles expose bottom full-width text Edit and Delete

- **GIVEN** an editable cards or tasks editor and a read-only live or staff view of the same kind of items
- **WHEN** the user compares those surfaces
- **THEN** editable **task** tiles show Edit and Delete as stacked full-width outline buttons with icons at the bottom (task-set action chrome parity; Delete MAY be negative/red)
- **AND** editable **answer** tiles keep their existing stacked action chrome
- **AND** live and staff views use the same list-tile chrome without Edit/Delete when not editable
- **AND** answer **list** grids keep playing-card chrome
- **AND** task **list** tiles use the portrait pack-card family chrome (SC-PACK-223/278)
- **AND** the task compose dialog answer pool uses mini-cards (not playing-card tiles) per SC-PACK-260

#### Scenario [SC-PACK-225]: Answer tiles use fixed 150×200 or 300×200 sizes

- **GIVEN** a surface listing an answer card without description and an answer card with description
- **WHEN** the user views those tiles
- **THEN** the card without description is 150 CSS pixels wide and 200 CSS pixels tall
- **AND** the card with description is 300 CSS pixels wide and 200 CSS pixels tall with a vertical splitter between content and description

#### Scenario [SC-PACK-226]: Task tiles are 300×200

- **GIVEN** a surface listing tasks
- **WHEN** the user views a task tile
- **THEN** the tile width is strictly less than its height (portrait pack-card family — supersedes the former fixed 300×200 requirement)
- **AND** the tile MUST NOT be fixed at 300 CSS pixels wide and 200 CSS pixels tall


#### Scenario [SC-PACK-227]: Answer and task tiles stay readable in dark theme

- **GIVEN** the application is in dark theme
- **WHEN** the user views answer and task list tiles including editable Edit controls
- **THEN** tile text and edit controls remain visible against the tile background
- **AND** the tile MUST NOT present light text on an unresolved white card background

### Requirement: Answer slots visible on every question list row

Wherever the client lists tasks/questions for review or editing (staff moderation hub preview, add-task-set page question list, task-set editor list, live pack drill-in task list), each task row MUST show its **filled** answer slots. A filled slot MUST display the referenced answer card’s textual content (same resolution as when viewing a live pack / task-set page), and MUST show the answer card’s description as secondary text when present (same product sense as peek/compose filled slots; omit secondary when description is empty). Empty slots MUST NOT be rendered on the list tile. When a task has **no** filled slots, the list tile MUST omit the slots row entirely and MUST NOT show muted empty-slot placeholder copy (e.g. `content.slotEmpty`) in the tile body. A generic filled placeholder (e.g. «заполнен») MUST NOT be shown when the card content is available. For staff preview of an add-task-set (`task_set`) request, answer cards used for slot resolution MUST come from the pack’s **live** answers (revision for add-task-set does not store cards). Staff MUST be able to see slot bindings without opening a separate nested tasks-only page. A separate top-level «Ответы» list on the staff hub for `task_set` is not required when slots already show card text.

#### Scenario [SC-PACK-127]: Staff hub and add-task-set lists show slots

- **GIVEN** a moderation preview or add-task-set page with tasks that have slots
- **WHEN** the questions list renders
- **THEN** each task row shows its filled answer slots
- **AND** empty slots are not shown on the list tile
- **AND** when a filled slot’s answer card has a non-empty description, that description appears as secondary text on the mini-card
- **AND** the same rule applies on staff request hub and on the author add-task-set list

#### Scenario [SC-PACK-134]: Staff task_set preview shows answer text in slots

- **GIVEN** an open add-task-set (`task_set`) moderation request whose tasks reference live answer cards by id
- **AND** those live cards have non-empty content
- **WHEN** staff opens the staff request hub preview
- **THEN** each filled slot shows that card’s content text
- **AND** MUST NOT show only a generic «filled» / «заполнен» placeholder for those slots

### Requirement: Task answer slots match peek slot chrome

On **task compose dialog** surfaces (slot row while creating/editing a question), answer slots MUST use the same approximate size and visual chrome as shared peek answer slots on the game board for this change: empty = dashed unlabeled; filled = answer mini-card with thin primary border; portrait aspect ratio ≈ pack task-set cards at the **half-size** tile band (see SC-PACK-260/277 and `game/board` SC-BOARD-51). Dense Quasar chips that read as a different scale from peek MUST NOT be the primary slot presentation on compose. On **task list / card tiles**, **filled** slots MUST use portrait answer mini-card chrome comparable to peek/compose filled slots (including thin primary border and optional answer description secondary); empty slots MUST be omitted from the list tile (not compact labeled empty chrome).

#### Scenario [SC-PACK-237]: Compose slots match peek scale

- **GIVEN** a verified editor is composing a task and filling answer slots in the task compose dialog
- **WHEN** the slot row is rendered
- **THEN** each empty slot uses peek-comparable dashed unlabeled chrome and pack task-set portrait proportion at the half-size band
- **AND** each filled slot shows the answer as a mini-card comparable to the peek filled slot including the thin primary border

#### Scenario [SC-PACK-238]: Task tile slots match peek scale

- **GIVEN** a task list or card tile showing a task with one or more filled slots (and optionally empty draft slots)
- **WHEN** the slots are rendered after this redesign
- **THEN** each filled slot uses portrait answer mini-card chrome comparable to peek/compose filled slots
- **AND** empty slots are not shown on the tile
- **AND** the compose dialog slot row remains the surface that also matches empty dashed peek chrome (SC-PACK-237)

### Requirement: Task compose uses a peek-like dialog

On task-set editing surfaces (creator/staff task-set editor and post-publish add-task-set), creating or editing a task MUST happen in a dialog, not in an always-visible inline compose form above the task list. When compose is allowed, the surface MUST expose an add-question control above the task list that opens an empty create dialog; when compose is not allowed (`viewOnly` / `readOnly`), the add control MUST be hidden. Activating Edit on a task tile MUST open the same dialog prefilled for that task only when compose is allowed. Inside the dialog the client MUST present an editable question, a difficulty control rendered as **three interactive difficulty dots** plus a **word-only** difficulty badge (SC-PACK-276 / SC-PACK-280) instead of a standalone numeric-only selector as the primary control, then a **centered** ordered slot row, then below it an answer pool of mini-cards comparable to the shared game peek answer pool (same half-size portrait band, grid up to 4×4 with vertical scroll when needed — not a catalog playing-card tile grid and not flat outline chips as the primary pool chrome). The dialog’s content width MUST be anchored to one row of **four** answer-pool mini-cards plus horizontal padding, with approximately **50 CSS px** extra slack allowed (SC-PACK-279). Empty slots MUST be dashed with no placeholder label text; filled slots MUST show the chosen answer as a mini-card with the thin primary border. Slot fill/clear/add/remove behavior and draft persistence rules remain as today. Successful save MUST close the dialog. The dialog MUST NOT show a Cancel button; dismissing without save MUST still be possible via backdrop click or Escape (SC-PACK-281). The primary Save / Add control MUST use full-width primary unelevated chrome comparable to the game peek submit button (SC-PACK-281). The compose question control font-size MUST NOT be required to match the game peek ×1.5 type (SC-BOARD-54). The **game peek** modal width and difficulty-dot size MUST remain unchanged by this requirement.

#### Scenario [SC-PACK-258]: Add question opens dialog above the task list

- **GIVEN** a verified editor on a task-set editing surface with compose allowed
- **WHEN** the editor activates the add-question control above the task list
- **THEN** a dialog opens for creating a task
- **AND** the page does not show a persistent inline task create/edit form above the list

#### Scenario [SC-PACK-259]: Edit task opens the same dialog

- **GIVEN** a verified editor viewing editable task tiles on a task-set editing surface
- **WHEN** the editor activates Edit on a task tile
- **THEN** the same task dialog opens prefilled with that task’s question, difficulty, and slots
- **AND** saving or dismissing (backdrop / Escape) closes the dialog without leaving an inline form on the page

#### Scenario [SC-PACK-260]: Task dialog matches peek slot and answer-pool layout

- **GIVEN** a verified editor has the task compose dialog open with available answer cards
- **WHEN** the dialog is shown
- **THEN** the question and difficulty header appear above a centered ordered slot row (dots at the left edge; word-only badge horizontally centered)
- **AND** empty slots use dashed chrome without placeholder slot-label text
- **AND** below the slots the answer choices appear as mini-cards comparable to the game peek answer pool (pack task-set portrait aspect ratio at the half-size band)
- **AND** the dialog content width is anchored to one row of four pool mini-cards plus horizontal padding with approximately 50 CSS px slack
- **AND** the dialog MUST NOT use a catalog playing-card tile grid as the answer picker
- **AND** the primary Save / Add control is a full-width primary unelevated button comparable to game peek submit
- **AND** no Cancel button is shown in the dialog actions

### Requirement: Task compose difficulty uses interactive dots and word badge

In the task compose dialog, difficulty MUST be chosen with three dots matching the pack task-set difficulty indicator (resting: filled count = current difficulty; **all** dots use that level’s single color — green / amber / red for 1 / 2 / 3 — same as pack task-set summary cards). Hovering (or equivalent pointer preview) over the N-th dot MUST preview filled dots `1…N` **all** in that level’s color N; a **click/tap** on the N-th dot MUST set difficulty to N (works without hover on touch). The dialog MUST show a status badge with **only** the difficulty **word** for the current value (лёгкая / средняя / сложная) — **without** a `Сложность:` prefix and not a digit-only or `1 — …` select label. Layout MUST match game peek: **dots at the left edge**, **badge horizontally centered** (not one centered dots+badge cluster). In the **task compose dialog only**, each difficulty dot’s visual diameter MUST be approximately the **same height** as the word difficulty badge (larger hit target than the 6px pack/peek resting dots). The previous primary difficulty dropdown MAY be removed; difficulty MUST remain constrained to 1|2|3. Game peek read-only dots MUST keep their existing size.

#### Scenario [SC-PACK-276]: Compose difficulty dots preview on hover and commit on click

- **GIVEN** a verified editor has the task compose dialog open with difficulty currently 1
- **WHEN** the editor hovers the third difficulty dot
- **THEN** all three dots preview as filled in the hard (red) difficulty color
- **WHEN** the editor clicks the second difficulty dot
- **THEN** the task difficulty becomes 2
- **AND** the badge shows the medium difficulty word only (no `Сложность:` prefix)
- **AND** the badge is horizontally centered in the dialog (not clustered with the dots)
- **AND** the resting dots sit at the left edge of the dialog content
- **AND** the resting dots show two filled amber dots and one empty outline
- **AND** each compose difficulty dot’s visual height approximately matches the word badge height

### Requirement: Compose slots and pool share peek mini-card chrome

Task compose empty slots, filled slots, and the answer pool MUST share the same visual contract as the game peek modal for this change: empty = dashed blank; filled = answer mini-card with thin primary border; pool = mini-cards with pack task-set aspect ratio at the half-size band, at most four columns and four visible rows with vertical scroll when there are more answers. This contract applies to the **task compose dialog**; **task list tiles** show filled slots with the same filled mini-card product sense (SC-PACK-238) and omit empty slots.

#### Scenario [SC-PACK-277]: Compose empty slot has no label; pool scrolls past 4×4

- **GIVEN** a verified editor opens task compose with more than sixteen available answer cards and at least one empty slot
- **WHEN** the dialog is shown
- **THEN** each empty slot shows dashed chrome with no slot-label text
- **AND** the answer pool shows at most four columns and four visible rows without scrolling
- **AND** further answers are reachable by scrolling the pool downward

## ADDED Requirements

### Requirement: Task list tile layout matches peek header and filled slots

On every surface that lists tasks as tiles, each task tile MUST present, top to bottom: (1) a header with three difficulty dots at the left edge and a word-only difficulty badge centered horizontally; (2) the task question at approximately **24 CSS px** (game peek question parity); (3) a single horizontal row of **filled** answer mini-cards only, **horizontally centered**, with horizontal scroll when the row overflows and visible air above the actions divider so the row is not flush against the bottom chrome. The tile MUST be portrait (width &lt; height) and use pack-card family resting chrome comparable to pack task-set summary cards. Answer mini-cards on the tile MUST show answer content and, when present, answer description secondary — the same product sense as filled slots in the game peek and task compose dialogs. Edit/Delete stacking rules remain as in the playing-card presentation requirement (outline + icon for tasks).

#### Scenario [SC-PACK-278]: Task tile shows dots, centered badge, question, filled slots with horizontal scroll

- **GIVEN** a task list with a task of difficulty 2 that has three filled slots whose combined mini-card width exceeds the tile content width
- **WHEN** the user views that task tile
- **THEN** two difficulty dots are filled (level color for difficulty 2) at the left of the header
- **AND** the word-only medium difficulty badge is horizontally centered in the header
- **AND** the question text appears below the header at approximately 24 CSS pixels
- **AND** three filled answer mini-cards appear in a horizontally centered row
- **AND** there is visible air between the slots row and the actions divider (or tile bottom when not editable)
- **AND** the slot row scrolls horizontally to reveal overflow without wrapping into a second row as the primary overflow strategy
- **AND** a task with no filled slots omits the slots row (no empty-slot placeholder copy in the tile body)

### Requirement: Task compose dialog is no wider than four answer mini-cards

The task compose dialog’s content width MUST be anchored to one row of **four** answer-pool mini-cards at the shared half-size tile band plus the dialog’s own horizontal padding, and MAY be approximately **50 CSS px** wider than that strict sum (comfort slack). The four-column answer pool MUST remain the width anchor. The game peek modal width MUST NOT be changed by this requirement.

#### Scenario [SC-PACK-279]: Compose dialog width fits four pool mini-cards

- **GIVEN** a verified editor opens the task compose dialog
- **WHEN** the dialog is shown at a desktop viewport
- **THEN** the dialog content width equals approximately four half-size answer mini-cards in one row plus horizontal padding plus about 50 CSS px slack (not the former fixed 360/400)
- **AND** the answer pool still lays out in at most four columns

### Requirement: Compose difficulty dots match badge height for hit targets

In the task compose dialog only, each interactive difficulty dot MUST render at approximately the same visual height as the word difficulty badge so pointer/touch targets are comfortable. Color, fill-count, hover preview, and click-to-commit behavior remain as in the interactive dots requirement. Game peek read-only dots MUST keep their existing smaller size.

#### Scenario [SC-PACK-280]: Compose dots are approximately badge-tall

- **GIVEN** a verified editor has the task compose dialog open
- **WHEN** the difficulty header is shown
- **THEN** each interactive difficulty dot’s visual height is approximately equal to the word badge’s height
- **AND** the game peek modal (when opened separately) still uses its existing smaller read-only dot size

### Requirement: Task compose Save matches peek submit; Cancel omitted

In the task compose dialog, the primary Save / Add control MUST be a **full-width** primary **unelevated** button with slim pack-card action height chrome comparable to the game peek submit button. Labels MUST remain the content save/add copy (not the game peek «Ответить» string). The dialog MUST NOT show a Cancel button. Closing without save MUST still be possible via backdrop click or Escape. The game peek submit control itself MUST remain unchanged.

#### Scenario [SC-PACK-281]: Compose Save is peek-like; Cancel is absent

- **GIVEN** a verified editor has the task compose dialog open
- **WHEN** the dialog actions are shown
- **THEN** a full-width primary unelevated Save / Add button is present with slim action-height chrome comparable to game peek submit
- **AND** no Cancel button is present in the dialog actions
- **AND** dismissing the dialog via backdrop or Escape closes it without leaving an inline form on the page
