## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-210 | covered (`packContentCsv` + `ContentPackEditorCsv`) |
| SC-PACK-211 | covered (`packContentCsv` + `ContentPackEditorCsv`) |
| SC-PACK-212 | covered (`ContentPackEditorCsv`) |
| SC-PACK-213 | covered (`packContentCsv` + `PackTasksCsvControls`) |
| SC-PACK-214 | covered (`packContentCsv` + `PackTasksCsvControls`) |
| SC-PACK-215 | covered (`packContentCsv` + `PackTasksCsvControls`) |
| SC-PACK-216 | covered (`PackTasksCsvControls`) |
| SC-PACK-217 | covered (shared `PackTasksCsvControls` on both task pages) |
| SC-PACK-218 | covered (framed CSV panel + `PackTasksCsvControls` / `ContentPackEditorCsv`) |
| SC-PACK-219 | covered (`ContentPackEditorCsv`) |
| SC-PACK-220 | covered (`PackTasksCsvControls`) |
| SC-PACK-221 | covered (Import → file picker; framed controls) |
| SC-PACK-222 | covered (`PackAnswerCardTile` vertical split) |
| SC-PACK-223 | covered (`PackTaskTile` chips row) |
| SC-PACK-224 | covered (bottom full-width text Edit/Delete) |
| SC-PACK-225 | covered (150×200 / 300×200 answer sizes) |
| SC-PACK-226 | covered (task 300×200) |
| SC-PACK-227 | covered (contrast tokens) |
| SC-PACK-228 | covered (`PackListCardTile` catalog 150×200 + description) |
| SC-PACK-229 | covered (task-set 150×200 + bottom actions) |
| SC-PACK-230 | covered (server mocha staff draft) |

Related: answer/task model and «tasks need cards» gate — main `content/packs`. This change adds CSV import/export, framed CSV panel UX, playing-card presentation (sizes/split/actions), catalog + task-set cards, and staff draft status fix.

## ADDED Requirements

### Requirement: CSV answer import and export on cards editor

On the **cards pack** editing surface (answer cards), a verified editor who MAY edit cards MUST be able to **export** the current draft answer cards as a CSV download and **import** a CSV that **appends** new answer cards to the end of the draft (existing cards MUST NOT be replaced or re-ordered by import). Each non-empty CSV line MUST represent one answer card. The field separator MUST be the semicolon character `;` with **no** quoting/escaping dialect: every `;` splits fields. The first field MUST be the card content; any following fields MUST be description paragraphs joined into the card description with newline separators. Empty description fields MUST yield an empty description. Import MUST respect the same read-only / blocked-pack rules as manual card edits: when cards are not editable, import MUST be unavailable. Export MUST be unavailable when there are zero draft answer cards. On import failure the client MUST show an error **inside the CSV controls frame** and MUST NOT append partial cards from that file.

#### Scenario [SC-PACK-210]: Export answers downloads semicolon CSV

- **GIVEN** a verified editor on the cards editor with one or more draft answer cards
- **WHEN** the user exports answers
- **THEN** the client downloads a CSV where each card is one line `content;paragraph1;paragraph2;…` using `;` as separator
- **AND** the download name is derived from the pack title

#### Scenario [SC-PACK-211]: Import answers appends cards

- **GIVEN** a verified editor on the cards editor with editable cards and N existing draft answer cards
- **WHEN** the user imports a valid answers CSV with M non-empty lines
- **THEN** the draft has N+M answer cards
- **AND** the M imported cards are appended after the previous ones
- **AND** duplicate content relative to existing cards is allowed

#### Scenario [SC-PACK-212]: Answer import unavailable when cards are read-only

- **GIVEN** a cards editor surface where answer cards are not editable (blocked pack or non-editable editor role)
- **WHEN** the user views import/export controls for answers
- **THEN** answer import is disabled or omitted
- **AND** the draft answer cards are unchanged

#### Scenario [SC-PACK-219]: Answer export unavailable when there are no cards

- **GIVEN** a verified editor on the cards editor with zero draft answer cards
- **WHEN** the user views export controls for answers
- **THEN** answer export is disabled or omitted

### Requirement: CSV task import and export on task editing surfaces

