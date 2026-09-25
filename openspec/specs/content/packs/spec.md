# content/packs Specification

## Purpose

UGC-наборы (**набор карточек** + задания): одна рабочая копия до первой публикации в каталог, после approve — freeze live для non-staff (только add-task-set), staff Edit под exclusive lock без очереди, soft-unpublish пака и набора заданий с republish, единый submit без personal drafts и dual answers|tasks. Коллекция, публичный каталог, cascade слотов, жёлтая подсветка, слоты на строке задания, очередь staff и «На модерации»; задел difficulty 1–3 под peek. Без привязки к tourist-room и без замены peek-stub.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-01 | covered |
| SC-PACK-02 | covered |
| SC-PACK-03 | covered |
| SC-PACK-04 | covered |
| SC-PACK-05 | covered |
| SC-PACK-06 | covered |
| SC-PACK-07 | covered |
| SC-PACK-78 | covered |
| SC-PACK-79 | covered |
| SC-PACK-80 | covered |
| SC-PACK-81 | covered |
| SC-PACK-82 | covered |
| SC-PACK-83 | covered |
| SC-PACK-84 | covered |
| SC-PACK-08 | covered |
| SC-PACK-09 | covered |
| SC-PACK-10 | covered (modified ACL) |
| SC-PACK-11 | covered (modified ACL) |
| SC-PACK-12 | covered (modified ACL) |
| SC-PACK-13 | covered |
| SC-PACK-14 | covered |
| SC-PACK-15 | removed (superseded by SC-PACK-100…) |
| SC-PACK-16 | removed (superseded by SC-PACK-100…) |
| SC-PACK-17 | removed (superseded by SC-PACK-100…) |
| SC-PACK-18 | covered |
| SC-PACK-19 | covered |
| SC-PACK-20 | covered |
| SC-PACK-21 | covered |
| SC-PACK-22 | covered |
| SC-PACK-23 | covered |
| SC-PACK-24 | covered |
| SC-PACK-25 | covered |
| SC-PACK-26 | covered |
| SC-PACK-27 | covered |
| SC-PACK-28 | covered |
| SC-PACK-29 | covered |
| SC-PACK-30 | covered |
| SC-PACK-31 | covered |
| SC-PACK-32 | covered |
| SC-PACK-33 | covered (unchanged; peek stub not touched) |
| SC-PACK-34 | covered |
| SC-PACK-35 | covered |
| SC-PACK-36 | covered |
| SC-PACK-37 | covered |
| SC-PACK-39 | covered |
| SC-PACK-40 | covered |
| SC-PACK-41 | covered |
| SC-PACK-42 | removed (superseded by SC-PACK-100…) |
| SC-PACK-43 | removed (superseded by SC-PACK-100…) |
| SC-PACK-44 | covered |
| SC-PACK-45 | covered |
| SC-PACK-46 | covered |
| SC-PACK-47 | covered |
| SC-PACK-48 | covered |
| SC-PACK-49 | covered |
| SC-PACK-50 | covered |
| SC-PACK-51 | covered |
| SC-PACK-52 | covered |
| SC-PACK-53 | covered (client live Edit ACL) |
| SC-PACK-54 | covered (client collection trash confirm) |
| SC-PACK-55 | covered |
| SC-PACK-56 | covered |
| SC-PACK-57 | removed (superseded by SC-PACK-100…) |
| SC-PACK-58 | removed (superseded by SC-PACK-100…) |
| SC-PACK-59 | covered |
| SC-PACK-60 | covered |
| SC-PACK-61 | covered (client live Edit hidden) |
| SC-PACK-62 | covered (client pending non-author) |
| SC-PACK-63 | covered (client pending author amend) |
| SC-PACK-64 | covered (client inCollection) |
| SC-PACK-65 | covered (client trash + soft-unpublish row) |
| SC-PACK-66 | covered (client collection → add-task-set) |
| SC-PACK-70 | covered |
| SC-PACK-71 | covered |
| SC-PACK-72 | covered |
| SC-PACK-73 | covered |
| SC-PACK-74 | covered |
| SC-PACK-75 | covered |
| SC-PACK-76 | covered |
| SC-PACK-77 | covered |
| SC-PACK-100 | covered (server mocha + client) |
| SC-PACK-101 | covered (server mocha) |
| SC-PACK-102 | covered (server mocha) |
| SC-PACK-103 | covered (server mocha) |
| SC-PACK-104 | covered (server mocha) |
| SC-PACK-105 | covered (server mocha) |
| SC-PACK-106 | covered (client UX + store) |
| SC-PACK-107 | covered (client UX) |
| SC-PACK-108 | covered (server mocha) |
| SC-PACK-109 | covered (server mocha) |
| SC-PACK-110 | covered (server mocha) |
| SC-PACK-111 | covered (server mocha) |
| SC-PACK-112 | covered (server mocha) |
| SC-PACK-113 | covered (server mocha) |
| SC-PACK-114 | covered (server mocha) |
| SC-PACK-115 | covered (client vitest) |
| SC-PACK-116 | covered (client vitest) |
| SC-PACK-117 | covered (client vitest) |
| SC-PACK-118 | covered (client vitest) |
| SC-PACK-119 | covered (client vitest) |
| SC-PACK-120 | covered (server mocha) |
| SC-PACK-121 | covered (client vitest) |
| SC-PACK-122 | covered (server mocha) |
| SC-PACK-123 | covered (server mocha) |
| SC-PACK-124 | covered (server mocha) |
| SC-PACK-125 | covered (client vitest) |
| SC-PACK-126 | covered (client vitest) |
| SC-PACK-127 | covered (client vitest) |
| SC-PACK-128 | covered (client vitest) |
| SC-PACK-129 | covered (client vitest) |
| SC-PACK-130 | covered (client vitest) |
| SC-PACK-131 | covered (server mocha + client) |
| SC-PACK-132 | covered (client vitest) |
| SC-PACK-133 | covered (client vitest) |
| SC-PACK-134 | covered (server + client vitest) |
| SC-PACK-135 | covered (server mocha + client vitest) |
| SC-PACK-136 | covered (client vitest) |
| SC-PACK-137 | covered (server mocha) |
| SC-PACK-138 | covered (server mocha) |
| SC-PACK-139 | covered (client vitest) |
| SC-PACK-140 | covered (server mocha) |
| SC-PACK-141 | covered (server mocha) |
| SC-PACK-142 | removed (superseded by SC-PACK-170 / unified list) |
| SC-PACK-143 | removed (superseded by SC-PACK-170 / unified list) |
| SC-PACK-144 | removed (superseded by SC-PACK-170 / unified list) |
| SC-PACK-145 | removed (superseded by SC-PACK-170 / unified list) |
| SC-PACK-146 | removed (superseded by SC-PACK-170 / unified list) |
| SC-PACK-147 | removed (superseded by SC-PACK-170 / unified list) |
| SC-PACK-148 | covered (mocha) |
| SC-PACK-149 | covered (mocha) |
| SC-PACK-150 | covered (mocha) |
| SC-PACK-151 | covered (vitest) |
| SC-PACK-152 | covered (vitest) |
| SC-PACK-153 | covered (vitest) |
| SC-PACK-154 | covered (mocha/vitest) |
| SC-PACK-155 | covered (mocha/vitest) |
| SC-PACK-156 | covered (mocha) |
| SC-PACK-157 | covered (mocha) |
| SC-PACK-158 | covered (mocha) |
| SC-PACK-159 | covered (mocha) |
| SC-PACK-160 | covered (mocha) |
| SC-PACK-161 | covered (mocha) |
| SC-PACK-162 | covered (mocha) |
| SC-PACK-163 | covered (mocha) |
| SC-PACK-164 | covered (mocha) |
| SC-PACK-165 | covered (mocha) |
| SC-PACK-166 | covered (vitest) |
| SC-PACK-170 | covered (mocha) |
| SC-PACK-171 | covered (mocha/vitest) |
| SC-PACK-172 | covered (mocha/vitest) |
| SC-PACK-173 | covered (mocha/vitest) |
| SC-PACK-174 | covered (mocha/vitest) |
| SC-PACK-175 | covered (mocha/vitest) |
| SC-PACK-176 | covered (mocha/vitest) |
| SC-PACK-177 | covered (mocha/vitest) |
| SC-PACK-178 | covered (mocha/vitest) |
| SC-PACK-179 | covered (mocha/vitest) — strengthen: add-task-set GET |
| SC-PACK-180 | covered (vitest) |
| SC-PACK-181 | covered (vitest) |
| SC-PACK-182 | covered (vitest) |
| SC-PACK-183 | covered (vitest) |
| SC-PACK-184 | covered (vitest) |
| SC-PACK-185 | covered (vitest) |
| SC-PACK-186 | covered (vitest) |
| SC-PACK-187 | covered (mocha) |
| SC-PACK-188 | covered (mocha/vitest) |
| SC-PACK-189 | covered (mocha/vitest) |
| SC-PACK-190 | covered (vitest) |
| SC-PACK-191 | covered (vitest) |
| SC-PACK-192 | covered (vitest) |
| SC-PACK-193 | covered (vitest) — strengthen: offset zone |
| SC-PACK-194 | covered (vitest) |
| SC-PACK-195 | covered (vitest) |
| SC-PACK-196 | covered (mocha) |
| SC-PACK-197 | covered (mocha) |

