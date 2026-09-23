# content/packs Specification

## Purpose

UGC-наборы (**набор карточек** + задания): создание/правка verified-пользователями, коллекция, публичный каталог после approve ответов, **раздельная** модерация answers/tasks, видимые статусы и треды на страницах редактора, блокировка; задел difficulty 1–3 под будущие peek-награды. Без привязки к tourist-room и без замены peek-stub.

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
| SC-PACK-08 | covered |
| SC-PACK-09 | covered |
| SC-PACK-10 | covered |
| SC-PACK-11 | covered |
| SC-PACK-12 | covered |
| SC-PACK-13 | covered |
| SC-PACK-14 | covered |
| SC-PACK-15 | covered |
| SC-PACK-16 | covered |
| SC-PACK-17 | covered |
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
| SC-PACK-42 | covered |
| SC-PACK-43 | covered |
| SC-PACK-44 | covered |
| SC-PACK-45 | covered |
| SC-PACK-46 | covered |
| SC-PACK-47 | covered |
| SC-PACK-48 | covered |
| SC-PACK-49 | covered |
| SC-PACK-50 | covered |
| SC-PACK-51 | covered |
| SC-PACK-52 | covered |
| SC-PACK-53 | covered |
| SC-PACK-54 | covered |
| SC-PACK-55 | covered |
| SC-PACK-56 | covered |
| SC-PACK-57 | covered |
| SC-PACK-58 | covered |
| SC-PACK-59 | covered |
| SC-PACK-60 | covered |
| SC-PACK-61 | covered |
| SC-PACK-62 | covered |
| SC-PACK-63 | covered |
| SC-PACK-64 | covered |
| SC-PACK-65 | covered |
| SC-PACK-66 | covered |
| SC-PACK-70 | covered |
| SC-PACK-71 | covered |
| SC-PACK-72 | covered |

## ADDED Requirements

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

When composing a task, the editor MUST support adding an empty slot, removing a slot (minimum one slot remains), filling the next empty slot by selecting an answer card tile, and clearing a filled slot by selecting that slot. Changing an answer card’s content that is referenced by task slots MUST clear those slot references. Deleting an answer card MUST NOT by itself block answers submit. Tasks submit MUST be rejected while any submitted task has an empty slot or while fewer than two tasks exist. Answers submit MUST be rejected while fewer than two answer cards exist.

#### Scenario [SC-PACK-07]: Changing a referenced answer card clears dependent slots

- **GIVEN** a task whose slots reference answer card A
- **WHEN** the editor changes the content of answer card A
- **THEN** those slots become empty
- **AND** tasks submit remains blocked until the slots are filled again
- **AND** answers submit is not blocked solely because those slots are empty

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

Only users who have the pack in their collection and who satisfy create/edit identity rules (non-anonymous, verified email) MUST be able to enter edit mode and submit changes. The client MUST expose **Edit** from the user’s collection list **and** from the live pack page when that pack is in the user’s collection, subject to pending rules below. Any authenticated user (including anonymous and unverified) MUST be able to view an approved non-blocked pack in the public catalog and add it to their collection. The live pack response MUST report whether the caller already has the pack in their collection so the client can show an accurate collect state. Removing a pack from the user’s own collection MUST use a clear trash/delete affordance and MUST ask for confirmation before the mutation. Unauthenticated callers MUST be rejected for collection mutations.

While the pack has a pending `answers` or `tasks` moderation request authored by user A, the live (and collection) **Edit** control MUST be shown to A and MUST be hidden from other collection members. When there is no pending request of either type, any collection member who passes identity gates MAY see Edit. Ineligible users (anonymous / unverified) who somehow reach Edit MUST still get the auth/verify prompt and MUST NOT mutate.

#### Scenario [SC-PACK-10]: User without collection cannot edit

- **GIVEN** a verified user who does not have pack P in their collection
- **WHEN** the user attempts to edit or submit changes for P
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-11]: Guest may add an approved pack to collection

- **GIVEN** an authenticated anonymous user and an approved non-blocked pack in the public catalog
- **WHEN** the user adds the pack to their collection
- **THEN** the pack is in that user’s collection
- **AND** the guest still cannot create or edit packs

#### Scenario [SC-PACK-12]: Unauthenticated collection add is rejected

