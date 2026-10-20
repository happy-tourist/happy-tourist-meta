## Traceability

| SC-ID | Coverage target |
|-------|-----------------|
| SC-PACK-237 | modified (compose still matches peek chrome — half size + filled border) |
| SC-PACK-238 | modified (PackTaskTile carve-out) |
| SC-PACK-258 | covered (dialog shell — unchanged intent) |
| SC-PACK-259 | covered (edit prefills — unchanged intent) |
| SC-PACK-260 | modified (slot/pool chrome parity — polish) |
| SC-PACK-263 | modified (reuse without used chrome — filled border only on slots) |
| SC-PACK-276 | pending (difficulty dots + word-only badge, centered) |
| SC-PACK-277 | pending (compose empty/filled slots + pool — half size + filled border) |

Related game peek chrome: `game/board` SC-BOARD-50…54. List playing-card tiles SC-PACK-222…224 remain out of compose pool. **Apply scope:** client compose chrome only — task `difficulty` `1|2|3` and slot `answerCardId` persistence via existing `/api/content/*` remain unchanged (no new HTTP/DB fields, no server route/schema edits). Interactive dots (SC-PACK-276) only change the compose **control**; the persisted value stays `1|2|3` as today. Peek portrait unlabeled AR applies to compose dialog + game peek; **not** to `PackTaskTile` list rows (SC-PACK-238 carve-out). Game-only question ×1.5 (SC-BOARD-54) does **not** apply to compose `q-input`.

## MODIFIED Requirements

### Requirement: Task answer slots match peek slot chrome

On **task compose dialog** surfaces (slot row while creating/editing a question), answer slots MUST use the same approximate size and visual chrome as shared peek answer slots on the game board for this change: empty = dashed unlabeled; filled = answer mini-card with thin primary border; portrait aspect ratio ≈ pack task-set cards at the **half-size** tile band (see SC-PACK-260/277 and `game/board` SC-BOARD-51). Dense Quasar chips that read as a different scale from peek MUST NOT be the primary slot presentation on compose. On **task list / card tiles** that display slot labels (`PackTaskTile` and equivalent list rows), slots MAY keep compact labeled empty-slot chrome and MUST NOT be required to adopt the peek modal’s unlabeled portrait mini-card aspect ratio (readability carve-out — see SC-PACK-238).

#### Scenario [SC-PACK-237]: Compose slots match peek scale

- **GIVEN** a verified editor is composing a task and filling answer slots in the task compose dialog
- **WHEN** the slot row is rendered
- **THEN** each empty slot uses peek-comparable dashed unlabeled chrome and pack task-set portrait proportion at the half-size band
- **AND** each filled slot shows the answer as a mini-card comparable to the peek filled slot including the thin primary border

#### Scenario [SC-PACK-238]: Task tile slots match peek scale

- **GIVEN** a task list or card tile (`PackTaskTile`) showing slot labels for a task
- **WHEN** the slots are rendered after the peek/compose chrome redesign
- **THEN** those slots MAY keep empty-slot label text and compact sizes
- **AND** they MUST NOT be required to match the peek modal’s unlabeled portrait mini-card aspect ratio
- **AND** the compose dialog slot row (not the list tile) remains the surface that matches peek chrome (SC-PACK-237)

### Requirement: Task compose uses a peek-like dialog

On task-set editing surfaces (creator/staff task-set editor and post-publish add-task-set), creating or editing a task MUST happen in a dialog, not in an always-visible inline compose form above the task list. When compose is allowed, the surface MUST expose an add-question control above the task list that opens an empty create dialog; when compose is not allowed (`viewOnly` / `readOnly`), the add control MUST be hidden. Activating Edit on a task tile MUST open the same dialog prefilled for that task only when compose is allowed. Inside the dialog the client MUST present an editable question, a difficulty control rendered as **three interactive difficulty dots** plus a **word-only** difficulty badge (SC-PACK-276) instead of a standalone numeric-only selector as the primary control, then a **centered** ordered slot row, then below it an answer pool of mini-cards comparable to the shared game peek answer pool (same half-size portrait band, grid up to 4×4 with vertical scroll when needed — not a catalog playing-card tile grid and not flat outline chips as the primary pool chrome). Empty slots MUST be dashed with no placeholder label text; filled slots MUST show the chosen answer as a mini-card with the thin primary border. Slot fill/clear/add/remove behavior and draft persistence rules remain as today. Cancel or successful save MUST close the dialog. The compose question control font-size MUST NOT be required to match the game peek ×1.5 type (SC-BOARD-54).

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
- **THEN** the question and difficulty header (dots + word-only badge, centered) appear above a centered ordered slot row
- **AND** empty slots use dashed chrome without placeholder slot-label text
- **AND** below the slots the answer choices appear as mini-cards comparable to the game peek answer pool (pack task-set portrait aspect ratio at the half-size band)
- **AND** the dialog MUST NOT use a catalog playing-card tile grid as the answer picker