## Requirements

### Requirement: Verified registered user may create a pack

A non-anonymous user with a valid JWT and verified email MUST be able to create a content pack with a non-empty title and a description (description MAY be empty). On create the pack MUST enter the creator’s collection and MUST start as not publicly listed until answers are approved. After create the client MUST open the **cards pack** editing surface («Набор карточек»). Anonymous guests and unverified email/password users MUST NOT create packs; the client MUST prompt them to sign in or confirm email instead.

#### Scenario [SC-PACK-01]: Verified user creates a pack into own collection

- **GIVEN** an authenticated non-anonymous user with verified email
- **WHEN** the user creates a pack with a non-empty title
- **THEN** the pack is stored
- **AND** the pack appears in that user’s collection
- **AND** the pack is not listed in the public catalog

#### Scenario [SC-PACK-02]: Guest cannot create a pack

- **GIVEN** an authenticated anonymous user
- **WHEN** the user attempts to create a pack
- **THEN** the system rejects the create
- **AND** the client offers sign-in / email confirmation instead of a silent failure

#### Scenario [SC-PACK-03]: Unverified email cannot create a pack

- **GIVEN** an authenticated email/password user whose email is not verified
- **WHEN** the user attempts to create a pack
- **THEN** the system rejects the create
- **AND** the client offers email confirmation

### Requirement: Pack holds answer cards and task sets

A pack MUST contain answer cards and may contain one or more task sets. Each answer card MUST have textual content and MAY have a textual description. Each task MUST have a textual question, a difficulty of exactly `1`, `2`, or `3`, and one or more answer slots that reference answer cards from the same pack. Task sets MUST be labeled by the contributing user’s display identity (author and, when applicable, co-author labels are display-only and MUST NOT grant extra permissions beyond collection membership). Creating or editing tasks MUST require that the pack already has at least one answer card in the editor’s draft. Any collection member who satisfies create/edit identity rules MUST be allowed to edit answer cards and tasks (no per-author ACL beyond collection + verify).

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

### Requirement: Answer-slot editing and separate submit rules

When composing a task, the editor MUST support adding an empty slot, removing a slot (minimum one slot remains), filling the next empty slot by selecting an answer card tile, and clearing a filled slot by selecting that slot. Changing an answer card’s **content** that is referenced by task slots MUST clear those slot references. Changing only an answer card’s **description** MUST NOT clear slots. Deleting an answer card that is referenced MUST clear those slots; deleting an unreferenced card MUST NOT alter task structure. Persisting a draft that only applies such cascade clears MUST succeed and MUST NOT be rejected as a tasks edit under the dirty-answers lock. Deleting an answer card MUST NOT by itself block answers submit. Tasks submit MUST be rejected while any submitted task has an empty slot or while fewer than two tasks exist. Answers submit MUST be rejected while fewer than two answer cards exist.

#### Scenario [SC-PACK-07]: Changing a referenced answer card clears dependent slots

- **GIVEN** a task whose slots reference answer card A
- **WHEN** the editor changes the content of answer card A
- **THEN** those slots become empty
- **AND** the draft save succeeds
- **AND** answers become dirty and answers submit remains available when card minima are met
- **AND** tasks submit remains blocked until the slots are filled again
- **AND** answers submit is not blocked solely because those slots are empty

#### Scenario [SC-PACK-78]: Cascade draft save is not rejected as tasks-edit under dirty answers

- **GIVEN** a pack with no open answers request and tasks that reference answer card A
- **WHEN** the editor changes A’s content or deletes A so that slots clear as a cascade
- **THEN** the draft persist succeeds
- **AND** the system does not return the dirty-answers tasks-lock error for that cascade-only change
- **AND** answers submit is enabled when at least two answer cards remain

#### Scenario [SC-PACK-79]: Description-only change does not clear slots

- **GIVEN** a task whose slots reference answer card A
- **WHEN** the editor changes only A’s description
- **THEN** those slots stay filled
- **AND** answers become dirty so answers submit is available when minima are met

#### Scenario [SC-PACK-80]: Deleting an unused answer card leaves tasks unchanged

- **GIVEN** answer card A is not referenced by any task slot
- **WHEN** the editor deletes A
- **THEN** task sets and slots are unchanged
- **AND** answers become dirty when the deletion differs from the last answers submit

#### Scenario [SC-PACK-08]: Answers submit requires card minima only

- **GIVEN** a pack with fewer than two answer cards
- **WHEN** the user attempts to submit answers for moderation
- **THEN** the system rejects the answers submit

#### Scenario [SC-PACK-09]: Tasks submit requires task minima and filled slots

- **GIVEN** a pack with fewer than two tasks, or any task with an empty slot
- **WHEN** the user attempts to submit tasks for moderation
- **THEN** the system rejects the tasks submit

#### Scenario [SC-PACK-35]: Separate submits succeed at their minima

- **GIVEN** a pack with at least two answer cards and an actor allowed to submit
- **WHEN** the user submits answers for moderation
- **THEN** an answers pending request and thread exist for that change author
- **AND GIVEN** answers are no longer dirty and the pack has at least two tasks with filled slots and valid difficulty
- **WHEN** the user submits tasks for moderation
- **THEN** a distinct tasks pending request and thread exist

#### Scenario [SC-PACK-36]: Dirty answers without pending block task create and edit

- **GIVEN** a verified editor whose answers draft is dirty relative to the last answers submit and the pack has **no** answers-pending request
- **WHEN** the editor attempts to create or edit tasks
- **THEN** the system rejects or prevents the action
- **AND** after a successful answers submit the editor MAY create and edit tasks again while answers stay pending

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

### Requirement: Public catalog shows live content after answers approval

The public catalog and the public pack page MUST list and show only packs that have an approved live answers revision (and live tasks as required for answers approve), and are not blocked. Draft and pending content MUST NOT appear as the public live view. While newer answers or tasks changes are pending, the world MUST continue to see the last approved live snapshot.

#### Scenario [SC-PACK-13]: Pending changes do not replace live public view

- **GIVEN** pack P has an approved live snapshot and a pending answers or tasks request
- **WHEN** any user opens the public catalog or public pack page for P
- **THEN** they see the approved live content
- **AND** they do not see the pending draft as live

#### Scenario [SC-PACK-14]: Never-approved pack is absent from catalog

- **GIVEN** a pack that has never had answers approved
- **WHEN** a user browses the public catalog
- **THEN** that pack is not listed

#### Scenario [SC-PACK-37]: Catalog listing follows answers approval

- **GIVEN** staff has approved tasks into live and then approved answers for pack P
- **WHEN** a user browses the public catalog
- **THEN** pack P is listed (unless blocked)

### Requirement: Staff moderation via answers hub