On the **add-task-set** surface and on the **task-set editor** surface for an existing set, a verified editor who MAY edit that set’s tasks MUST be able to **export** the **current** task set as CSV and **import** a CSV that **appends** tasks to that set. Each non-empty CSV line MUST represent one task: `question;difficulty;slotText1;slotText2;…` with `;` as separator and no escaping. The `difficulty` field MUST be `1`, `2`, or `3` when present and valid; when empty or not a valid difficulty the imported task MUST use difficulty `1`. Each slot text after difficulty MUST resolve to an answer card in the **current answer context** by exact match on that card’s content text (first match when duplicates exist). The answer context for add-task-set MUST be the pack’s live answer cards; for the creator task-set editor MUST be the pack draft answer cards. Export MUST cover only the task set being edited; the download name MUST use the pack title and a numeric set index. Export MUST be unavailable when the current set has zero tasks. On import failure the client MUST show an error **inside the CSV controls frame** and MUST NOT append tasks from that file.

#### Scenario [SC-PACK-213]: Export tasks downloads current set CSV

- **GIVEN** a verified editor on add-task-set or task-set editor with one or more tasks in the current set
- **WHEN** the user exports tasks
- **THEN** the client downloads a CSV where each task is one line `question;difficulty;slotContent1;slotContent2;…`
- **AND** filled slots are serialized as the referenced answer card’s content text
- **AND** the download name includes the pack title and a set number

#### Scenario [SC-PACK-214]: Import tasks appends when all slot answers exist

- **GIVEN** a verified editor on add-task-set or task-set editor
- **AND** the answer context has cards whose content texts include every slot text in the CSV
- **WHEN** the user imports a valid tasks CSV with M non-empty lines
- **THEN** M tasks are appended to the current set

#### Scenario [SC-PACK-215]: Import tasks rejects whole file when slot answers are missing

- **GIVEN** a verified editor on a task editing surface with a non-empty answer context
- **WHEN** the user imports a tasks CSV that references a slot text absent from the answer context
- **THEN** the client refuses the entire import
- **AND** lists the missing answer texts
- **AND** the working task set is unchanged

#### Scenario [SC-PACK-216]: Task import unavailable without answer context

- **GIVEN** a task editing surface whose answer context has zero answer cards
- **WHEN** the user views task CSV import
- **THEN** task import is disabled or omitted

#### Scenario [SC-PACK-217]: Task CSV controls on both task surfaces

- **GIVEN** a verified editor who may edit tasks via add-task-set and via an existing task-set editor
- **WHEN** the user opens either surface
- **THEN** both surfaces expose the same task CSV import and export affordances (subject to read-only and answer-gate rules)

#### Scenario [SC-PACK-220]: Task export unavailable when the current set is empty

- **GIVEN** a verified editor on add-task-set or task-set editor with zero tasks in the current set
- **WHEN** the user views export controls for tasks
- **THEN** task export is disabled or omitted

### Requirement: CSV controls use a framed panel with format hint and inline errors

On surfaces that expose CSV import/export, Export and Import MUST appear together inside a shared visual frame. Below those controls the client MUST show a short CSV format recommendation for that surface (answers: `content;paragraph1;paragraph2;…`; tasks: `question;difficulty;slot1;slot2;…`). The client MUST NOT attach a tooltip to Import as the primary format/help path. Activating Import (when enabled) MUST open the system file picker **immediately** without an intermediate in-app modal. When import fails, the client MUST present the error **inside that frame**, MUST leave the draft unchanged for that attempt, and MUST keep the frame visible so the user can retry. Empty question or empty slot columns MAY still be loaded; existing submit minima and empty-slot rules continue to block moderation submit until the editor fixes them manually. Export MUST download immediately when enabled.

#### Scenario [SC-PACK-218]: Failed task import keeps framed feedback and draft unchanged

- **GIVEN** a task CSV import that cannot be applied because required answer texts are missing
- **WHEN** the user selects that file after activating Import
- **THEN** the client shows that import is not possible and lists the missing answers inside the CSV controls frame
- **AND** the working task set is unchanged
- **AND** no import modal is required for that feedback

#### Scenario [SC-PACK-221]: Import opens the system file picker directly

- **GIVEN** a verified editor for whom CSV import is available
- **WHEN** the user activates Import
- **THEN** the system file picker opens without an intermediate format modal
- **AND** the CSV format recommendation remains visible in the framed controls

