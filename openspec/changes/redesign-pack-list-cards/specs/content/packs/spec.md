## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-228 | covered (client vitest — catalog pack card 180×260 + set preview) |
| SC-PACK-229 | unchanged (task-set cards) |
| SC-PACK-249 | covered (client vitest — set preview rows + overflow «ещё K») |
| SC-PACK-250 | covered (client vitest — outline+icon catalog actions; whole-card open) |
| SC-PACK-251 | covered (client vitest — revise badge red outline; uppercase title) |
| SC-PACK-252 | covered (server mocha — listCatalog lightweight taskSetsPreview) |
| SC-PACK-253 | covered (server mocha — soft-unpub + author neverLive in preview) |

Related: task-set summary chrome — SC-PACK-229/239…248 (unchanged). Playing-cards — SC-PACK-222…227 (unchanged).

## MODIFIED Requirements

### Requirement: Pack catalog and task-set lists use fixed card tiles

The unified packs **catalog** list MUST render each pack as a rounded card in a wrapping row with resting size about **180×260** CSS pixels (width MUST stay near 180; height MAY grow slightly when four set rows, overflow caption, and stacked actions are present). Each catalog card MUST show moderation/status chrome at the **top** when a status badge applies. On the catalog, a favorites star MUST appear at the **top-left**; the top-right MUST NOT host Edit. Catalog action controls MUST appear at the **bottom** as stacked full-width **outline** buttons with a leading icon and text label (slim height in the product sense ~28–32 CSS px, same pattern as task-set cards). The catalog card body MUST show a truncated pack title rendered in **uppercase**, a truncated pack **description**, and a **preview list of task sets** for that pack (not difficulty-breakdown rows from the task-set card chrome). Activating the catalog card body (outside action controls and the favorites star) MUST open the pack as today (live or Edit per existing list routing rules). Individual set-preview rows MUST NOT be separate navigation targets.

The **task-set** lists on the live pack and cards-editor surfaces MUST render each task set as a rounded card in a wrapping row. Task-set cards MUST use the denser summary chrome (status badge when applicable, title without author, task-count and difficulty 1/2/3 rows, bottom actions) defined in the task-set card chrome requirement. Width MUST stay near ~150–160 CSS pixels; height MAY be taller than 200 CSS pixels so the summary rows remain readable. Existing open/navigation and soft-unpublish rules MUST remain.

#### Scenario [SC-PACK-228]: Catalog packs render as 150×200 cards with description