Moderator and admin MUST see a single queue of packs that have an **open** answers and/or tasks-only moderation request with status **pending** or **rejected** (needs revision). Tasks-only open requests (no open answers request) MUST appear. When both answers and tasks are open for the same pack, the queue MUST list the pack once via the answers request (nested tasks as today). Queue rows MUST show whether the open request is «на модерации» (pending) or «нужна доработка» (rejected). Opening a queue item MUST show the same hub layout: an answers/cards preview of the **full pack context** first, then a **list** of task sets with status marks — NOT a fully expanded dump of all questions. For answers-open, answers come from the answers request revision; for tasks-only, answers MUST come from the current live answers (read-only context). Staff MUST open a nested tasks page to review questions/slots and to approve/reject/cancel **tasks**. Approve/reject/cancel **answers** and the answers thread MUST be available on the hub only when an answers request is open (pending or rejected); for tasks-only they MUST NOT be shown or MUST be disabled. When answers are also open/pending, staff MUST approve **tasks** before approving **answers**. Approving answers MUST require live tasks already present and MUST publish the pack to the public catalog (unless blocked). For tasks-only, approving tasks MUST update live tasks without requiring an answers approve. Staff MAY reject with comment (request stays in the queue as needs-revision), cancel (leaves queue), approve from pending **or** rejected, or message on the relevant request thread. Threads remain visible only to that request’s change author and staff. After approval of a type, a later cycle of the same type MUST open a new thread. Non-staff MUST NOT perform staff actions.

#### Scenario [SC-PACK-18]: Staff approves answers into catalog after live tasks

- **GIVEN** pack P has answers pending, live tasks already approved, and a staff actor
- **WHEN** the actor approves the answers request
- **THEN** the submitted answers become the live answers revision
- **AND** the pack appears in the public catalog (unless blocked)
- **AND** answers are no longer pending

#### Scenario [SC-PACK-19]: Staff rejects with comment — stays in moderation

- **GIVEN** a pending answers or tasks request and staff actor
- **WHEN** the actor rejects with a non-empty comment
- **THEN** the change author can read the comment in that request’s thread
- **AND** both staff and author see status «нужна доработка» / needs-revision for that type
- **AND** the pack remains listed in the staff moderation queue
- **AND** the author may amend and resubmit on the same thread when locks allow

#### Scenario [SC-PACK-20]: Staff cancels open request

- **GIVEN** a pending or rejected answers or tasks request and staff actor
- **WHEN** the actor cancels that request
- **THEN** that request is no longer open for moderation
- **AND** the pack leaves the staff queue for that request
- **AND** locks for that type are released accordingly

#### Scenario [SC-PACK-21]: Non-staff cannot approve or cancel

- **GIVEN** an authenticated user with role `user`
- **WHEN** the user attempts to approve, reject, cancel, or block a pack
- **THEN** the system rejects the request

#### Scenario [SC-PACK-22]: New improvement opens a new thread per type

- **GIVEN** an approved answers (or tasks) cycle and an eligible editor who submits a new change of the same type
- **WHEN** moderation starts for that new change
- **THEN** a new moderation thread is created for that type (not a continuation of the old approved thread)

#### Scenario [SC-PACK-30]: Staff queue lists open answers and tasks-only packs

- **GIVEN** pack A has answers pending and pack B has only tasks pending
- **WHEN** staff opens the moderation queue
- **THEN** pack A is listed
- **AND** pack B is listed for its tasks-only pending
- **AND** a role `user` session MUST NOT access that staff queue

#### Scenario [SC-PACK-73]: Rejected request stays in staff queue

- **GIVEN** pack P had a pending tasks (or answers) request that staff rejected with a comment
- **WHEN** staff opens the moderation queue
- **THEN** pack P is still listed
- **AND** the row indicates needs-revision / «нужна доработка»

#### Scenario [SC-PACK-74]: Staff may approve a rejected request without resubmit

- **GIVEN** an answers or tasks request in rejected (needs-revision) status and staff actor
- **WHEN** the actor approves that request without a new author submit
- **THEN** the request’s submitted revision becomes live for that type (as on pending approve)
- **AND** the request leaves the staff queue
- **AND** post-reject draft edits that were not resubmitted MUST NOT become live

#### Scenario [SC-PACK-39]: Staff must approve tasks before answers

- **GIVEN** answers pending and tasks still pending (not live) for pack P
- **WHEN** staff attempts to approve answers
- **THEN** the system rejects answers approve until tasks are live
- **AND** staff can approve the nested tasks request from the tasks page first

#### Scenario [SC-PACK-40]: Tasks-only pending is visible with full pack hub

- **GIVEN** pack P has tasks pending and no answers pending
- **WHEN** staff opens the moderation queue and opens pack P
- **THEN** pack P appears in the queue
- **AND** the hub shows answers/cards context for the whole pack then the task-set list
- **AND** answers approve/reject controls are not available
- **AND** staff MAY approve or reject the tasks request from the nested tasks page

#### Scenario [SC-PACK-70]: Tasks-only hub uses live answers as context

- **GIVEN** pack P has approved live answers and a tasks-only pending request
- **WHEN** staff opens the tasks-only hub for P
- **THEN** the answers preview reflects the live answers content (full pack context)
- **AND** task sets reflect the pending tasks revision

#### Scenario [SC-PACK-71]: Tasks-only approve does not require answers approve

- **GIVEN** pack P is tasks-only pending and already catalog-live from a prior answers approve
- **WHEN** staff approves the tasks request
- **THEN** live tasks update accordingly
- **AND** no answers approve step is required to complete that cycle

#### Scenario [SC-PACK-72]: Client staff UI gates answers actions when tasks-only

- **GIVEN** staff opens a tasks-only hub item
- **WHEN** the hub renders
- **THEN** answers appear above the task-set list
- **AND** answers approve/reject buttons are hidden or disabled
- **AND** nested tasks moderation remains available

#### Scenario [SC-PACK-44]: Staff hub lists task sets; tasks page for detail

- **GIVEN** staff opens an answers-pending pack hub
- **WHEN** the hub renders
- **THEN** task sets that still need staff tasks work appear as a navigable list with status marks
- **AND** questions are not fully expanded on the hub
- **AND** the hub MUST NOT show a redundant separate control whose only job is to open the same tasks page as a list row
- **WHEN** staff opens a listed task set / tasks entry while tasks are still pending
- **THEN** the nested tasks page shows questions and tasks moderation actions

#### Scenario [SC-PACK-51]: After approving tasks staff returns to answers hub

- **GIVEN** staff is on the nested tasks page with tasks pending and approves tasks
- **WHEN** the approve succeeds
- **THEN** the client navigates to the answers hub for that pack
- **AND** the client MUST NOT leave staff on an empty tasks page that errors with a not-public / not-published pack message

#### Scenario [SC-PACK-52]: Moderated task sets leave the staff list

- **GIVEN** staff hub for an answers-pending pack whose tasks are already approved (live tasks, no tasks pending)
- **WHEN** the hub renders the nested task-set list
- **THEN** fully moderated task sets are omitted from that actionable list (or the list is empty with a clear live-tasks hint)
- **AND** staff continue answers moderation on the hub

### Requirement: Author and staff may exchange messages on an open thread

While a moderation request is not cancelled and not finally closed by approval without further work, the change author and staff MUST be able to append messages to that thread. Other users MUST NOT read or write that thread. The **cards** editing page MUST show the answers-type thread (status + messages + reply when pending or rejected). The **tasks** editing page MUST show the tasks-type thread the same way. Reject comments MUST be visible to the change author on the corresponding page without requiring a separate obscure route.

#### Scenario [SC-PACK-23]: Change author posts a reply in the thread

- **GIVEN** an open moderation thread for author A
- **WHEN** A posts a non-empty message
- **THEN** the message appears in the thread for A and for staff

#### Scenario [SC-PACK-24]: Unrelated user cannot read the thread

- **GIVEN** an open moderation thread for author A
- **WHEN** another non-staff user requests the thread
- **THEN** the system rejects or returns no thread content

#### Scenario [SC-PACK-45]: Author sees needs-revision status and comment on editor page

- **GIVEN** staff rejected an answers (or tasks) request with a non-empty comment
- **WHEN** change author A opens the corresponding cards (or tasks) editing page
- **THEN** the page shows «нужна доработка» / needs-revision status for that type
- **AND** the reject comment is readable in that type’s thread on the page
- **AND** the pack remains reachable from the author’s «На модерации» list

