# content/packs — delta (simplify-content-pack-editing)

Суперседит co-edit на personal drafts / dual post-publish submit / foreign-pending. Канон: `openspec/specs/content/packs/spec.md`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-100 | pending |
| SC-PACK-101 | pending |
| SC-PACK-102 | pending |
| SC-PACK-103 | pending |
| SC-PACK-104 | pending |
| SC-PACK-105 | pending |
| SC-PACK-106 | pending |
| SC-PACK-107 | pending |
| SC-PACK-108 | pending |
| SC-PACK-109 | pending |
| SC-PACK-110 | pending |
| SC-PACK-111 | pending |
| SC-PACK-112 | pending |
| SC-PACK-113 | pending |
| SC-PACK-114 | pending |
| SC-PACK-115 | pending |
| SC-PACK-116 | pending |
| SC-PACK-117 | pending |
| SC-PACK-118 | pending |
| SC-PACK-119 | pending |
| SC-PACK-120 | pending |
| SC-PACK-121 | pending |
| SC-PACK-122 | pending |
| SC-PACK-123 | pending |
| SC-PACK-124 | pending |
| SC-PACK-125 | pending |
| SC-PACK-126 | pending |
| SC-PACK-127 | pending |
| SC-PACK-128 | pending |
| SC-PACK-129 | pending |
| SC-PACK-130 | pending |
| SC-PACK-131 | pending |
| SC-PACK-132 | pending |
| SC-PACK-133 | pending |
| SC-PACK-10 | pending |
| SC-PACK-11 | pending |
| SC-PACK-12 | pending |
| SC-PACK-53 | pending |
| SC-PACK-54 | pending |
| SC-PACK-61 | pending |
| SC-PACK-62 | pending |
| SC-PACK-63 | pending |
| SC-PACK-64 | pending |
| SC-PACK-65 | pending |
| SC-PACK-66 | pending |

## ADDED Requirements

### Requirement: Single working copy until first catalog publish

Until a pack has an approved live (catalog) answers revision, the pack MUST have exactly one mutable working copy owned by the **creator**. Other non-staff users MUST NOT obtain a separate personal draft or edit that pack’s cards or task sets. The system MUST NOT maintain concurrent per-user drafts for the same pack.

#### Scenario [SC-PACK-100]: Creator edits working copy before publish

- **GIVEN** verified user A created unpublished pack P
- **WHEN** A edits cards and task sets on P
- **THEN** changes apply to P’s single working copy
- **AND** no other non-staff user can edit P’s content

#### Scenario [SC-PACK-101]: Non-creator cannot open a personal draft

- **GIVEN** unpublished pack P created by A
- **WHEN** verified non-staff user B attempts to edit cards or tasks of P
- **THEN** the system rejects the attempt

### Requirement: First submit includes cards and at least one task set

The creator MUST submit the working copy for moderation as **one** pack submission that includes answer cards and at least one task set meeting existing minima (≥2 cards; each submitted task set ≥2 tasks). Staff MUST NOT approve a first publish when the submission has zero task sets. Hard **reject** MUST NOT be offered; staff MAY mark **needs revision** («Доработать»), after which the creator MAY amend the same working copy and resubmit. The creator MUST be able to **cancel** their own open first-publish request.

#### Scenario [SC-PACK-102]: Submit without task set is rejected

- **GIVEN** unpublished pack P with cards but no task set
- **WHEN** creator submits P for moderation
- **THEN** the system rejects the submit

#### Scenario [SC-PACK-103]: Staff single approve publishes pack

- **GIVEN** open first-publish request for P with valid cards and ≥1 task set
- **WHEN** staff chooses a single Approve action for that request
- **THEN** P enters the public catalog with that content live
- **AND** no separate answers-only vs tasks-only approve is required for first publish

#### Scenario [SC-PACK-104]: Needs revision keeps working copy editable by creator

- **GIVEN** open first-publish request for P
- **WHEN** staff marks needs revision
- **THEN** P is not in the catalog
- **AND** creator A MAY edit the same working copy and resubmit
- **AND** the request is not a terminal hard-reject

#### Scenario [SC-PACK-105]: Creator cancels own request