- **GIVEN** no auth session
- **WHEN** a collection-add request is made
- **THEN** the system rejects the request

#### Scenario [SC-PACK-53]: Live Edit when pack is in collection and idle

- **GIVEN** an authenticated user who has approved pack P in their collection and P has no pending answers or tasks request
- **WHEN** the live pack page for P renders
- **THEN** the page shows a primary Edit control that navigates into the editor
- **AND** Edit remains available from that user’s collection list

#### Scenario [SC-PACK-54]: Remove from collection asks for confirmation

- **GIVEN** an authenticated user with pack P in their collection
- **WHEN** the user chooses remove-from-collection
- **THEN** the client asks for confirmation before calling the remove API
- **AND** cancelling the dialog leaves P in the collection

#### Scenario [SC-PACK-61]: Live Edit hidden when not in collection

- **GIVEN** an authenticated verified user viewing live pack P that is **not** in their collection
- **WHEN** the live pack page renders
- **THEN** the page MUST NOT show a primary Edit control

#### Scenario [SC-PACK-62]: Live Edit hidden for non-author while pending

- **GIVEN** pack P is in user B’s collection and has a pending answers or tasks request authored by user A (A ≠ B)
- **WHEN** B opens the live pack page for P
- **THEN** Edit is hidden for B

#### Scenario [SC-PACK-63]: Pending author still sees Live Edit

- **GIVEN** pack P is in author A’s collection and has a pending answers or tasks request authored by A
- **WHEN** A opens the live pack page for P
- **THEN** Edit is shown so A MAY continue editing / resubmit

#### Scenario [SC-PACK-64]: Collect button reflects membership

- **GIVEN** an authenticated user who already has pack P in their collection
- **WHEN** the live pack page for P loads
- **THEN** the collect control shows an in-collection state and MUST NOT present a fresh “add” as if P were absent
- **AND** the live pack payload includes `inCollection: true` for that caller

#### Scenario [SC-PACK-65]: Collection trash icon and row click isolation

- **GIVEN** an authenticated user on the collection list with pack P
- **WHEN** the list renders
- **THEN** remove-from-collection uses a trash/delete icon (not a minus-only glyph)
- **AND WHEN** the user activates remove
- **THEN** confirmation is required before the API call
- **AND WHEN** the user clicks the row (not the action icons)
- **THEN** they navigate to the live view if P has live content (else editor)
- **AND** activating Edit or trash MUST NOT navigate via the row link instead of the intended action

#### Scenario [SC-PACK-66]: Collection Edit reaches editor when pack has live

- **GIVEN** an authenticated eligible user with pack P in their collection and P has live content
- **WHEN** the user activates Edit on the collection list row
- **THEN** the client opens the pack editor
- **AND** MUST NOT leave the user on the live view without editor fields due to competing row navigation

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

### Requirement: Dual pending locks and races

A pack MAY have at most one pending request per type (`answers`, `tasks`). While the editor’s answers draft is **dirty** relative to the last successful answers submit and there is **no** answers-pending request, no user MUST be allowed to create or edit **tasks**. While pack answers are **pending** under change author A, **only A** MUST be allowed to submit or resubmit answers; other users MUST NOT submit answers. While answers are pending under A, **A MAY** create and edit tasks even if A’s answers draft is dirty again; **other** users MUST NOT create, edit, or submit tasks. A MAY still submit tasks so staff can approve tasks before answers. While tasks are pending under change author T, only T MUST be allowed to submit or resubmit tasks; other users MUST NOT win a competing tasks submit (race → error; personal draft retained). Answers submit MUST NOT require live or pending tasks.

#### Scenario [SC-PACK-15]: Dirty answers without pending block task editing for everyone

- **GIVEN** pack P has dirty answers and no answers-pending request
- **WHEN** any verified collection member attempts to create or edit tasks for P
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-57]: Answers-pending author may edit tasks while answers dirty

- **GIVEN** pack P is answers-pending under author A and A’s answers draft is dirty again
- **WHEN** A creates or edits tasks for P
- **THEN** the system allows the write

#### Scenario [SC-PACK-58]: Non-author cannot edit tasks while answers pending