### Requirement: Staff may block a pack for everyone (server retains; UI deferred)

Moderator and admin MUST remain authorized on the server to block/unblock a pack. A blocked pack MUST remain visible in catalog and collections with a clear blocked state and MUST NOT be editable or submittable. Unblock MUST be restricted to moderator|admin. In this revision the **client MUST NOT** expose block/unblock buttons or entry points (staff discoverability deferred). Staff/public hard delete of a **published** pack remains out of scope.

#### Scenario [SC-PACK-25]: Blocked pack shows as blocked everywhere

- **GIVEN** an approved pack that staff blocks via an authorized server call
- **WHEN** any user views the catalog, collection, or pack page
- **THEN** the pack is shown as blocked
- **AND** edit and submit are rejected

#### Scenario [SC-PACK-26]: Only staff may unblock

- **GIVEN** a blocked pack and a role `user` actor
- **WHEN** the actor attempts to unblock
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-59]: Client hides block and unblock controls

- **GIVEN** a staff session on content staff pages
- **WHEN** the staff UI renders
- **THEN** block and unblock affordances are not shown

### Requirement: Email notifications for moderation events

When the change author is a non-anonymous user with an email, the system MUST send Russian email notifications (same mail channel as support) for: staff approve, staff reject, new staff message on the thread, and staff block of the pack (per relevant request/pack). Anonymous authors MUST NOT receive email. Links in mail MUST open the client SPA hash route for the pack / moderation view.

#### Scenario [SC-PACK-27]: Approve notifies the change author by email

- **GIVEN** a pending answers or tasks change by a registered author with email
- **WHEN** staff approves that change
- **THEN** the author receives an email about the approval
- **AND** the email links to the client SPA pack / moderation view

#### Scenario [SC-PACK-28]: Anonymous change author gets no email

- **GIVEN** moderation would notify but the identity is anonymous
- **WHEN** a notifiable moderation event occurs
- **THEN** no email is sent for that author

### Requirement: Client surfaces — split editor, collection-first, autosave

The client MUST expose a **unified packs list** as the primary lobby entry into the packs section (with create, filters, favorites star, status badges for pending/needs_revision/draft/unpublished only — **not** an in_catalog badge, and row→live/editor without stealing action clicks). There MUST NOT be a collection list as the primary entry; collect/remove-from-collection affordances MUST NOT be offered. A separate non-staff «На модерации» page MUST NOT be required — open author items MUST be reachable via the **on moderation** list filter. Staff MUST retain the staff moderation queue. Live pack view MUST be available for in-catalog packs without collection membership; **Edit** follows author / task-set-author / staff rules in this change. Never-published and pack-level author draft/pending/needs_revision MUST open Edit from the list; open **add-task-set** only MUST open live first then the never-live set row → Edit. Cards editor, tasks / add-task-set, three-phase status vocabulary, cascade yellow, slot chips, quiet autosave, and delete-card confirm copy MUST remain as in the base capability except where membership/collection is removed.

#### Scenario [SC-PACK-81]: Yellow highlight on task and task set after cascade

- **GIVEN** cascade from an answers content change or delete emptied slots on task T in set S
- **WHEN** the editor views the task-set list and the tasks list for S
- **THEN** set S and task T are highlighted (yellow/warning) while those cascade gaps remain
- **AND** individual slot chips are not required to use that yellow highlight

#### Scenario [SC-PACK-82]: Delete-card confirm mentions published vs draft

- **GIVEN** pack P has a live/catalog revision
- **WHEN** the editor confirms deleting an answer card
- **THEN** the confirm copy states the card will be removed from the published pack
- **AND GIVEN** pack P has no live revision
- **WHEN** the editor confirms deleting an answer card
- **THEN** the confirm copy states the card will be removed from the draft

#### Scenario [SC-PACK-83]: Page subtitle statuses match list mark vocabulary

- **GIVEN** answers or tasks are in a three-phase state (awaiting submit, pending, or needs revision)
- **WHEN** the corresponding editor page renders
- **THEN** the status under the page title uses the same phase labels as the list/row marks

#### Scenario [SC-PACK-84]: Task list rows show answer slots

- **GIVEN** a task-set page with one or more tasks that have slots
- **WHEN** the tasks list renders
- **THEN** each task row shows its answer slots (filled content and/or empty)

#### Scenario [SC-PACK-75]: Author opens own packs from «На модерации»

- **GIVEN** user A is change author of an open pending or needs_revision request on pack P
- **WHEN** A applies the on-moderation filter on the unified packs list (replacing the separate «На модерации» page for non-staff)
- **THEN** pack P is listed
- **AND** packs where A is not the change author of an open request are not listed solely for that reason
- **AND** choosing P navigates to the cards editor (or add-task-set surface when the open request is task-set-only)

#### Scenario [SC-PACK-76]: Author list drops pack after approve or cancel

- **GIVEN** pack P appears under A’s on-moderation filter for an open request
- **WHEN** staff approves or cancels that open request (and no other open request remains for A on P)
- **THEN** P no longer appears under A’s on-moderation filter

#### Scenario [SC-PACK-77]: Cards answers status uses three-phase labels including dirty

- **GIVEN** answers are dirty relative to the last answers submit and there is no open answers request
- **WHEN** the cards page renders
- **THEN** answers status shows «ожидает отправки на модерацию»
- **AND GIVEN** answers become pending after submit
- **WHEN** the cards page renders
- **THEN** answers status shows «на модерации»

#### Scenario [SC-PACK-29]: Catalog lists approved packs

- **GIVEN** at least one pack with approved answers (catalog-eligible) exists
- **WHEN** an authenticated user opens the unified packs list (catalog browsing surface)
- **THEN** those packs are listed for browsing

#### Scenario [SC-PACK-31]: Ineligible create shows auth or verify prompt

- **GIVEN** a guest or unverified user on a create-pack entry point
- **WHEN** the user attempts to create
- **THEN** the client shows a prompt to sign in or confirm email
- **AND** no pack is created

#### Scenario [SC-PACK-41]: Lobby opens collection first

- **GIVEN** an authenticated user on the lobby
- **WHEN** the user opens the packs section navigation entry
- **THEN** the unified packs list is shown first (collection-first entry is removed)
- **AND** no collection page is required as the primary entry

#### Scenario [SC-PACK-46]: Submit disabled when nothing to send

- **GIVEN** an editor whose answers draft is not dirty relative to the last answers submit and already meets card minima
- **WHEN** the cards page renders
- **THEN** answers submit is disabled
- **AND GIVEN** tasks are not dirty relative to the last tasks submit and meet task minima
- **WHEN** the tasks page renders
- **THEN** tasks submit is disabled

#### Scenario [SC-PACK-47]: Question requires a selected answer slot

- **GIVEN** a user composing a new question with empty slots
- **WHEN** the user attempts to add/save the question
- **THEN** the client prevents the action until at least one slot references an answer card

#### Scenario [SC-PACK-48]: Task-set list shows three-phase moderation marks

- **GIVEN** the editor’s cards page task-set list for pack P
- **WHEN** a task set is dirty since the last successful tasks submit and tasks are not pending/rejected
- **THEN** that set shows «ожидает отправки на модерацию»
- **WHEN** tasks are pending for P
- **THEN** task sets in that open request show «на модерации» (not only a cleared dirty mark)
- **WHEN** tasks are rejected (needs-revision) for P
- **THEN** those sets show «нужна доработка»

#### Scenario [SC-PACK-60]: Needs-moderation marks clear after tasks approval without further edits

- **GIVEN** author A submitted tasks, staff approved tasks, and A did not edit tasks after that submit
- **WHEN** A opens the cards page task-set list (including after reload)
- **THEN** those task sets MUST NOT show «ожидает отправки» / dirty mark
- **AND** the tasks status MAY show approved

#### Scenario [SC-PACK-49]: Answers and tasks pages use the same three-phase vocabulary

- **GIVEN** answers (or tasks) are dirty without an open request, pending, or rejected for the editor’s pack
- **WHEN** the editor opens the cards (or tasks) page
- **THEN** the page shows the corresponding label: «ожидает отправки на модерацию» / «на модерации» / «нужна доработка» (or approved when applicable after a closed cycle)