- **GIVEN** open first-publish or add-task-set request authored by A
- **WHEN** A cancels that request
- **THEN** the request is closed without publishing
- **AND** A MAY later submit again from the retained working copy or add-task-set payload

### Requirement: After catalog publish non-staff cannot edit live content

Once pack P has live catalog content, non-staff users (including the creator) MUST NOT edit existing answer cards or existing task sets. Non-staff MUST NOT see cards or task lists in an **editing** surface for P; they MAY only add a **new** task set. The public live view MAY still show cards and tasks read-only.

#### Scenario [SC-PACK-106]: Creator has no Edit of cards after publish

- **GIVEN** published pack P created by A
- **WHEN** A views collection or live affordances for P
- **THEN** Edit of cards / existing task sets is unavailable to A
- **AND** A MAY still open a flow to add a new task set if A has P in collection and is verified

#### Scenario [SC-PACK-107]: Post-publish editor surfaces are add-only for non-staff

- **GIVEN** published pack P and verified non-staff user U with P in collection
- **WHEN** U opens the post-publish contribution UI for P
- **THEN** U does not see editable lists of existing cards or existing task sets
- **AND** U can start adding a new task set only

### Requirement: Anyone verified with pack in collection may submit a new task set

After publish, a verified non-anonymous user who has pack P in their collection MUST be able to create and submit a **new** task set for moderation. New tasks MUST reference only existing live answer cards. Editing that new set after it becomes live MUST be staff-only. Staff approval of an add-task-set request MUST publish **only** that task set into live (single Approve for that request). Needs revision and author Cancel apply as for first publish.

#### Scenario [SC-PACK-108]: Verified collector submits new task set

- **GIVEN** published pack P in verified user U’s collection
- **WHEN** U submits a new task set using only live card ids
- **THEN** an open moderation request for that task set exists
- **AND** live content is unchanged until staff Approve

#### Scenario [SC-PACK-109]: New task set cannot introduce new cards

- **GIVEN** published pack P
- **WHEN** a non-staff user attempts to add cards as part of a new task set submit
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-110]: Staff approve adds only the new task set

- **GIVEN** open add-task-set request for set S on published pack P
- **WHEN** staff Approves that request
- **THEN** live P includes S
- **AND** existing live cards and other task sets remain as they were

### Requirement: Staff may edit any pack live without moderation under exclusive lock

Moderator and admin MUST be able to open Edit on any pack (published or not) **without** requiring the pack in their collection. Staff content changes (including add card / add task set / edit / delete within policy) MUST apply **directly to live or to the single working copy** without entering the moderation queue. The client MUST NOT show staff a «submit for moderation» control for their own staff edits. While staff A holds an edit lock on pack P, staff B’s Edit attempt MUST fail with an error. The lock MUST release when A leaves the **entire** edit session (leaves Edit for live/collection/catalog/another pack) or after a short timeout — NOT when navigating between the cards editor and a task-set editor of the same pack. Saves of cards and of tasks during that session MUST use the staff direct-save path, not the creator working-copy path.

#### Scenario [SC-PACK-111]: Staff edits published pack without queue

- **GIVEN** published pack P and staff S
- **WHEN** S edits a card or task set on P and saves
- **THEN** the change is visible in the catalog live content
- **AND** no new moderation request is created for that save

#### Scenario [SC-PACK-112]: Staff Edit without collection membership

- **GIVEN** published pack P not in staff S’s collection
- **WHEN** S opens Edit on P
- **THEN** Edit is allowed

#### Scenario [SC-PACK-113]: Second staff Edit is rejected while locked

- **GIVEN** staff A holds the edit lock on pack P
- **WHEN** staff B attempts Edit on P
- **THEN** the system rejects B’s Edit with an error
- **AND** A retains the lock until release

#### Scenario [SC-PACK-114]: Creator delete unpublished remains

- **GIVEN** unpublished pack P created by A with no catalog live
- **WHEN** A hard-deletes P
- **THEN** P and its working copy and open requests are removed
- **AND** published packs remain non-deletable by the creator

#### Scenario [SC-PACK-115]: Staff saves task edits after opening from cards editor