- **GIVEN** pack P is answers-pending under author A and verified collection member B ≠ A
- **WHEN** B attempts to create or edit tasks for P
- **THEN** the system rejects the attempt

#### Scenario [SC-PACK-16]: Pending answers author may amend and resubmit answers

- **GIVEN** pack P is answers-pending under author A
- **WHEN** A updates the answers draft and submits answers again
- **THEN** the same answers moderation thread remains active
- **AND** answers stay pending
- **AND** after that submit, answers are not dirty

#### Scenario [SC-PACK-17]: Losing tasks submit keeps personal draft

- **GIVEN** two eligible editors prepared tasks while answers were not dirty and no answers pending blocked them, and the first successfully submitted tasks into pending
- **WHEN** the second attempts to submit tasks
- **THEN** the system rejects the second submit
- **AND** the second user’s draft remains available until that tasks request is approved or cancelled

#### Scenario [SC-PACK-42]: Only answers pending author may resubmit answers

- **GIVEN** pack P is answers-pending under author A and verified collection member B ≠ A
- **WHEN** B attempts to submit answers
- **THEN** the system rejects the submit

#### Scenario [SC-PACK-43]: Other users cannot submit tasks while answers pending

- **GIVEN** pack P is answers-pending under author A and verified collection member B ≠ A with a valid tasks draft
- **WHEN** B attempts to submit tasks
- **THEN** the system rejects the submit
- **AND** A MAY still submit tasks while A’s answers remain pending

### Requirement: Staff moderation via answers hub

Moderator and admin MUST see a single queue of packs that have an **answers** pending request and/or a **tasks-only** pending request (tasks pending with no answers pending MUST appear). When both answers and tasks are pending for the same pack, the queue MUST list the pack once via the answers request (nested tasks as today). Opening a queue item MUST show the same hub layout: an answers/cards preview of the **full pack context** first, then a **list** of task sets with status marks — NOT a fully expanded dump of all questions. For answers-pending, answers come from the answers request revision; for tasks-only, answers MUST come from the current live answers (read-only context). Staff MUST open a nested tasks page to review questions/slots and to approve/reject/cancel **tasks**. Approve/reject/cancel **answers** and the answers thread MUST be available on the hub only when an answers request is pending; for tasks-only they MUST NOT be shown or MUST be disabled. When answers are also pending, staff MUST approve **tasks** before approving **answers**. Approving answers MUST require live tasks already present and MUST publish the pack to the public catalog (unless blocked). For tasks-only, approving tasks MUST update live tasks without requiring an answers approve. Staff MAY reject with comment, cancel, or message on the relevant request thread. Threads remain visible only to that request’s change author and staff. After approval of a type, a later cycle of the same type MUST open a new thread. Non-staff MUST NOT perform staff actions.

#### Scenario [SC-PACK-18]: Staff approves answers into catalog after live tasks

- **GIVEN** pack P has answers pending, live tasks already approved, and a staff actor
- **WHEN** the actor approves the answers request
- **THEN** the submitted answers become the live answers revision
- **AND** the pack appears in the public catalog (unless blocked)
- **AND** answers are no longer pending

#### Scenario [SC-PACK-19]: Staff rejects with comment

- **GIVEN** a pending answers or tasks request and staff actor
- **WHEN** the actor rejects with a non-empty comment
- **THEN** the change author can read the comment in that request’s thread
- **AND** the author may amend and resubmit on the same thread when locks allow

#### Scenario [SC-PACK-20]: Staff cancels pending

- **GIVEN** a pending answers or tasks request and staff actor
- **WHEN** the actor cancels that request
- **THEN** that request is no longer pending
- **AND** locks for that type are released accordingly

#### Scenario [SC-PACK-21]: Non-staff cannot approve or cancel

- **GIVEN** an authenticated user with role `user`
- **WHEN** the user attempts to approve, reject, cancel, or block a pack
- **THEN** the system rejects the request

#### Scenario [SC-PACK-22]: New improvement opens a new thread per type

- **GIVEN** an approved answers (or tasks) cycle and an eligible editor who submits a new change of the same type
- **WHEN** moderation starts for that new change
- **THEN** a new moderation thread is created for that type (not a continuation of the old approved thread)

#### Scenario [SC-PACK-30]: Staff queue lists answers-pending and tasks-only packs