#### Scenario [SC-PACK-50]: Quiet autosave without top saving caption

- **GIVEN** the editor changes a card field that triggers autosave
- **WHEN** the draft saves
- **THEN** the UI does not insert a top-of-page saving caption that shifts layout
- **AND** button loading MAY indicate in-flight save/submit

### Requirement: Difficulty is stored for future peek rewards

Task difficulty values `1`, `2`, and `3` MUST be persisted with each task as the intended future peek step reward. This capability MUST NOT change tourist-room peek runtime behavior in this change (board rewards remain as specified in `game/board`).

#### Scenario [SC-PACK-32]: Difficulty is stored on the task

- **GIVEN** a verified editor saves a task with difficulty `2`
- **WHEN** the task is later retrieved
- **THEN** the stored difficulty is `2`

#### Scenario [SC-PACK-33]: Peek stub behavior unchanged

- **GIVEN** this capability is deployed
- **WHEN** a player opens a peek in a tourist room
- **THEN** peek still uses the existing stub correct/incorrect controls from `game/board`
- **AND** pack tasks are not required to answer the peek

### Requirement: Author may delete unpublished pack or task set

While a pack is **unpublished** (no approved live answers / not in the public catalog), the pack’s **creator** (`createdBy`) MUST be able to: (1) hard-delete the entire pack («набор карточек»), which MUST remove drafts, revisions, collection memberships, and moderation requests/messages (**cascade** pending staff items); (2) delete one task set («набор заданий») from the author’s draft and from any unpublished live-tasks snapshot, even if staff already approved those tasks. Other users MUST NOT perform these deletes. Deleting a published pack (live answers in catalog) remains out of scope for authors and staff in this revision. Destructive deletes MUST ask for confirmation in the client.

#### Scenario [SC-PACK-55]: Creator deletes unpublished pack with cascade

- **GIVEN** pack P has no live answers revision, was created by user A, and has an answers-pending and/or tasks-pending request
- **WHEN** A confirms delete of the pack
- **THEN** P is removed from storage and from all collections
- **AND** those pending requests no longer appear in the staff queue

#### Scenario [SC-PACK-56]: Creator deletes a task set including approved liveTasks while unpublished

- **GIVEN** pack P is unpublished, created by A, has live tasks already approved, and draft task set S
- **WHEN** A confirms delete of task set S
- **THEN** S is absent from A’s draft
- **AND** S is absent from the pack’s live-tasks content
- **AND** a non-creator MUST NOT be able to delete S

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

After publish, a verified non-anonymous user MUST be able to create and submit a **new** task set for an in-catalog non-blocked pack **without** collection membership. New tasks MUST reference only existing live answer cards. After that set becomes live, its author MAY re-edit it through moderation (see ADDED); other non-staff MUST NOT edit it except pack `createdBy` under pack edit rules. Staff approval of an add-task-set request MUST publish **only** that task set into live. Needs revision and author Cancel apply as for first publish. Guests and unverified users MUST NOT add-task-set.

#### Scenario [SC-PACK-108]: Verified collector submits new task set

- **GIVEN** published in-catalog pack P and verified user U (collection membership not required)
- **WHEN** U submits a new task set using only live card ids
- **THEN** an open moderation request for that task set exists
- **AND** live content is unchanged until staff Approve

#### Scenario [SC-PACK-109]: New task set cannot introduce new cards

- **GIVEN** published pack P
- **WHEN** a non-staff user attempts to add cards as part of a new task set submit
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-110]: Staff approve adds only the new task set

- **GIVEN** open add-task-set request for set S on published pack P and staff who has taken the request
- **WHEN** staff Approves that request
- **THEN** live P includes S
- **AND** existing live cards and other task sets remain as they were

### Requirement: Staff may edit any pack live without moderation under exclusive lock

Moderator and admin MUST be able to open Edit on any pack (published or not) **without** collection membership, subject to the exclusive edit lock and **only when** no open author-driven moderation request blocks staff content edit. Staff content changes MUST apply **directly to live or to the single working copy** without entering the moderation queue. The client MUST NOT show staff a «submit for moderation» control for their own staff edits. While any eligible editor A holds an edit lock on pack P, another actor’s Edit attempt MUST fail. The lock MUST release when the holder leaves the entire edit session or after the shared TTL — NOT when navigating between cards and task-set editors of the same pack. Staff moderation queue actions (approve / needs_revision / cancel) require take (see ADDED). Staff MUST NOT see the author my-moderation nav control.

#### Scenario [SC-PACK-111]: Staff edits published pack without queue

- **GIVEN** published pack P with no open author content request and free edit lock, and staff S
- **WHEN** S edits a card or task set on P and saves
- **THEN** the change is visible in the catalog live content
- **AND** no new moderation request is created for that save

#### Scenario [SC-PACK-112]: Staff Edit without collection membership

- **GIVEN** published pack P and staff S
- **WHEN** S opens Edit on P (lock free, no blocking author request)
- **THEN** Edit is allowed without collection membership

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
- **WHEN** the user views the packs list chrome
- **THEN** the «На модерации» (author my-moderation) control is not shown
- **AND** staff MUST open the staff moderation queue from the shared header, not from an embedded list control

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

Wherever the client lists tasks/questions for review or editing (staff moderation hub preview, add-task-set page question list, task-set editor list, live pack drill-in task list), each task row MUST show its answer slots. A filled slot MUST display the referenced answer card’s textual content (same resolution as when viewing a live pack / task-set page). An empty slot MUST show an empty affordance. A generic filled placeholder (e.g. «заполнен») MUST NOT be shown when the card content is available. For staff preview of an add-task-set (`task_set`) request, answer cards used for slot resolution MUST come from the pack’s **live** answers (revision for add-task-set does not store cards). Staff MUST be able to see slot bindings without opening a separate nested tasks-only page. A separate top-level «Ответы» list on the staff hub for `task_set` is not required when slots already show card text.

#### Scenario [SC-PACK-127]: Staff hub and add-task-set lists show slots

- **GIVEN** a moderation preview or add-task-set page with tasks that have slots
- **WHEN** the questions list renders
- **THEN** each task row shows its answer slots (filled and/or empty)
- **AND** the same rule applies on staff request hub and on the author add-task-set list

#### Scenario [SC-PACK-134]: Staff task_set preview shows answer text in slots

- **GIVEN** an open add-task-set (`task_set`) moderation request whose tasks reference live answer cards by id
- **AND** those live cards have non-empty content
- **WHEN** staff opens the staff request hub preview
- **THEN** each filled slot chip shows that card’s content text
- **AND** MUST NOT show only a generic «filled» / «заполнен» placeholder for those slots

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

### Requirement: Task-set rows show author display name

Wherever the client lists or titles a task set (live pack summary, cards editor list, staff moderation hub, task-set / questions page header when a set label is shown), each set MUST be labeled with the contributing author’s display identity: preferred form «Набор заданий {n} от {name}» (or equivalent i18n). The name MUST be the user’s `displayName` when non-empty; otherwise the local-part of their email (substring before `@`). The name MUST be visible to **all** viewers including public live. Multiple sets by the same author MUST each show the author name on their own row (no collapsing). Optional co-author labels MAY appear beside the author name; they remain display-only.

#### Scenario [SC-PACK-135]: Live and editor show author on every task-set row

- **GIVEN** pack P with two task sets by the same author whose displayName is «Мария»
- **WHEN** a viewer opens the live pack page or the cards editor task-set list
- **THEN** each task-set row includes «от Мария» (or equivalent) in its label
- **AND** the two rows are not merged into one
- **AND WHEN** the author has no displayName but email `ivan@example.com`
- **THEN** the label uses local-part `ivan`

### Requirement: Back from questions list is not the pack title

On the task-set questions page (including live drill-in read-only), the control that navigates back to the pack live page or cards editor MUST use a stable back label (e.g. «Вернуться» or the existing «К карточкам»). It MUST NOT use the pack title (or task-set title) as that control’s visible label.

#### Scenario [SC-PACK-136]: Live drill-in back is not pack title

- **GIVEN** a viewer on the live pack questions view for a task set of pack P titled «Большой пак»
- **WHEN** the page header back control renders
- **THEN** the control label is a generic back affordance («Вернуться» / «К карточкам» or equivalent)
- **AND** MUST NOT equal P’s title «Большой пак»