- **GIVEN** published pack P and staff S holding an edit lock after opening Edit on cards
- **WHEN** S navigates to a task-set editor for P and saves a question change
- **THEN** the save succeeds via staff direct-save
- **AND** the client MUST NOT call the creator working-copy save path
- **AND** S MUST NOT see a hard error that a published pack cannot be edited via working copy

#### Scenario [SC-PACK-116]: Staff does not see author my-moderation nav

- **GIVEN** an authenticated moderator or admin
- **WHEN** the user views the content catalog or collection chrome
- **THEN** the «На модерации» (author my-moderation) control is not shown
- **AND** staff MAY still open the staff moderation queue control

### Requirement: Add-task-set affordance beside tasks section on live

For a verified non-staff collector viewing a published pack’s live page, the primary control to start adding a new task set MUST appear **beside the «Задания» / task-sets section heading**, not as a header-only action and not as a row action in the collection list. The collection list MUST NOT show a dedicated add-task-set icon/button per row (trash and staff Edit remain as applicable). Staff add new task sets through the full Edit surface (button beside the task-sets heading in the editor), not via the non-staff add-task-set live control.

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

### Requirement: Content editor hints must not shift layout height

On content pack editor surfaces (cards editor, task-set editor, add-task-set page), validation and save guidance that depends on form state (for example empty slots, submit minima, autosave/moderation hints) MUST NOT appear or disappear in a way that changes the page’s vertical layout height. Such guidance MUST use tooltips on disabled controls and/or always-reserved / always-visible static hint space.

#### Scenario [SC-PACK-119]: Slot hint does not jump page height

- **GIVEN** a user editing a task form with a non-empty question and no filled slot
- **WHEN** the client shows guidance that a slot answer is required
- **THEN** showing or hiding that guidance MUST NOT change the surrounding layout height
- **AND** a tooltip on the disabled save control is an acceptable form of the guidance

### Requirement: Staff soft-unpublish and republish

Moderator and admin MUST be able to **unpublish** a pack that currently appears in the public catalog and to **republish** it with a single action that restores catalog visibility **without** creating a new moderation request. Unpublish MUST be a soft-hide: live content remains stored; the pack MUST leave the **public** catalog. Staff MUST still see the pack in the **shared catalog list** with a clear «снято с публикации» state. Non-staff users MUST NOT open the live pack view or deep-link after unpublish. In a non-staff user’s collection, the row MUST appear disabled/gray with the label «Снято с публикации», MUST NOT navigate on row click, and MUST still allow remove-from-collection (trash + confirm). After the first catalog approve, the **creator** is treated like any other non-staff user for that pack (add-task-set only while in catalog; no working-copy Edit after soft-unpublish). Staff MUST be able to Edit a soft-unpublished pack under the exclusive lock (same staff-save path as published). Unpublish/republish MUST NOT be conflated with **block**. Staff unpublish/republish controls MUST appear in the **catalog**, the **collection** list, and **inside the pack** live view. Staff MUST confirm before unpublishing a pack. Republish of a pack MUST NOT require confirmation. Staff moderation queue MUST NOT expose pack unpublish/republish controls.

#### Scenario [SC-PACK-120]: Staff unpublish hides pack from public catalog

- **GIVEN** in-catalog pack P and staff S
- **WHEN** S chooses «Снять с публикации» for P (after confirmation)
- **THEN** non-staff callers MUST NOT see P in the public catalog
- **AND** staff callers MUST still see P in the shared catalog list with an unpublished/снято state
- **AND** P’s live content remains stored for staff access

#### Scenario [SC-PACK-121]: Non-staff collection row is gray and non-navigating

- **GIVEN** soft-unpublished pack P in non-staff user U’s collection
- **WHEN** U views the collection list
- **THEN** the row for P is visually disabled/gray
- **AND** shows the label «Снято с публикации» (or equivalent i18n)
- **AND** activating the row MUST NOT open live or editor
- **AND** U MAY still remove P from collection via trash + confirmation

#### Scenario [SC-PACK-122]: Non-staff cannot open soft-unpublished live

- **GIVEN** soft-unpublished pack P and non-staff user U
- **WHEN** U requests the live pack view or deep-link for P
- **THEN** the system rejects or shows a non-enterable «снято» state
- **AND** U MUST NOT receive editable or full live content