- **GIVEN** pack A has answers pending and pack B has only tasks pending
- **WHEN** staff opens the moderation queue
- **THEN** pack A is listed
- **AND** pack B is listed for its tasks-only pending
- **AND** a role `user` session MUST NOT access that queue

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

#### Scenario [SC-PACK-45]: Author sees reject status and comment on editor page

- **GIVEN** staff rejected an answers (or tasks) request with a non-empty comment
- **WHEN** change author A opens the corresponding cards (or tasks) editing page
- **THEN** the page shows rejected / needs-revision status for that type
- **AND** the reject comment is readable in that type’s thread on the page

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

The client MUST expose: a collection list as the primary entry from lobby into the packs section (with Edit, trash remove-from-collection + confirm, and row→live/editor without stealing action clicks); a public catalog reachable from the collection; a live pack view with collect state from `inCollection` and with **Edit** when the pack is in the user’s collection subject to pending-author rules (SC-PACK-53/61–63); a **cards pack** editing page titled as a card pack / «Набор карточек» (pack title/description, single card form, cards list with edit affordance, nested task-set list labeled by author/coauthor with per-set needs-moderation marks when applicable, answers submit, answers status label, answers thread, and author delete-pack when unpublished); a **task-set** editing page nested under cards (single question form, slots, answer tiles, questions list, tasks submit, tasks status label, tasks thread, and author delete-task-set when the pack is unpublished) that follows D1′ dirty/pending rules; and a staff moderation queue that lists answers-pending and tasks-only-pending packs, whose hub shows answers/cards context first then a task-set list and opens a nested tasks page, hiding answers approve when tasks-only. Draft edits MUST autosave **without** a top-of-page «saving» caption that shifts layout; save/submit affordances MAY show button loading instead. Destructive deletes MUST ask for confirmation. Answers submit MUST be disabled when answers are not dirty or minima fail; tasks submit MUST be disabled when answers are dirty, when tasks are not dirty, or when minima fail. Adding/saving a question MUST require at least one filled answer slot. Create task-set MUST show a hover/tooltip hint when blocked. Loading, empty, and error states MUST be visible. Create/edit entry points MUST show the auth/verify modal when the user is ineligible. The public catalog MUST NOT list unapproved packs; moderation statuses appear on editor pages, not as catalog badges. Staff block/unblock controls MUST NOT be shown.

#### Scenario [SC-PACK-29]: Catalog lists approved packs

- **GIVEN** at least one pack with approved answers (catalog-eligible) exists
- **WHEN** an authenticated user opens the catalog
- **THEN** those packs are listed for browsing

#### Scenario [SC-PACK-31]: Ineligible create shows auth or verify prompt

- **GIVEN** a guest or unverified user on a create-pack entry point
- **WHEN** the user attempts to create
- **THEN** the client shows a prompt to sign in or confirm email
- **AND** no pack is created

#### Scenario [SC-PACK-41]: Lobby opens collection first

- **GIVEN** an authenticated user on the lobby
- **WHEN** the user opens the packs section navigation entry
- **THEN** the collection page is shown first
- **AND** the catalog remains reachable from the collection

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

#### Scenario [SC-PACK-48]: Changed task sets show needs-moderation mark

- **GIVEN** the editor changed task set S since the last successful tasks submit and has not submitted tasks again
- **WHEN** the user returns to the cards page task-set list (including after reload)
- **THEN** set S shows a needs-moderation mark
- **AND** unchanged sets do not show that mark solely for S’s edits

#### Scenario [SC-PACK-60]: Needs-moderation marks clear after tasks approval without further edits

- **GIVEN** author A submitted tasks, staff approved tasks, and A did not edit tasks after that submit
- **WHEN** A opens the cards page task-set list (including after reload)
- **THEN** those task sets MUST NOT show a needs-moderation mark
- **AND** the tasks status MAY show approved

#### Scenario [SC-PACK-49]: Status labels on cards and tasks pages

- **GIVEN** answers (or tasks) are pending, rejected, or approved for the editor’s pack
- **WHEN** the editor opens the cards (or tasks) page
- **THEN** the page shows the corresponding status label for that type (pending / rejected needs revision / approved as applicable)

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