### Requirement: Soft-unpublish pack cancels open moderation requests

When staff soft-unpublishes a pack that is currently in the public catalog, the system MUST set every **open** moderation request for that pack (`pending` or `needs_revision`, any request type including `pack`, `task_set`, and legacy `answers`/`tasks`) to **cancelled**. After that cascade, those requests MUST NOT appear in the author’s «На модерации» list or in the staff moderation queue. Soft-unpublish of an already soft-unpublished pack MUST NOT re-cancel or re-notify. **Republish** MUST NOT restore cancelled requests; authors MUST submit again if they still want review. Soft-unpublish of an individual **task set** MUST NOT by itself cancel pack-level open requests. The staff confirm dialog for pack unpublish MUST warn that open moderation requests will be cancelled. The system MUST NOT append a moderation thread message solely for this cascade cancel.

#### Scenario [SC-PACK-137]: Unpublish cancels open add-task-set request

- **GIVEN** in-catalog pack P with an open `task_set` moderation request authored by A
- **WHEN** staff soft-unpublishes P
- **THEN** that request status is `cancelled`
- **AND** the request MUST NOT appear on A’s «На модерации» list
- **AND** the request MUST NOT appear in the staff moderation queue

#### Scenario [SC-PACK-138]: Republish does not restore cancelled request

- **GIVEN** pack P was soft-unpublished while A had an open request that was cascade-cancelled
- **WHEN** staff republishes P
- **THEN** A’s cancelled request remains cancelled
- **AND** A MUST NOT see that request on «На модерации» solely because of republish

#### Scenario [SC-PACK-139]: Confirm warns about cancelling open requests

- **GIVEN** staff S about to soft-unpublish in-catalog pack P
- **WHEN** the client shows the unpublish confirmation
- **THEN** the confirm copy MUST state that open moderation requests for P will be cancelled
- **AND** cancelling the dialog MUST leave P in catalog and leave open requests unchanged

### Requirement: Email on pack unpublish cascade cancel

When staff soft-unpublish of a pack cascade-cancels one or more open moderation requests, the system MUST send Russian email (same mail channel as other content moderation mail) to each distinct **change author** of those cancelled requests who is a non-anonymous user with an email. Each such author MUST receive **exactly one** email for that unpublish event, even if several of their requests on that pack were cancelled. The email MUST state that the pack was soft-unpublished and that their pending moderation submission(s) were cancelled. The email MUST **NOT** include any URL or deep-link. Anonymous authors MUST NOT receive email. Manual author/staff Cancel of a request (outside this cascade) MUST NOT gain a new email requirement from this change. Soft-unpublish when the pack was already soft-unpublished MUST NOT send cascade-cancel email again.

#### Scenario [SC-PACK-140]: Cascade cancel notifies each author once without links

- **GIVEN** in-catalog pack P with open moderation request(s) by registered author A with email
- **WHEN** staff soft-unpublishes P (cascade-cancelling A’s open request(s) on P)
- **THEN** A receives exactly one Russian email about the unpublish and cancelled submission(s)
- **AND** that email contains no URL / deep-link
- **AND** anonymous change authors receive no email

#### Scenario [SC-PACK-141]: Multiple requests for one author → one email

- **GIVEN** in-catalog pack P with two open moderation requests both authored by A
- **WHEN** staff soft-unpublishes P
- **THEN** both requests are cancelled
- **AND** A receives exactly one cascade-cancel email for that event

### Requirement: Unified packs list with status badges

The client MUST expose a single **packs list** (reachable from the lobby as the packs section entry, and later from the shared header) showing: (1) all **approved in-catalog** non-blocked packs to any authenticated user (including anonymous); (2) for the current user, that user’s **own never-published** packs (creator drafts, including those with an open first-publish request); (3) for staff, soft-unpublished packs with a clear unpublished state. Status badges on rows MUST cover **pending**, **needs_revision**, **draft**, and soft-**unpublished** (staff) when applicable. Rows that are simply published / in catalog MUST **NOT** show an «В каталоге» / in_catalog status badge. Non-authors MUST NOT see another user’s never-published packs. There MUST be a **Create pack** affordance for eligible verified users. The system MUST NOT require collection membership to view or open an in-catalog pack.

#### Scenario [SC-PACK-148]: In-catalog packs visible to all sessions

- **GIVEN** an approved in-catalog pack P and any authenticated user U (including anonymous)
- **WHEN** U opens the packs list
- **THEN** P appears on the list
- **AND** P MUST NOT show an «В каталоге» / in_catalog status badge

#### Scenario [SC-PACK-149]: Author sees own draft and pending on the same list

- **GIVEN** creator A has never-published pack P with an open pending moderation request
- **WHEN** A opens the packs list
- **THEN** P appears for A with a pending status
- **AND** another non-staff user B does not see P on the list

#### Scenario [SC-PACK-150]: Draft without submit shows draft status

- **GIVEN** creator A has never-published pack P with no open moderation request
- **WHEN** A opens the packs list
- **THEN** P appears for A with a draft status

### Requirement: Packs list filters

The packs list MUST support filters: **all** (default); **on moderation** (caller’s own open pending or needs_revision requests — packs and add-task-set as applicable); **drafts** (caller’s never-published packs without an open request, or equivalently never-published not yet submitted); **mine** (packs where the caller is `createdBy` OR has contributed at least one task set on that pack); **favorites** (packs the caller has starred). Guests and anonymous users MUST only see the public catalog subset; filters that require identity beyond catalog (mine / favorites / moderation / drafts) MUST yield empty or be unavailable for guests. Staff soft-unpublished visibility rules MUST still apply when staff use filters.

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

### Requirement: Pack favorites (star)

A registered non-anonymous user MUST be able to star and unstar an **in-catalog** pack from the packs list and from the pack detail surface. Starred state MUST persist per user. The packs list and pack detail MUST show a star affordance reflecting favorite state. Anonymous guests MUST NOT star packs. Soft-unpublished packs MUST NOT be newly starred by non-staff; existing stars MAY remain invisible to non-staff while unpublished. Favorites MUST NOT replace catalog visibility — they are a personal filter only.

#### Scenario [SC-PACK-154]: Registered user stars and unstars

- **GIVEN** a registered user U and in-catalog pack P
- **WHEN** U stars P
- **THEN** P appears under U’s favorites filter
- **WHEN** U unstars P
- **THEN** P no longer appears under U’s favorites filter

#### Scenario [SC-PACK-155]: Guest cannot star

- **GIVEN** an anonymous guest and in-catalog pack P
- **WHEN** the guest attempts to star P
- **THEN** the system rejects the action

### Requirement: Author may re-edit published pack content through moderation

After pack P has live catalog content, the pack **creator** (`createdBy`) MUST be able to open Edit on cards / existing task sets under the exclusive edit lock, mutate a working copy, and **submit** changes into a new or resumed moderation request. Author saves MUST NOT apply directly to live without staff approve. While an open author-driven moderation request exists for that pack content cycle, staff MUST NOT acquire the content edit lock or save live content edits for P. Other non-staff users (except task-set authors per their set rule below) MUST NOT edit existing live cards or existing live task sets.

#### Scenario [SC-PACK-156]: Creator submits post-publish edits to moderation

- **GIVEN** in-catalog pack P created by A and no open author content request blocking staff
- **WHEN** A acquires the edit lock, amends cards, and submits
- **THEN** an open moderation request exists with A as change author
- **AND** live catalog content remains the last approved snapshot until staff approve

#### Scenario [SC-PACK-157]: Staff cannot content-edit while author request open

- **GIVEN** pack P has an open author pending or needs_revision request for pack content
- **WHEN** staff S attempts to acquire content Edit lock or staff-save on P
- **THEN** the system rejects the action

### Requirement: Task-set author may re-edit own set through moderation

A verified user who is the **author** of an existing task set on pack P MUST be able to Edit that task set (not other users’ sets and not answer cards unless they are also pack `createdBy`) under the exclusive pack edit lock and submit changes through moderation. Approval MUST update only that set’s live content. Other non-staff users MUST NOT edit that set.