- **GIVEN** an authenticated user on the packs catalog with at least one pack row that has a description and one or more task sets
- **WHEN** the user views the list
- **THEN** each pack is shown as a rounded card about 180 CSS pixels wide and about 260 CSS pixels tall (height MAY grow slightly with content)
- **AND** a favorites star control is at the top-left
- **AND** status chrome when applicable appears at the top (revise badge top-right on the mock)
- **AND** the body shows a truncated uppercase pack title and truncated description
- **AND** the body shows a preview of that pack’s task sets (label «Набор заданий #{n}» or equivalent, task count, leading document-style icon)
- **AND** Edit / soft-unpublish / republish when available are bottom full-width outline+icon controls (not top-right; not text-only)

#### Scenario [SC-PACK-229]: Task-set lists render as 150×200 cards with bottom actions

- **GIVEN** a live pack or cards editor surface with one or more task sets
- **WHEN** the user views the task-set list
- **THEN** each task set is shown as a rounded summary card (width ~150–160px; height MAY exceed 200px)
- **AND** status chrome when applicable appears at the top
- **AND** Edit / soft-unpublish / republish when available appear as bottom full-width controls
- **AND** the body shows a truncated task-set label without author/coauthor
- **AND** the body shows task count and difficulty 1/2/3 breakdown rows

## ADDED Requirements

### Requirement: Catalog pack cards preview existing task sets

On the unified packs catalog card, the client MUST list existing task sets for the pack as preview rows. Each visible row MUST show a leading document-style icon (the same product icon family used for the task-set card total row), the label «Набор заданий #{n}» (or equivalent i18n with a hash before the ordinal, no author), and the task count for that set aligned to the right. The card MUST show at most **four** set rows; when more sets exist, it MUST show an overflow caption of the product sense «ещё {k}» (or equivalent i18n) where `{k}` is the number of sets not shown. Soft-unpublished sets MUST appear in the preview with muted chrome for viewers who can see the pack card. Never-live add-task-set ghost sets MUST appear in the preview **only for that set’s author**, with an up-to-date task count. Preview rows MUST NOT be individually clickable for navigation. Pale horizontal dividers MUST NOT appear between set-preview rows; a pale divider MUST appear above the action controls when actions are present. Hover on a clickable catalog card MUST change border and optional soft shadow only (no scale), consistent with task-set card hover product sense.

#### Scenario [SC-PACK-249]: Catalog card shows up to four sets and overflow

- **GIVEN** an in-catalog pack with six published task sets of known task counts
- **WHEN** a user views that pack’s catalog card
- **THEN** the card shows at most four set-preview rows with ordinal labels and counts
- **AND** shows an overflow caption indicating two more sets (product sense «ещё 2»)
- **AND** each visible row has a leading document-style icon
- **AND** activating a set-preview row alone does not navigate away from the pack open affordance of the card

#### Scenario [SC-PACK-250]: Catalog card outline actions and whole-card open

- **GIVEN** a catalog pack card where Edit and staff soft-unpublish apply
- **WHEN** the user views the card
- **THEN** Edit and «Снять с публикации» appear as stacked full-width outline buttons with leading icons
- **AND WHEN** the user activates the card body outside actions and the star
- **THEN** the client opens the pack per existing list routing (live or Edit)

#### Scenario [SC-PACK-251]: Catalog revise badge and uppercase title match mock product sense

- **GIVEN** a catalog pack whose list status is needs revision («ДОРАБОТАТЬ»)
- **WHEN** the user views the card in light or dark theme
- **THEN** the status badge uses a red outline with red label text (not a loud solid Quasar warning fill)
- **AND** the badge label is the short uppercase product sense «ДОРАБОТАТЬ» (not the long list status «Нужна доработка»)
- **AND** the pack title is shown in uppercase

### Requirement: Packs list returns lightweight task-set preview

The unified packs list HTTP response (`GET /api/content/packs` / `listCatalog`) MUST include, for each listed pack, a lightweight **`taskSetsPreview`** array sufficient to render the catalog card without loading the full pack revision (no answer cards, no per-task slots). The server MUST return the **complete** preview for that pack (all sets in order); the client MAY show at most four rows plus an overflow caption. Each preview entry MUST include a stable set `id`, a 1-based `ordinal` equal to the entry’s index in `taskSetsPreview` order (same product sense as live «Набор заданий #{n}»), a `taskCount`, and `inCatalog` (soft-unpublished when false). For the calling user who is the author of a never-live add-task-set ghost (same retention rules as live: pending, needs_revision, or cancelled cycle), that ghost MUST be included in the preview with `neverLive: true` and an up-to-date task count; other callers MUST NOT receive foreign never-live ghosts. Soft-unpublished sets MUST be included in the preview for callers who already receive the pack row. Preview sets MUST come from the same revision already used for that list row’s title/description. The preview MUST NOT require N+1 full live-pack fetches on the client.

#### Scenario [SC-PACK-252]: List catalog includes lightweight set previews

- **GIVEN** an authenticated user and an in-catalog pack with two published task sets
- **WHEN** the client requests the unified packs list
- **THEN** that pack’s payload includes `taskSetsPreview` with two entries
- **AND** each entry has `id`, 1-based `ordinal`, `taskCount`, and `inCatalog`
- **AND** the payload does not require full answer-card or slot graphs for those sets

#### Scenario [SC-PACK-253]: Preview includes soft-unpublished and author never-live

- **GIVEN** pack P with one published set, one soft-unpublished set, and a never-live add-task-set ghost owned by user A (same visibility as live, including retained cancelled cycles)
- **WHEN** user A requests the unified packs list
- **THEN** P’s `taskSetsPreview` includes the published set, the soft-unpublished set, and A’s never-live ghost with `neverLive: true` and current task count
- **AND** ordinals are 1-based in preview order (soft-unpub and ghost stay in sequence; not renumbered to published-only)
- **AND WHEN** another non-staff user who can see P requests the list
- **THEN** P’s preview includes the published and soft-unpublished sets
- **AND** MUST NOT include A’s never-live ghost
