# content/packs — delta: guest filter chrome, СНЯТО list/detail, add-task-set gate, view-only task-set

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-286 | covered (ContentUnifiedList — guest sees **no** filter chrome) |
| SC-PACK-165 | covered (ContentPackAcl — reject beyond login gate; server mocha reject unchanged) |
| SC-PACK-287 | covered (ContentPackAcl — guest add-task-set CTA + login gate) |
| SC-PACK-288 | covered (ContentPackTasksCsvHide — guest view-only task-set; no auth gate) |
| SC-PACK-289 | covered (ContentUnifiedList — non-staff packs list hides soft-unpublished) |
| SC-PACK-290 | covered (ContentPackAcl — unverified add-task-set CTA + verify gate) |
| SC-PACK-291 | covered (ContentPackTasksCsvHide — unverified view-only task-set; no verify gate) |
| SC-PACK-132 | covered (ContentFollowUp4 — non-staff omit soft-unpublished task-set cards; staff keep СНЯТО) |
| SC-PACK-292 | covered (ContentFollowUp4 — guest omits soft-unpublished task-set card on live) |

Related: `content/maps` (same guest filter / СНЯТО list); `ui/branding` (list page chrome + translucent underlay); `game/presence` (HUD translucent). Create/add-task-set gates unchanged on server. Soft-unpublish **storage** rules — SC-PACK-120… (unchanged); this delta **revises** live task-set **visibility** for non-staff (SC-PACK-132). `listCatalog` already omits soft-unpublished packs for non-staff; client list omit remains defense-in-depth. Guest view-only (SC-PACK-288) reuses `getLivePack` — **no** new read endpoint.

## MODIFIED Requirements

### Requirement: Packs list filters

The packs list MUST support filters: **all** (default); **on moderation** (caller’s own open pending or needs_revision requests — packs and add-task-set as applicable); **drafts** (caller’s never-published packs without an open request, or equivalently never-published not yet submitted); **mine** (packs where the caller is `createdBy` OR has contributed at least one task set on that pack); **favorites** (packs the caller has starred). Guests and anonymous users MUST only see the public catalog subset. For anonymous guests, the client MUST **hide the entire filter chrome** — including the **all** control — and MUST NOT leave identity filters visible as disabled controls. Staff soft-unpublished visibility rules MUST still apply when staff use filters.

#### Scenario [SC-PACK-151]: Filter on moderation shows only own open items

- **GIVEN** author A has pack P pending and user B has pack Q pending
- **WHEN** A applies the on-moderation filter
- **THEN** A sees P and MUST NOT see Q solely via that filter

#### Scenario [SC-PACK-152]: Filter mine includes task-set contributor

- **GIVEN** verified user U contributed an approved or pending task set on published pack P and U is not `createdBy` of P
- **WHEN** U applies the mine filter
- **THEN** P appears for U

#### Scenario [SC-PACK-153]: Guest cannot use favorites filter meaningfully

- **GIVEN** an anonymous guest on the packs list
- **WHEN** the guest views the list
- **THEN** only in-catalog packs are shown
- **AND** starring / favorites filter MUST NOT grant a personal favorites set
- **AND** the favorites filter control MUST NOT be visible

#### Scenario [SC-PACK-286]: Guest sees no filter chrome

- **GIVEN** an anonymous guest on the packs list
- **WHEN** the list page renders
- **THEN** no filter controls are shown (including **all** / «Все»)
- **AND** the filter row chrome itself is absent (not an empty button row)
- **AND** on-moderation, drafts, mine, and favorites filter controls are not shown
- **AND** only in-catalog packs are listed (effective filter remains **all**)

### Requirement: Add-task-set affordance beside tasks section on live

For a verified non-staff user eligible to add a task set viewing a published **in-catalog** pack’s live page, the primary control to start adding a new task set MUST appear **beside the «Задания» / task-sets section heading**, not as a header-only action and not as a row action in the collection list. The collection list MUST NOT show a dedicated add-task-set icon/button per row (trash and staff Edit remain as applicable). Staff add new task sets through the full Edit surface (button beside the task-sets heading in the editor), not via the non-staff add-task-set live control. An **anonymous guest** or **unverified** (email not confirmed) user on an in-catalog non-blocked live pack MUST still **see** the same «Добавить набор заданий» control beside the tasks section; activating it MUST open the existing login (anonymous) or verify-email (unverified) gate (same product sense as create-pack on the catalog list), MUST NOT navigate into the add-task-set editor, and MUST NOT create a task set.