#### Scenario [SC-PACK-123]: Staff republish restores catalog without moderation

- **GIVEN** soft-unpublished pack P with stored live content and staff S
- **WHEN** S chooses «Опубликовать снова»
- **THEN** P appears in the public catalog with the same live content
- **AND** no new moderation request is created for that republish
- **AND** no confirmation dialog is required for republish

#### Scenario [SC-PACK-124]: Staff may Edit soft-unpublished pack

- **GIVEN** soft-unpublished pack P and staff S
- **WHEN** S opens Edit on P and saves
- **THEN** the save succeeds via staff direct-save under the exclusive lock
- **AND** non-staff still cannot enter or edit P

#### Scenario [SC-PACK-125]: Creator after first publish has no special unpublish rights

- **GIVEN** pack P created by A that was approved into the catalog and then soft-unpublished by staff
- **WHEN** A views collection or attempts Edit / live open for P
- **THEN** A is treated as any non-staff user (gray non-navigating collection row; no enter; no working-copy Edit)
- **AND** A MUST NOT unpublish or republish P

#### Scenario [SC-PACK-129]: Staff unpublish/republish in collection with confirm

- **GIVEN** staff S viewing the collection list with in-catalog pack P
- **WHEN** the collection list renders
- **THEN** S MUST see «Снять с публикации» (or republish when soft-unpublished) on that row
- **AND WHEN** S chooses unpublish
- **THEN** the client MUST ask for confirmation before calling the unpublish API
- **AND** cancelling the dialog MUST leave P in catalog
- **AND** staff moderation queue MUST NOT show pack unpublish/republish controls

### Requirement: Cascade yellow visible on task set in cards editor

After an answers cascade clears one or more slots, the client MUST highlight both the affected **task** (on the task-set editor page) and the containing **task set** (on the cards editor list) with the same yellow/warning outline while cascade gaps remain. Applying the highlight class without visible styles is insufficient.

#### Scenario [SC-PACK-126]: Task-set row outline visible after answer delete cascade

- **GIVEN** cards editor for pack P with task set S containing task T that referenced deleted/changed answer card A
- **AND** cascade emptied at least one slot on T
- **WHEN** the cards editor task-set list renders
- **THEN** the row for S MUST show a visible yellow/warning outline (same visual language as task cascade highlight)
- **AND** opening S MUST still show T highlighted while the gap remains

### Requirement: Answer slots visible on every question list row

Wherever the client lists tasks/questions for review or editing (staff moderation hub preview, add-task-set page question list, task-set editor list, live pack drill-in task list), each task row MUST show its answer slots (filled card content and/or empty). Staff MUST be able to see slot bindings without opening a separate nested tasks-only page.

#### Scenario [SC-PACK-127]: Staff hub and add-task-set lists show slots

- **GIVEN** a moderation preview or add-task-set page with tasks that have slots
- **WHEN** the questions list renders
- **THEN** each task row shows its answer slots (filled and/or empty)
- **AND** the same rule applies on staff request hub and on the author add-task-set list

### Requirement: Add-task-set author sees moderation status and thread

When the change author opens the add-task-set page for an open `task_set` request (including navigation from «На модерации»), the page MUST show the open request status (pending vs needs_revision) and the same moderation thread UX as the cards editor: existing messages readable, and reply allowed while the request is open (pending or needs_revision). Staff needs_revision comments MUST be visible without a separate obscure route.

#### Scenario [SC-PACK-128]: Author amend add-task-set sees needs_revision thread

- **GIVEN** author A has an open add-task-set request on pack P with status needs_revision and at least one staff message
- **WHEN** A opens the add-task-set page for P (including from «На модерации»)
- **THEN** the page shows a needs_revision (or equivalent) status
- **AND** shows the moderation thread including the staff comment(s)
- **AND** A MAY reply and amend/resubmit while the request remains open

### Requirement: Live pack shows task-set summary and drill-in

On the live pack view, the client MUST show answer cards and a **list of task-set rows** (not an inline expansion of all questions). Each published task-set row MUST show at least the task count and a difficulty breakdown (counts for difficulties 1, 2, and 3). Activating a published task-set row MUST navigate to a page that lists that set’s questions with answer slots. Soft-unpublished task-set rows follow the soft-unpublish task-set rules (gray, non-enterable for non-staff).