#### Scenario [SC-PACK-158]: Task-set author resubmits own set

- **GIVEN** published pack P with live task set S authored by U (U ≠ pack createdBy)
- **WHEN** U edits S under lock and submits
- **THEN** an open task-set moderation request exists for S
- **AND** live S remains unchanged until staff approve

#### Scenario [SC-PACK-159]: Task-set author cannot edit other sets or cards

- **GIVEN** published pack P with task set S1 by U and S2 by V, and answer cards owned by pack createdBy
- **WHEN** U attempts to edit S2 or answer cards
- **THEN** the system rejects the action

### Requirement: Exclusive edit lock includes author editors

The exclusive edit lock on a pack MUST be acquirable by an eligible editor: pack `createdBy` (when Edit is allowed), a task-set author editing their set, or staff (when staff Edit is allowed). While user A holds a non-expired lock, user B MUST NOT acquire it. The lock MUST use the same TTL policy as the existing staff edit lock (approximately five minutes after last activity / disconnect). Leaving the edit session or explicit unlock MUST release the lock. Staff MUST NOT acquire the lock while an open author moderation request blocks staff content edit (see above).

#### Scenario [SC-PACK-160]: Second editor rejected while author holds lock

- **GIVEN** creator A holds a non-expired edit lock on pack P
- **WHEN** staff S or another eligible editor attempts to acquire the lock
- **THEN** the system rejects with an edit-locked outcome

### Requirement: Staff must take a moderation request before moderating

Moderator and admin MUST **take** an open moderation request (pending or needs_revision) before they MAY approve, needs-revision/reject with comment, cancel, or otherwise perform moderation actions on that request. Take MUST be available on the staff queue row and on the request detail surface. While staff A holds a non-expired take on request R, other staff MUST NOT take R and MUST NOT perform moderation actions on R. Take TTL MUST match the edit-lock TTL policy. Release MUST occur on explicit release, leaving the moderation surface, successful terminal action (approve / cancel), or TTL expiry. Author amend and resubmit on the same open request MUST remain allowed while staff holds take. Content Edit by staff remains separately governed by the edit lock and author-open-request block.

#### Scenario [SC-PACK-161]: Cannot approve without take

- **GIVEN** open pack request R and staff S who has not taken R
- **WHEN** S attempts to approve R
- **THEN** the system rejects the action

#### Scenario [SC-PACK-162]: Second staff cannot take while held

- **GIVEN** staff A holds a non-expired take on request R
- **WHEN** staff B attempts to take R
- **THEN** the system rejects the action
- **AND** B MUST NOT approve, needs-revision, or cancel R

#### Scenario [SC-PACK-163]: Author may resubmit while staff holds take

- **GIVEN** staff A holds take on open request R authored by U
- **WHEN** U amends and resubmits on R
- **THEN** the resubmit succeeds subject to existing validation
- **AND** A’s take remains until release, terminal action, or TTL

### Requirement: Verified user may add task set without collection

After publish, any verified non-anonymous user MUST be able to create and submit a **new** task set for an in-catalog non-blocked pack without collection membership. Guests and unverified users MUST NOT. Add-task-set moderation and approve rules otherwise remain as in the base capability (new set only; submit through queue).

#### Scenario [SC-PACK-164]: Verified non-collector submits add-task-set

- **GIVEN** in-catalog pack P and verified user U with no collection membership concept required
- **WHEN** U creates and submits a new task set for P
- **THEN** an open task-set moderation request exists
- **AND** the system does not require collection membership

#### Scenario [SC-PACK-165]: Guest cannot add-task-set

- **GIVEN** an anonymous guest and in-catalog pack P
- **WHEN** the guest attempts to start add-task-set
- **THEN** the system rejects the action

### Requirement: Non-staff have no author my-moderation navigation

Non-staff users MUST NOT be shown a separate «На модерации» navigation entry for packs. Their open items MUST be reachable via the packs list filter **on moderation**. Staff MUST retain the shared staff moderation queue. The author my-moderation HTTP list MAY remain for staff tooling or be unused by non-staff UI.

#### Scenario [SC-PACK-166]: Non-staff packs chrome hides my-moderation nav

- **GIVEN** a non-staff authenticated user on the packs section
- **WHEN** the user views packs navigation chrome
- **THEN** the author «На модерации» control is not shown
- **AND** the on-moderation list filter is available

### Requirement: Task-set rows show moderation status for set author and staff

On the live pack surface task-set list, each **visible** task set MUST show a moderation status distinguishing at least: live/published without open author work; open **pending**; open **needs_revision**; and author **draft** (working edits not in an open request, including after Cancel). An open request MUST mark **only** the task set(s) that belong to that request (matched via the request revision payload / set identity among that author’s sets). The system MUST NOT apply one author’s open status to every live task set sharing that author’s user id. The **author of that task set** and **staff** (moderator|admin) MUST see these marks on **already-live** sets under open re-edit. The pack creator MUST NOT see another user’s task-set moderation marks solely by being pack `createdBy`. Other non-staff users MUST NOT see foreign set moderation marks. Never-live add-task-set visibility is specified separately (set author only).

#### Scenario [SC-PACK-171]: Set author sees pending on live set row

- **GIVEN** published pack P with task set S authored by U and an open pending request for S
- **WHEN** U opens the live pack task-set list
- **THEN** S shows a pending status indication

#### Scenario [SC-PACK-172]: Set author sees needs_revision on live set row

- **GIVEN** published pack P with task set S authored by U and an open needs_revision request for S
- **WHEN** U opens the live pack task-set list
- **THEN** S shows a needs_revision status indication

#### Scenario [SC-PACK-173]: Pack creator does not see foreign set moderation marks

- **GIVEN** pack P created by A, task set S authored by U (U ≠ A), and S has an open pending request
- **WHEN** non-staff A views the live pack task-set list
- **THEN** A MUST NOT see a pending/needs_revision moderation mark on S solely as pack creator

#### Scenario [SC-PACK-174]: Staff sees set moderation marks

- **GIVEN** pack P with already-live task set S under open pending moderation and staff S1
- **WHEN** S1 views the live pack task-set list
- **THEN** S shows a pending status indication

#### Scenario [SC-PACK-187]: Open status does not fan out to sibling live sets

- **GIVEN** published pack P with live task sets S0 and S1 both authored by U
- **AND** an open needs_revision (or pending) request that covers only S1 (e.g. re-edit or add of S1)
- **WHEN** U opens the live pack task-set list
- **THEN** S1 shows the open status
- **AND** S0 MUST NOT show pending or needs_revision solely because U authored both

### Requirement: Cancel returns author work to draft on the unified list

When staff or the change author **cancels** an open moderation request (pack, task_set, or equivalent pack content cycle), the system MUST keep the working copy edits and MUST NOT treat Cancel as hard-delete. For the change author, the affected entity MUST appear as **draft** on the unified packs list (including the drafts filter) so they can open Edit and submit again. Other users MUST continue to see the last approved **live** catalog snapshot when one exists. If the author **hard-deletes** the unpublished entity, it MUST be removed (not shown as draft). Soft-unpublish cascade cancel of open requests MUST keep working edits and follow the same author-draft visibility rules for those authors.

For a **never-live add-task-set** cycle, Cancel MUST keep the request revision payload. Opening the add-task-set Edit surface after Cancel MUST load that retained snapshot (task sets with questions and slots), not an empty task-set list. The author MUST be able to amend and submit again from that restored draft.

#### Scenario [SC-PACK-175]: Staff cancel yields author draft on list

- **GIVEN** author A has an open pending pack or task_set request on entity E and staff who has taken the request
- **WHEN** staff cancels that request
- **THEN** A sees E as draft on the unified packs list
- **AND** the working edits remain available for Edit

#### Scenario [SC-PACK-176]: Author cancel yields author draft on list

- **GIVEN** author A has an open pending request on entity E
- **WHEN** A cancels that request
- **THEN** A sees E as draft on the unified packs list

#### Scenario [SC-PACK-177]: Author delete removes entity

- **GIVEN** never-published pack P created by A with no catalog live
- **WHEN** A hard-deletes P
- **THEN** P is removed and MUST NOT appear as a draft for A

