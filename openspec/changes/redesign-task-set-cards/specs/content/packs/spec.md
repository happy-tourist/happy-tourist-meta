## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-04…06, 34 | unchanged (attribution without display author) |
| SC-PACK-130 | covered (client vitest — stats on task-set card) |
| SC-PACK-132 | covered (client vitest — short unpublished badge) |
| SC-PACK-135 | covered (client vitest — no author/coauthor; `#{n}` ordinal) |
| SC-PACK-228 | unchanged (catalog packs still 150×200) |
| SC-PACK-229 | covered (client vitest — redesigned task-set card) |
| SC-PACK-239 | covered (client vitest — stats rows + colored dots + pale dividers between all stats rows + above actions) |
| SC-PACK-240 | covered (client vitest — short status badges) |
| SC-PACK-241 | covered (client vitest — outline icon actions; short Снять/Вернуть; slim height) |
| SC-PACK-242 | covered (client vitest — light/dark readable) |
| SC-PACK-243 | covered (client vitest — `#{n}` title; total icon + «Заданий:»; short card actions) |
| SC-PACK-244 | covered (client vitest — column grid lead/label/count; vertical fill; reserved top without badge) |

Related: playing-card answer/task tiles — main SC-PACK-222…227 (unchanged). Catalog pack cards — SC-PACK-228 (unchanged). Lobby create/list labels — `lobby/rooms` delta.

## MODIFIED Requirements

### Requirement: Pack holds answer cards and task sets

A pack MUST contain answer cards and may contain one or more task sets. Each answer card MUST have textual content and MAY have a textual description. Each task MUST have a textual question, a difficulty of exactly `1`, `2`, or `3`, and one or more answer slots that reference answer cards from the same pack. Each task set MUST remain attributed to its contributing user for edit/ACL purposes (`authorUserId` and related rules). Task-set **labels shown in the UI** MUST NOT include the author’s display name or co-author labels; co-author labels MUST NOT be offered as a product feature. Creating or editing tasks MUST require that the pack already has at least one answer card in the editor’s draft. Any collection member who satisfies create/edit identity rules MUST be allowed to edit answer cards and tasks (no per-author ACL beyond collection + verify).

#### Scenario [SC-PACK-04]: Answer card has content and description

- **GIVEN** a verified user editing answers for a pack in their collection
- **WHEN** the user adds an answer card with non-empty content and an optional description
- **THEN** the card is stored on that pack

#### Scenario [SC-PACK-05]: Task has question, difficulty, and slots

- **GIVEN** a verified user editing a pack that already has answer cards
- **WHEN** the user adds a task with a question, difficulty in `1`|`2`|`3`, and at least one slot filled from those answer cards
- **THEN** the task is stored in a task set attributed to that user

#### Scenario [SC-PACK-06]: Invalid difficulty is rejected

- **GIVEN** a verified user editing tasks for a pack
- **WHEN** the user submits a task with difficulty outside `1`|`2`|`3`
- **THEN** the system rejects the write

#### Scenario [SC-PACK-34]: Tasks cannot be created without answer cards

- **GIVEN** a verified user with a pack that has zero answer cards in draft
- **WHEN** the user attempts to create a task or open task-set editing
- **THEN** the system rejects or prevents the action until at least one answer card exists

### Requirement: Live pack shows task-set summary and drill-in

On the live pack view, the client MUST show answer cards and a **list of task-set cards** (not an inline expansion of all questions). Each published task-set card MUST show at least the task count and a difficulty breakdown (counts for difficulties 1, 2, and 3) as distinct summary rows. Activating a published task-set card MUST navigate to a page that lists that set’s questions with answer slots. Soft-unpublished task-set cards follow the soft-unpublish task-set rules (muted chrome, non-enterable for non-staff).

#### Scenario [SC-PACK-130]: Live pack drills into task set questions

- **GIVEN** in-catalog pack P with at least one published task set S that has tasks of mixed difficulties
- **WHEN** a viewer opens the live pack page for P
- **THEN** the page lists S as a summary card with task count and difficulty 1/2/3 counts
- **AND** MUST NOT expand S’s questions inline on that page
- **AND WHEN** the viewer activates the card for S
- **THEN** the client opens a questions view for S showing each task’s answer slots

### Requirement: Staff soft-unpublish and republish task set