#### Scenario [SC-PACK-130]: Live pack drills into task set questions

- **GIVEN** in-catalog pack P with at least one published task set S that has tasks of mixed difficulties
- **WHEN** a viewer opens the live pack page for P
- **THEN** the page lists S as a summary row with task count and difficulty 1/2/3 counts
- **AND** MUST NOT expand S’s questions inline on that page
- **AND WHEN** the viewer activates the row for S
- **THEN** the client opens a questions view for S showing each task’s answer slots

### Requirement: Staff soft-unpublish and republish task set

Moderator and admin MUST be able to soft-unpublish an entire **task set** on a live pack and to republish it without a new moderation request. Soft-unpublish of a task set MUST NOT remove individual questions as a separate product action. Soft-unpublish MUST keep the set stored; non-staff viewers MUST see the set row as gray with «Снято с публикации» and MUST NOT enter the set. Staff MUST still be able to Edit the soft-unpublished set (staff-save / lock session) and MUST have republish on the **task-set row** (editor and live list) and **inside** the task-set page. Staff MUST confirm before unpublishing a set; republish MUST NOT require confirmation. The system MUST reject unpublishing a task set when it is the **only** published task set on the pack. Soft-unpublish of individual tasks/questions MUST NOT be offered. Staff moderation queue MUST NOT expose task-set unpublish controls. Hard-delete of published or soft-unpublished task sets is out of scope for this requirement.

#### Scenario [SC-PACK-131]: Staff cannot unpublish the only published task set

- **GIVEN** live pack P with exactly one published task set S and an authenticated staff user
- **WHEN** the staff user attempts to soft-unpublish S
- **THEN** the system rejects the attempt (or the client disables the control)
- **AND** S remains published

#### Scenario [SC-PACK-132]: Soft-unpublished task set is gray and non-enterable

- **GIVEN** live pack P with published task set S1 and soft-unpublished task set S2
- **WHEN** a non-staff viewer opens live P
- **THEN** S2’s row is gray with «Снято с публикации»
- **AND** activating S2 MUST NOT open the questions view
- **AND** staff MAY still Edit S2 and choose «Опубликовать снова» on the row or inside the task-set page without confirmation
- **AND WHEN** staff chooses «Снять с публикации» on a published set that is not the last published set
- **THEN** the client asks for confirmation before the API call

### Requirement: Add-task-set answer tiles match task editor chips

On the add-task-set page, the answer-card picker used to fill slots MUST use the same rounded chip visual language as the staff/creator task-set editor (not rectangular button tiles).

#### Scenario [SC-PACK-133]: Add-task-set answer tiles are rounded chips

- **GIVEN** author A on the add-task-set page with live answer cards available
- **WHEN** the answer tiles picker renders
- **THEN** each tile is a rounded chip consistent with the task-set editor answer picker
- **AND** MUST NOT use rectangular primary outline buttons for those tiles

## MODIFIED Requirements

### Requirement: Collection gates editing; anyone with session may collect approved packs

Only the **creator** MAY edit a **never-published** pack’s working copy (cards and task sets) and submit first publish, subject to verified non-anonymous identity. After the pack has been approved into the public catalog at least once, non-staff users (including the creator) MUST NOT enter full Edit of existing cards or task sets; while the pack is **in catalog**, verified collectors MAY only use the **add task set** flow. Soft-unpublished packs MUST NOT be enterable by non-staff. **Staff** (moderator/admin) MUST be able to Edit any pack (in-catalog or soft-unpublished) without collection membership, subject to the exclusive staff lock. Any authenticated user (including anonymous and unverified) MUST be able to view an approved **in-catalog** non-blocked pack in the public catalog and add it to their collection. The live pack response MUST report `inCollection`. Removing a pack from the user’s own collection MUST use a clear trash affordance and confirmation (including for soft-unpublished gray rows). Unauthenticated callers MUST be rejected for collection mutations.

#### Scenario [SC-PACK-10]: User without collection cannot edit

- **GIVEN** a verified non-staff user who does not have published pack P in their collection
- **WHEN** the user attempts to edit existing content or submit an add-task-set for P
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-11]: Guest may add an approved pack to collection