#### Scenario [SC-PACK-117]: Live add-task-set control sits by tasks section

- **GIVEN** published pack P in verified non-staff user U’s collection
- **WHEN** U opens the live pack page for P
- **THEN** U sees «Добавить набор заданий» next to the tasks section heading
- **AND** U does not rely on a collection-list row add-task-set button

#### Scenario [SC-PACK-118]: Collection list has no add-task-set row button

- **GIVEN** an authenticated verified non-staff user on the collection list with published pack P
- **WHEN** the list renders
- **THEN** the row for P MUST NOT show an add-task-set action icon
- **AND** trash / remove-from-collection may still appear
- **AND** staff MAY still see Edit on rows where applicable

#### Scenario [SC-PACK-287]: Guest sees add-task-set and gets login gate

- **GIVEN** an anonymous guest on the live page of in-catalog non-blocked pack P
- **WHEN** the tasks section renders
- **THEN** «Добавить набор заданий» is visible beside the section heading
- **AND WHEN** the guest activates that control
- **THEN** a login/authorization required dialog is shown (same product sense as create-pack gate)
- **AND** the client MUST NOT open the add-task-set editor for that activation

#### Scenario [SC-PACK-290]: Unverified sees add-task-set and gets verify gate

- **GIVEN** an authenticated unverified user (email not confirmed) on the live page of in-catalog non-blocked pack P
- **WHEN** the tasks section renders
- **THEN** «Добавить набор заданий» is visible beside the section heading
- **AND WHEN** the user activates that control
- **THEN** a verify-email required dialog is shown (same product sense as create-pack gate)
- **AND** the client MUST NOT open the add-task-set editor for that activation

### Requirement: Verified user may add task set without collection

After publish, any verified non-anonymous user MUST be able to create and submit a **new** task set for an in-catalog non-blocked pack without collection membership. Guests and unverified users MUST NOT successfully create or submit an add-task-set (server MUST reject anonymous / unverified as today). The client MUST show the add-task-set affordance to anonymous and unverified users so activation opens the login/verify gate (see add-task-set affordance requirement) — not as a path into the editor. Unverified users MUST be prompted to confirm email instead of starting add-task-set. Add-task-set moderation and approve rules otherwise remain as in the base capability (new set only; submit through queue).

#### Scenario [SC-PACK-164]: Verified non-collector submits add-task-set

- **GIVEN** in-catalog pack P and verified user U with no collection membership concept required
- **WHEN** U creates and submits a new task set for P
- **THEN** an open task-set moderation request exists
- **AND** the system does not require collection membership

#### Scenario [SC-PACK-165]: Guest cannot add-task-set

- **GIVEN** an anonymous guest and in-catalog pack P
- **WHEN** the guest attempts to start or submit add-task-set beyond the login gate
- **THEN** the system rejects creating/submitting the task set
- **AND** the client login gate remains the primary UX instead of a silent failure

## ADDED Requirements

### Requirement: Unified packs list hides soft-unpublished for non-staff

On the unified packs catalog list, soft-unpublished packs (`hasLive` and not in catalog) MUST be shown **only to staff**. Non-staff users (including anonymous and the pack creator after first publish) MUST NOT see those packs as list cards. The server list already omits them for non-staff; the client MUST still omit any such row defensively if present in the payload. Staff MUST retain soft-unpublished chrome (short «СНЯТО» / muted) and unpublish/republish actions as already specified. Collection soft-unpublished rows and live deep-link reject rules are unchanged.

#### Scenario [SC-PACK-289]: Non-staff packs list omits soft-unpublished cards

- **GIVEN** soft-unpublished pack P (`hasLive` and not in catalog) and a non-staff authenticated user U (including anonymous)
- **WHEN** U opens the unified packs catalog list
- **THEN** P MUST NOT appear as a list card for U
- **AND** staff MUST still see P with soft-unpublished chrome when applicable

### Requirement: Guest may view published task set read-only