#### Scenario [SC-PACK-178]: Others keep live snapshot after cancel of post-publish edits

- **GIVEN** in-catalog pack P with live content, author A’s open pending content request cancelled, and non-author user B
- **WHEN** B opens the packs list
- **THEN** B sees P with the live in-catalog snapshot
- **AND** A sees P as draft for their retained working edits

#### Scenario [SC-PACK-179]: Cancelled task_set request is draft for set author

- **GIVEN** set author U had an open task_set request on pack P that was cancelled (staff or U)
- **WHEN** U opens the packs list and/or the live pack task-set list
- **THEN** U can reach the retained working edits as draft (list and/or set-row draft mark)

#### Scenario [SC-PACK-196]: Cancelled never-live add-task-set Edit restores submitted snapshot

- **GIVEN** verified user U submitted a never-live add-task-set on in-catalog pack P with at least two tasks and filled slots
- **AND** that request was cancelled (by U or by staff after take)
- **WHEN** U opens the add-task-set Edit surface for P (GET draft)
- **THEN** the draft includes the same task questions and slot answer references that were submitted
- **AND** the draft MUST NOT be an empty task-set list solely because the request is cancelled

#### Scenario [SC-PACK-197]: After cancel add-task-set author can amend and resubmit retained draft

- **GIVEN** set author U has a cancelled never-live add-task-set draft restored on Edit for pack P
- **WHEN** U amends a question and submits again
- **THEN** a new or resumed open pending task_set request exists carrying the amended content

### Requirement: Breadcrumbs replace pack back affordances

Content pack surfaces MUST show breadcrumbs **below** the shared elevated header chrome in the page/layout zone that receives header offset (not inside the elevated header bar, and not covered by the fixed header), e.g. Lobby / Packs / pack title / cards or task set. The live pack detail MUST NOT require a separate «К наборам» control when breadcrumbs provide that path. The live tasks drill-in MUST NOT require a separate «Вернуться» control when breadcrumbs provide return to the pack. Pack moderation thread, staff moderation queue/detail, and author my-moderation MUST NOT show a separate «К наборам» when breadcrumbs cover Lobby / Packs / Модерация. Staff moderation routes MUST show breadcrumbs (Lobby / Модерация [/ …]).

#### Scenario [SC-PACK-180]: Breadcrumbs on pack live and tasks

- **GIVEN** an authenticated user on a live pack or its tasks drill-in
- **WHEN** the page renders
- **THEN** breadcrumbs include a path back toward Lobby and Packs

#### Scenario [SC-PACK-181]: Pack detail has no «К наборам» when crumbs present

- **GIVEN** an authenticated user on live pack detail with breadcrumbs
- **WHEN** the page renders
- **THEN** a separate «К наборам» control is not shown

#### Scenario [SC-PACK-182]: Tasks drill-in has no «Вернуться» when crumbs present

- **GIVEN** an authenticated user on live tasks drill-in with breadcrumbs
- **WHEN** the page renders
- **THEN** a separate «Вернуться» control is not shown

#### Scenario [SC-PACK-193]: Pack breadcrumbs sit below elevated header

- **GIVEN** an authenticated user on a pack live or tasks route with breadcrumbs
- **WHEN** the page renders
- **THEN** breadcrumbs are outside the elevated shared header bar
- **AND** breadcrumbs sit in the page/layout zone offset for the header (not covered by the fixed header)

#### Scenario [SC-PACK-194]: Pack moderation has no «К наборам» when crumbs present

- **GIVEN** an authenticated user on a pack moderation thread with breadcrumbs
- **WHEN** the page renders
- **THEN** a separate «К наборам» control is not shown

#### Scenario [SC-PACK-195]: Staff moderation chrome has no «К наборам»

- **GIVEN** an authenticated moderator or admin on the staff moderation queue or staff request detail with breadcrumbs
- **WHEN** the page renders
- **THEN** a separate «К наборам» control is not shown on that chrome

### Requirement: Staff moderation entry is not on packs list chrome

Moderator and admin MUST open the staff moderation queue from the **shared application header** (see `ui/branding`), not from an embedded «Модерация» control on the packs list chrome. Packs list chrome MUST NOT show a staff moderation nav entry.

#### Scenario [SC-PACK-183]: Staff reaches queue from header

- **GIVEN** an authenticated moderator or admin on any non-Game authenticated screen with the shared header
- **WHEN** the user activates the header «Модерация» (or equivalent) control
- **THEN** the staff moderation queue is shown

#### Scenario [SC-PACK-184]: Packs list has no staff moderation link

- **GIVEN** an authenticated moderator or admin on the packs list
- **WHEN** the user views packs list chrome
- **THEN** an embedded staff «Модерация» control is not shown on that list chrome

### Requirement: No in_catalog status badge on packs list

Published in-catalog packs MUST appear on the packs list without an «В каталоге» / in_catalog status badge. Other status badges (pending, needs_revision, draft, unpublished) MUST remain when applicable.

#### Scenario [SC-PACK-185]: Published pack has no in_catalog badge

- **GIVEN** an approved in-catalog pack P with no open author-facing draft/pending/needs_revision mark for the caller
- **WHEN** the caller opens the packs list
- **THEN** P has no «В каталоге» / in_catalog status badge

### Requirement: Never-published pack opens Edit directly

Choosing a never-published pack from the packs list MUST open the pack editor (Edit) directly, not a separate browse-only surface first.

#### Scenario [SC-PACK-186]: Never-published pack row opens Edit

- **GIVEN** creator A has never-published pack P
- **WHEN** A chooses P from the packs list
- **THEN** the pack editor (Edit) opens

### Requirement: Never-live add-task-set appears for set author on live task-set list

While set author U has a **never-live** task set in an add-task-set cycle (open **pending** or **needs_revision**, or author **draft** after Cancel with retained working), the live pack task-set list for U MUST include that set as a row with the same status vocabulary as the unified packs list (pending / needs_revision / draft). Non-authors (including pack `createdBy` and staff viewing live) MUST NOT see that never-live row; staff review that work via the staff moderation queue. Activating the never-live row MUST open **Edit** (add-task-set amend surface).

#### Scenario [SC-PACK-188]: Set author sees never-live set with needs_revision

- **GIVEN** published pack P with live set S0 and set author U’s never-live add-task-set S1 under open needs_revision
- **WHEN** U opens the live pack task-set list
- **THEN** S1 appears as a row with needs_revision status
- **AND** S0 remains available without falsely inheriting needs_revision solely from S1’s request (see SC-PACK-187)

#### Scenario [SC-PACK-189]: Others do not see never-live set on live list

- **GIVEN** the same pack P and never-live S1 for U, and non-author viewer V (pack creator or another user or staff on live)
- **WHEN** V opens the live pack task-set list
- **THEN** S1 MUST NOT appear as a never-live row for V

#### Scenario [SC-PACK-190]: Never-live set row opens Edit

- **GIVEN** set author U sees never-live set S1 on the live pack task-set list
- **WHEN** U activates that row
- **THEN** Edit (add-task-set amend) opens

### Requirement: Packs list open routing for author work vs add-task-set

Choosing a pack from the packs list MUST open **Edit** when the caller’s author-facing state for that pack is never-published or pack-level **draft / pending / needs_revision**. When the caller’s only open work on an in-catalog pack is an **add-task-set** request (or never-live set draft), choosing the pack MUST open the **live** pack surface (answers and task-set list) first — not the add-task-set editor directly. Clean in-catalog packs without author-facing draft/pending/needs_revision MUST open live browse.

#### Scenario [SC-PACK-191]: Packs list with open add-task-set opens live first

- **GIVEN** in-catalog pack P and set author U with an open add-task-set request on P (and no pack-level never-published draft for U as pack editor)
- **WHEN** U chooses P from the packs list
- **THEN** the live pack surface opens (answers / task-set list)
- **AND** the add-task-set Edit surface does not open solely from that pack-row choice

#### Scenario [SC-PACK-192]: Pack-level pending opens Edit from list

- **GIVEN** creator A has pack P with author-facing pending or needs_revision for pack-level content (or never-published draft)
- **WHEN** A chooses P from the packs list
- **THEN** the pack editor (Edit) opens