- **GIVEN** an authenticated anonymous user and an approved non-blocked **in-catalog** pack in the public catalog
- **WHEN** the user adds the pack to their collection
- **THEN** the pack is in that user’s collection
- **AND** the guest still cannot create or edit packs

#### Scenario [SC-PACK-12]: Unauthenticated collection add is rejected

- **GIVEN** no auth session
- **WHEN** a collection-add request is made
- **THEN** the system rejects the request

#### Scenario [SC-PACK-53]: Live Edit when pack is in collection and idle

- **GIVEN** published pack P and non-staff user U with P in collection and no open request that grants U an amend flow
- **WHEN** the live pack page for P renders
- **THEN** full Edit of existing content is unavailable to U
- **AND** staff MAY see Edit subject to the exclusive lock
- **AND** U MAY still use add-task-set if verified and in collection

#### Scenario [SC-PACK-54]: Remove from collection asks for confirmation

- **GIVEN** an authenticated user with pack P in their collection
- **WHEN** the user chooses remove-from-collection
- **THEN** the client asks for confirmation before calling the remove API
- **AND** cancelling the dialog leaves P in the collection

#### Scenario [SC-PACK-61]: Live Edit hidden when not in collection

- **GIVEN** an authenticated verified non-staff user viewing live pack P that is **not** in their collection
- **WHEN** the live pack page renders
- **THEN** the page MUST NOT show a primary Edit control for existing content

#### Scenario [SC-PACK-62]: Live Edit hidden for non-author while pending

- **GIVEN** published pack P in user B’s collection and an open add-task-set or first-publish request authored by A (A ≠ B)
- **WHEN** B opens the live pack page for P
- **THEN** full Edit of existing content remains unavailable to B

#### Scenario [SC-PACK-63]: Pending author still sees Live Edit

- **GIVEN** author A has an open needs-revision or pending request on pack P and P is in A’s collection
- **WHEN** A opens the live pack page or «На модерации» for P
- **THEN** A MAY continue amending and resubmitting that request’s payload
- **AND** if P is already published, A still MUST NOT fully edit existing live cards/sets outside add-task-set / amend-request flows

#### Scenario [SC-PACK-64]: Collect button reflects membership

- **GIVEN** an authenticated user who already has pack P in their collection
- **WHEN** the live pack page for P loads
- **THEN** the collect control shows an in-collection state and MUST NOT present a fresh “add” as if P were absent
- **AND** the live pack payload includes `inCollection: true` for that caller

#### Scenario [SC-PACK-65]: Collection trash icon and row click isolation

- **GIVEN** an authenticated user on the collection list with pack P that is in catalog (or never-published for creator)
- **WHEN** the list renders
- **THEN** remove-from-collection uses a trash/delete icon (not a minus-only glyph)
- **AND WHEN** the user activates remove
- **THEN** confirmation is required before the API call
- **AND WHEN** the user clicks the row (not the action icons)
- **THEN** they navigate to the live view if P is in catalog with live content (else creator editor when never-published and caller is creator)
- **AND** activating action icons MUST NOT navigate via the row link instead of the intended action
- **AND** soft-unpublished rows follow SC-PACK-121 (no navigate) instead of this navigate rule

#### Scenario [SC-PACK-66]: Collection Edit reaches editor when pack has live

- **GIVEN** a verified user with published pack P in their collection
- **WHEN** the user activates the collection contribution path for P (navigate to live, then add-task-set from the tasks section; not full Edit of live cards)
- **THEN** the client opens the add-task-set flow
- **AND** MUST NOT open a full cards/tasks editor of existing live content for non-staff
- **AND** the collection row itself MUST NOT be the primary add-task-set control

## REMOVED Requirements

### Requirement: Dual pending locks and races

**Reason:** Product removes concurrent personal drafts and dual answers|tasks submit races among collection members. First publish and add-task-set use single-request flows with needs-revision / Cancel instead.

**Migration:** Delete dual dirty/pending lock rules and scenarios SC-PACK-15/16/17/42/43/57/58 behavior; replace with ADDED first-submit / add-task-set / staff-lock requirements. Drop per-user `content_user_drafts` and any retain-draft-on-losing-submit behavior.