Moderator and admin MUST be able to soft-unpublish an entire **task set** on a live pack and to republish it without a new moderation request. Soft-unpublish of a task set MUST NOT remove individual questions as a separate product action. Soft-unpublish MUST keep the set stored; non-staff viewers MUST see the set card as muted with a short unpublished status badge (product sense «СНЯТО») and MUST NOT enter the set. Staff MUST still be able to Edit the soft-unpublished set (staff-save / lock session) and MUST have republish on the **task-set card** (editor and live list) and **inside** the task-set page. Staff MUST confirm before unpublishing a set; republish MUST NOT require confirmation. The system MUST reject unpublishing a task set when it is the **only** published task set on the pack. Soft-unpublish of individual tasks/questions MUST NOT be offered. Staff moderation queue MUST NOT expose task-set unpublish controls. Hard-delete of published or soft-unpublished task sets is out of scope for this requirement. On **task-set summary cards**, soft-unpublish and republish action labels MUST use short product copy («Снять» / «Вернуть» or equivalent i18n). Pack/catalog soft-unpublish copy and confirmation dialog wording MAY remain longer («Снять с публикации» / set confirm titles) and MUST NOT be required to match the card short labels.

#### Scenario [SC-PACK-131]: Staff cannot unpublish the only published task set

- **GIVEN** live pack P with exactly one published task set S and an authenticated staff user
- **WHEN** the staff user attempts to soft-unpublish S
- **THEN** the system rejects the attempt (or the client disables the control)
- **AND** S remains published

#### Scenario [SC-PACK-132]: Soft-unpublished task set is gray and non-enterable

- **GIVEN** live pack P with published task set S1 and soft-unpublished task set S2
- **WHEN** a non-staff viewer opens live P
- **THEN** S2’s card is muted with a short unpublished badge (e.g. «СНЯТО»)
- **AND** activating S2 MUST NOT open the questions view
- **AND** staff MAY still Edit S2 and choose republish on the card or inside the task-set page without confirmation
- **AND WHEN** staff chooses unpublish on a published set that is not the last published set
- **THEN** the client asks for confirmation before the API call

### Requirement: Task-set rows show author display name

Wherever the client lists or titles a task set (live pack summary, cards editor list, staff moderation hub, task-set / questions page header when a set label is shown, and any other task-set title surface in content), each set MUST be labeled **without** the contributing author’s display name and **without** co-author labels. Preferred form: «Набор заданий #{n}» (or equivalent i18n with a hash before the ordinal). Multiple sets MUST remain separate cards/rows (no collapsing). Internal attribution (`authorUserId`) and edit ACL MUST remain unchanged.

#### Scenario [SC-PACK-135]: Live and editor show author on every task-set row

- **GIVEN** pack P with two task sets by the same author whose displayName is «Мария»
- **WHEN** a viewer opens the live pack page, the cards editor task-set list, the staff hub set headings, or a task-set questions page header
- **THEN** each set label is of the form «Набор заданий #{n}» (or equivalent with `#`) **without** «от Мария» / author / coauthor text
- **AND** the two sets are not merged into one

### Requirement: Pack catalog and task-set lists use fixed card tiles

The unified packs **catalog** list MUST render each pack as a rounded card in a wrapping row with fixed size **150×200** CSS pixels. Each catalog card MUST show moderation/status chrome at the **top** when a status badge applies. On the catalog, a favorites star MUST appear at the **top-left**; the top-right MUST NOT host Edit. Catalog action controls MUST appear at the **bottom** as stacked full-width **text** buttons. The catalog card body MUST show a truncated pack title and a truncated pack **description**.

The **task-set** lists on the live pack and cards-editor surfaces MUST render each task set as a rounded card in a wrapping row. Task-set cards MUST use the denser summary chrome (status badge when applicable, title without author, task-count and difficulty 1/2/3 rows, bottom actions) defined in the task-set card chrome requirement. Width MUST stay near catalog width (~150–160 CSS pixels); height MAY be taller than 200 CSS pixels so the summary rows remain readable. Existing open/navigation and soft-unpublish rules MUST remain. Catalog pack cards MUST NOT adopt the task-set summary rows.

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
- **THEN** each task set is shown as a rounded summary card (width ~150–160px; height MAY exceed 200px)
- **AND** status chrome when applicable appears at the top
- **AND** Edit / soft-unpublish / republish when available appear as bottom full-width controls
- **AND** the body shows a truncated task-set label without author/coauthor
- **AND** the body shows task count and difficulty 1/2/3 breakdown rows

## ADDED Requirements

### Requirement: Task-set cards use denser summary chrome

