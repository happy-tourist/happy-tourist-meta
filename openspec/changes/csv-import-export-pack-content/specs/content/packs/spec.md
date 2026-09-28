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
| SC-PACK-218 | covered (`PackTasksCsvControls` + editor unreadable-file) |

Related: answer/task model and «tasks need cards» gate — main `content/packs` (SC-PACK-04, SC-PACK-34). This change only adds CSV import/export on editor surfaces.

## ADDED Requirements

### Requirement: CSV answer import and export on cards editor

On the **cards pack** editing surface (answer cards), a verified editor who MAY edit cards MUST be able to **export** the current draft answer cards as a CSV download and **import** a CSV that **appends** new answer cards to the end of the draft (existing cards MUST NOT be replaced or re-ordered by import). Each non-empty CSV line MUST represent one answer card. The field separator MUST be the semicolon character `;` with **no** quoting/escaping dialect: every `;` splits fields. The first field MUST be the card content; any following fields MUST be description paragraphs joined into the card description with newline separators. Empty description fields MUST yield an empty description. Import MUST respect the same read-only / blocked-pack rules as manual card edits: when cards are not editable, import MUST be unavailable. After a successful import the file dialog MUST close; on failure the client MUST show an error and MUST NOT append partial cards from that file.

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

### Requirement: CSV task import and export on task editing surfaces

On the **add-task-set** surface and on the **task-set editor** surface for an existing set, a verified editor who MAY edit that set’s tasks MUST be able to **export** the **current** task set as CSV and **import** a CSV that **appends** tasks to that set. Each non-empty CSV line MUST represent one task: `question;difficulty;slotText1;slotText2;…` with `;` as separator and no escaping. The `difficulty` field MUST be `1`, `2`, or `3` when present and valid; when empty or not a valid difficulty the imported task MUST use difficulty `1`. Each slot text after difficulty MUST resolve to an answer card in the **current answer context** by exact match on that card’s content text (first match when duplicates exist). The answer context for add-task-set MUST be the pack’s live answer cards; for the creator task-set editor MUST be the pack draft answer cards. Export MUST cover only the task set being edited; the download name MUST use the pack title and a numeric set index. After successful import the file dialog MUST close; on failure the client MUST show an error and MUST NOT append tasks from that file.

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
- **AND** each imported task’s slots reference the matched answer cards by identity
- **AND** empty or invalid difficulty fields become difficulty `1`

#### Scenario [SC-PACK-215]: Import tasks blocked when slot answers are missing

- **GIVEN** a verified editor on add-task-set or task-set editor with a non-empty answer context
- **AND** a tasks CSV that references at least one slot text not present as any answer card content
- **WHEN** the user imports that CSV
- **THEN** the import is rejected for the whole file
- **AND** the client reports which answer texts are missing
- **AND** the current task set is unchanged

#### Scenario [SC-PACK-216]: Task import unavailable without answer cards

- **GIVEN** a verified editor on add-task-set or task-set editor whose answer context has zero answer cards
- **WHEN** the user views task import controls
- **THEN** task import is disabled or omitted with guidance that answers are required first
- **AND** no tasks are appended

#### Scenario [SC-PACK-217]: Task CSV controls on both task surfaces

- **GIVEN** a verified editor who may edit tasks via add-task-set and via an existing task-set editor
- **WHEN** the user opens either surface
- **THEN** both surfaces expose the same task CSV import and export affordances (subject to read-only and answer-gate rules)

### Requirement: CSV import uses a file picker dialog with clear failure feedback

Choosing CSV import MUST open a file-selection dialog. On successful parse and apply the dialog MUST close. When import is impossible (including missing answer texts for tasks, or unreadable file), the client MUST present an error explaining that import failed and MUST leave the draft unchanged for that attempt. Empty question or empty slot columns MAY still be loaded; existing submit minima and empty-slot rules continue to block moderation submit until the editor fixes them manually.

#### Scenario [SC-PACK-218]: Failed task import keeps dialog feedback and draft unchanged

- **GIVEN** a task CSV import that cannot be applied because required answer texts are missing
- **WHEN** the user selects that file
- **THEN** the client shows that import is not possible and lists the missing answers
- **AND** the working task set is unchanged