### Requirement: Playing-card presentation for answer cards and tasks

Wherever the client shows a list or selectable set of pack **answer cards** or **tasks** (cards editor, task editors, slot picker, live pack/tasks view, staff moderation views), each item MUST be rendered as a rounded playing-card tile in a wrapping row. An answer card tile MUST use a **vertical** splitter: content on one side and description on the other when description is present; when description is empty the tile MUST be **150×200** without an empty description half; when description is present the tile MUST be **300×200**. A task tile MUST be **300×200** with a **vertical** splitter between question and slots; filled slot answers MUST appear **side by side** as filled slot chips (not a vertical stack of lines); task difficulty MUST appear at the **top-left**. Content/question/slot type MUST be roughly twice the previous body size so tiles remain readable at these widths. Tile background and foreground MUST remain readable in both light and dark themes. On surfaces where the item is editable, Edit and Delete MUST appear as **full-width text buttons stacked at the bottom** of the tile (not icon buttons in a corner). Changing content MUST be available through Edit, not by treating a body click as edit. Add/create forms MUST remain above the card grid. Overflow text MUST scroll inside its half rather than unbounded grow of the row.

#### Scenario [SC-PACK-222]: Answer cards render as vertically split playing-card tiles

- **GIVEN** a surface that lists answer cards including one with description and one without
- **WHEN** the user views the list
- **THEN** each card is a rounded tile in a wrapping row
- **AND** the card with description shows content and description side by side with a vertical splitter
- **AND** the card without description is a 150×200 rectangle without an empty description half
- **AND** long description text scrolls inside its half

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
- **AND** slot picker surfaces use the same answer-card chrome for selection

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

### Requirement: Pack catalog and task-set lists use fixed card tiles

The unified packs **catalog** list and the **task-set** lists on the live pack and cards-editor surfaces MUST render each pack or task set as a rounded card in a wrapping row with fixed size **150×200** CSS pixels. Each card MUST show moderation/status chrome at the **top** when a status badge applies. On the catalog, a favorites star MUST appear at the **top-left**; the top-right MUST NOT host Edit. Action controls that apply to the card (Edit, soft-unpublish, republish, and similar) MUST appear at the **bottom** as stacked full-width **text** buttons. The catalog card body MUST show a truncated pack title and a truncated pack **description**. Task-set card bodies MUST show a truncated task-set label. Existing open/navigation and soft-unpublish rules MUST remain.

#### Scenario [SC-PACK-228]: Catalog packs render as 150×200 cards with description

- **GIVEN** an authenticated user on the packs catalog with at least one pack row that has a description
- **WHEN** the user views the list
- **THEN** each pack is shown as a 150×200 card
- **AND** a favorites star control is at the top-left
- **AND** status chrome when applicable appears at the top
- **AND** the body shows a truncated pack title and truncated description
- **AND** Edit when available is a bottom full-width text control (not top-right)

#### Scenario [SC-PACK-229]: Task-set lists render as 150×200 cards with bottom actions

- **GIVEN** a live pack or cards editor surface with one or more task sets
- **WHEN** the user views the task-set list
- **THEN** each task set is shown as a 150×200 card
- **AND** status chrome when applicable appears at the top
- **AND** Edit when available is a bottom full-width text control
- **AND** the body shows a truncated task-set label

### Requirement: Staff do not see false draft after direct staff save

When a moderator or admin saves pack content through the staff direct-save path (no moderation queue), task-set and pack moderation marks shown to that staff viewer MUST NOT report author-facing **draft** solely because a retained working copy still differs from live. Open author **pending** or **needs_revision** marks MUST still appear when an author request is open. After staff direct-save, staff MUST see sets as live/published for status purposes unless a real open author request applies.

#### Scenario [SC-PACK-230]: Staff save does not leave draft badge for staff

- **GIVEN** a published in-catalog pack with no open author moderation request
- **AND** a staff editor who holds the edit lock and saves changes via staff direct-save
- **WHEN** that staff viewer then sees the pack or its task-set status marks
- **THEN** the client does not show a draft / needs-moderation status for those sets solely due to working≠live
- **AND** the saved content remains live for non-staff viewers per existing staff-save rules