On live pack and cards-editor task-set lists, each task-set card MUST present, top to bottom: optional short status badge; title «Набор заданий #{n}» (no author); a total-tasks row with a leading document-style icon and a «Заданий:» (or equivalent) label plus count; three difficulty rows (1 / 2 / 3) each with a three-dot indicator (filled count equals difficulty; filled dots use distinct colors for difficulties 1 / 2 / 3 in the product sense green / amber / red; unfilled dots are outline rings) and the count of tasks at that difficulty with capitalized difficulty labels ending in a colon; then bottom action controls. **Pale (low-contrast) horizontal dividers** MUST appear after the total row, **between each consecutive difficulty row**, and **above the action controls** when actions are present. Stats rows MUST use a three-column rhythm: a leading column whose width matches the three-dot group (total-row icon centered in that column), difficulty/total labels sharing one left edge, and counts right-aligned. Card content (badge band → title → stats → actions) MUST distribute to fill the card height; when no status badge applies, empty space MUST remain only in the reserved top status band. Difficulty rows MUST remain scannable in both light and dark themes. Action controls that apply to the card (Edit, soft-unpublish, republish, and similar) MUST be full-width **outline** buttons with a leading icon and text label (not icon-only corner affordances) and MUST use a **slim** control height in the product sense of roughly one stats-row (~28–32 CSS px), not the default tall Quasar button. Soft-unpublish on the card MUST read «Снять» (or equivalent short i18n); republish on the card MUST read «Вернуть» (or equivalent short i18n). Hover scale / enlarge of the card MUST NOT be required. Cascade-gap highlight on editor cards MUST remain when applicable.

#### Scenario [SC-PACK-239]: Task-set card shows counts and difficulty dots

- **GIVEN** a live or editor task-set list for set S with tasks of difficulties 1, 2, and 3
- **WHEN** the user views S’s card
- **THEN** the card shows a total tasks count row
- **AND** shows separate rows for difficulties 1, 2, and 3 with counts
- **AND** each difficulty row uses a three-dot indicator with filled count equal to that difficulty
- **AND** filled dots for difficulties 1, 2, and 3 use distinct colors (green / amber / red product sense)
- **AND** unfilled dots are outline rings rather than solid muted fills
- **AND** a pale horizontal divider separates the total row from the first difficulty row
- **AND** a pale horizontal divider separates each consecutive pair of difficulty rows
- **AND** a pale horizontal divider separates the last difficulty row from the action controls when actions are present

#### Scenario [SC-PACK-240]: Task-set card uses short status badges

- **GIVEN** task sets in pending, needs_revision, and soft-unpublished states on live or editor lists
- **WHEN** those cards render status chrome
- **THEN** badges use short labels in the product sense of «НА ПРОВЕРКЕ», «ДОРАБОТАТЬ», and «СНЯТО» (or equivalent i18n)
- **AND** MUST NOT rely on long page-subtitle phrases as the only badge text on the card

#### Scenario [SC-PACK-241]: Task-set card actions are outline with icons

- **GIVEN** a task-set card that exposes Edit and/or soft-unpublish
- **WHEN** the user views the card actions
- **THEN** each action is a full-width outline control with a leading icon and text
- **AND** MUST NOT be an icon-only control in a card corner
- **AND** each action uses a slim height in the product sense of roughly one stats-row (~28–32 CSS px), not the default tall button
- **AND** soft-unpublish on the card uses short «Снять» (or equivalent) rather than pack/catalog «Снять с публикации»
- **AND WHEN** republish is shown on the card
- **THEN** it uses short «Вернуть» (or equivalent)

#### Scenario [SC-PACK-242]: Task-set cards stay readable in dark theme

- **GIVEN** the application is in dark theme
- **WHEN** the user views task-set summary cards including status and actions
- **THEN** title, stats, badge, and actions remain visible against the card background
- **AND** the card MUST NOT present light text on an unresolved white card background

#### Scenario [SC-PACK-243]: Task-set card title hash and total-row chrome

- **GIVEN** a live or editor task-set list with set ordinal n
- **WHEN** the user views that set’s summary card
- **THEN** the title includes a hash before the ordinal (product sense «Набор заданий #{n}»)
- **AND** the total-tasks row shows a leading document-style icon and a «Заданий:» label (or equivalent) with the total count

#### Scenario [SC-PACK-244]: Task-set card column grid and vertical fill

- **GIVEN** a live or editor task-set summary card with total and difficulty rows
- **WHEN** the user views the card
- **THEN** the leading icon of the total row is centered within the horizontal span occupied by the three-dot indicators on difficulty rows
- **AND** the labels «Заданий:», «Лёгкие:», «Средние:», and «Сложные:» (or equivalent) share a common left edge
- **AND** the numeric counts are right-aligned with each other
- **AND** badge, title, stats, and actions are distributed to fill the card height
- **AND WHEN** the card has no status badge
- **THEN** empty space remains only in the reserved top status band (other blocks keep the fill rhythm)