### Requirement: Task compose allows the same answer card in multiple slots

On task compose surfaces (task compose dialog on task-set editor and add-task-set), filling answer slots MUST allow the same answer card to be selected into any number of slots, including every slot of the task. After a card is placed in one slot, its pool mini-card MUST remain selectable for other empty slots. The client MUST NOT disable that pool card solely because it is already used in a slot, and MUST NOT present a distinct «already used» highlight or consumed-vs-resting state that marks the card as spent (including MUST NOT apply the filled-slot primary border to the pool card). Clicking a filled slot MAY still clear that slot. Persistence and cascade rules for slot `answerCardId` references remain unchanged (multiple slots MAY reference the same card id).

#### Scenario [SC-PACK-263]: Same answer card fills multiple compose slots without used chrome

- **GIVEN** a verified editor has the task compose dialog open with at least one answer card and two or more empty slots
- **WHEN** the editor places that same answer card into more than one slot
- **THEN** each of those slots shows that card as a filled mini-card with the thin primary border
- **AND** the answer-pool card for that card remains available for further empty slots
- **AND** the pool card is not shown in a distinct already-used disabled or highlighted consumed state solely because it appears in a slot

## ADDED Requirements

### Requirement: Task compose difficulty uses interactive dots and word badge

In the task compose dialog, difficulty MUST be chosen with three dots matching the pack task-set difficulty indicator (resting: filled count = current difficulty; **all** dots use that level’s single color — green / amber / red for 1 / 2 / 3 — same as `PackTaskSetCardTile`). Hovering (or equivalent pointer preview) over the N-th dot MUST preview filled dots `1…N` **all** in that level’s color N; a **click/tap** on the N-th dot MUST set difficulty to N (works without hover on touch). Beside the dots the dialog MUST show a status badge with **only** the difficulty **word** for the current value (лёгкая / средняя / сложная) — **without** a `Сложность:` prefix and not a digit-only or `1 — …` select label. The dots+badge group SHOULD be horizontally centered like the game peek header. The previous primary difficulty dropdown MAY be removed; difficulty MUST remain constrained to 1|2|3.

#### Scenario [SC-PACK-276]: Compose difficulty dots preview on hover and commit on click

- **GIVEN** a verified editor has the task compose dialog open with difficulty currently 1
- **WHEN** the editor hovers the third difficulty dot
- **THEN** all three dots preview as filled in the hard (red) difficulty color
- **WHEN** the editor clicks the second difficulty dot
- **THEN** the task difficulty becomes 2
- **AND** the badge shows the medium difficulty word only (no `Сложность:` prefix)
- **AND** the resting dots show two filled amber dots and one empty outline

### Requirement: Compose slots and pool share peek mini-card chrome

Task compose empty slots, filled slots, and the answer pool MUST share the same visual contract as the game peek modal for this change: empty = dashed blank; filled = answer mini-card with thin primary border; pool = mini-cards with pack task-set aspect ratio at the half-size band, at most four columns and four visible rows with vertical scroll when there are more answers. This contract applies to the **task compose dialog** only; catalog/list task tiles (`PackTaskTile`) MAY keep compact labeled empty slots and MUST NOT be required to adopt the peek portrait mini-card grid (same carve-out as SC-PACK-238).

#### Scenario [SC-PACK-277]: Compose empty slot has no label; pool scrolls past 4×4

- **GIVEN** a verified editor opens task compose with more than sixteen available answer cards and at least one empty slot
- **WHEN** the dialog is shown
- **THEN** each empty slot shows dashed chrome with no slot-label text
- **AND** the answer pool shows at most four columns and four visible rows without scrolling
- **AND** further answers are reachable by scrolling the pool downward