An anonymous guest or unverified user MUST be able to open a **published in-catalog** task set on an in-catalog non-blocked pack for **view-only** inspection (questions and slots visible; no compose, CSV write, delete, or submit). The client MUST NOT block that live drill-in (`query.view=live` / equivalent) with the login or verify-email gate — today `ContentPackTasksPage.checkGate()` would otherwise block both; live boot MUST skip that gate and load `liveViewMode`. Auth gates remain for create / add-task-set / edit / mutate. Soft-unpublished task sets MUST NOT appear in the live list for non-staff (see staff soft-unpublish task set requirement) and MUST remain non-enterable if reached by deep link.

#### Scenario [SC-PACK-288]: Guest opens published task set view-only

- **GIVEN** an anonymous guest on live pack P with published in-catalog task set S
- **WHEN** the guest activates S from the task-set list
- **THEN** the client opens the task-set surface in view-only mode
- **AND** a login/authorization dialog MUST NOT block that open
- **AND** compose / add-question / destructive controls MUST NOT be available

#### Scenario [SC-PACK-291]: Unverified opens published task set view-only

- **GIVEN** an authenticated unverified user on live pack P with published in-catalog task set S
- **WHEN** the user activates S from the task-set list
- **THEN** the client opens the task-set surface in view-only mode
- **AND** a verify-email dialog MUST NOT block that open
- **AND** compose / add-question / destructive controls MUST NOT be available

## MODIFIED Requirements

### Requirement: Staff soft-unpublish and republish task set

Moderator and admin MUST be able to soft-unpublish an entire **task set** on a live pack and to republish it without a new moderation request. Soft-unpublish of a task set MUST NOT remove individual questions as a separate product action. Soft-unpublish MUST keep the set stored. Soft-unpublished task sets MUST be shown **only to staff** on the live pack task-set list (and editor/staff surfaces as already specified): non-staff viewers (including anonymous and the pack creator after first publish) MUST NOT see a card for a soft-unpublished set — same product sense as soft-unpublished packs/maps on catalog lists. Staff MUST see the set card as muted with a short unpublished status badge (product sense «СНЯТО»), MUST NOT treat it as enterable for non-staff deep links, MUST still be able to Edit the soft-unpublished set (staff-save / lock session), and MUST have republish on the **task-set card** (editor and live list) and **inside** the task-set page. Staff MUST confirm before unpublishing a set; republish MUST NOT require confirmation. The system MUST reject unpublishing a task set when it is the **only** published task set on the pack. Soft-unpublish of individual tasks/questions MUST NOT be offered. Staff moderation queue MUST NOT expose task-set unpublish controls. Hard-delete of published or soft-unpublished task sets is out of scope for this requirement. On **task-set summary cards**, soft-unpublish and republish action labels MUST use short product copy («Снять» / «Вернуть» or equivalent i18n). Pack/catalog soft-unpublish copy and confirmation dialog wording MAY remain longer («Снять с публикации» / set confirm titles) and MUST NOT be required to match the card short labels. Display ordinals for visible sets MAY renumber among sets shown to that viewer (staff sees soft-unpublished in sequence; non-staff only published).

#### Scenario [SC-PACK-131]: Staff cannot unpublish the only published task set

- **GIVEN** live pack P with exactly one published task set S and an authenticated staff user
- **WHEN** the staff user attempts to soft-unpublish S
- **THEN** the system rejects the attempt (or the client disables the control)
- **AND** S remains published

#### Scenario [SC-PACK-132]: Soft-unpublished task set is gray and non-enterable

- **GIVEN** live pack P with published task set S1 and soft-unpublished task set S2
- **WHEN** a non-staff viewer (including anonymous) opens live P
- **THEN** S2 MUST NOT appear as a task-set card for that viewer (staff-only visibility — same product sense as soft-unpublished packs/maps lists)
- **AND** S1 remains visible and enterable per published rules
- **AND WHEN** staff opens live P
- **THEN** S2’s card is muted with a short unpublished badge (e.g. «СНЯТО»)
- **AND** activating S2 for non-staff MUST NOT open the questions view (deep link / staff surfaces unchanged for staff Edit)
- **AND** staff MAY still Edit S2 and choose republish on the card or inside the task-set page without confirmation
- **AND WHEN** staff chooses unpublish on a published set that is not the last published set
- **THEN** the client asks for confirmation before the API call

#### Scenario [SC-PACK-292]: Guest does not see soft-unpublished task-set card

- **GIVEN** an anonymous guest on live pack P with soft-unpublished task set S2 and published set S1
- **WHEN** the tasks section renders
- **THEN** S2 MUST NOT appear in the task-set grid
- **AND** S1 remains listed
