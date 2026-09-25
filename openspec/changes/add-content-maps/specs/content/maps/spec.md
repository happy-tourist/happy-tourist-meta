## Purpose

UGC-карты поля 10×10 (старт / игровое / финиш / дыра): создание и paint-редактор, конфиг игроков×туристов, submit на модерацию с тредом автор↔staff, approve в общий список без коллекции; soft-unpublish и staff Edit после freeze автора. Без привязки к tourist-room.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-01 | pending |
| SC-MAP-02 | pending |
| SC-MAP-03 | pending |
| SC-MAP-04 | pending |
| SC-MAP-05 | pending |
| SC-MAP-06 | pending |
| SC-MAP-07 | pending |
| SC-MAP-08 | pending |
| SC-MAP-09 | pending |
| SC-MAP-10 | pending |
| SC-MAP-11 | pending |
| SC-MAP-12 | pending |
| SC-MAP-13 | pending |
| SC-MAP-14 | pending |
| SC-MAP-15 | pending |
| SC-MAP-16 | pending |
| SC-MAP-17 | pending |
| SC-MAP-18 | pending |
| SC-MAP-19 | pending |
| SC-MAP-20 | pending |
| SC-MAP-21 | pending |
| SC-MAP-22 | pending |
| SC-MAP-23 | pending |
| SC-MAP-24 | pending |
| SC-MAP-25 | pending |
| SC-MAP-26 | pending |
| SC-MAP-27 | pending |
| SC-MAP-28 | pending |
| SC-MAP-29 | pending |
| SC-MAP-30 | pending |

Related: moderation patterns — `content/packs`; roles — `support/roles`. Runtime board geometry — `game/board` (unchanged; maps not wired to rooms).

## ADDED Requirements

### Requirement: Verified user may create a map

A non-anonymous user with a valid JWT and verified email MUST be able to create a content map. On create the map MUST start unpublished (`inCatalog` false) with a working grid (initially all empty/hole cells) and default seat config of **1** player and **1** tourist per player. After create the client MUST open the map editing surface. Anonymous guests and unverified email/password users MUST NOT create maps; the client MUST prompt them to sign in or confirm email instead. Creating a map MUST NOT add it to any personal collection (maps have no collection).

#### Scenario [SC-MAP-01]: Verified user creates a map

- **GIVEN** an authenticated non-anonymous user with verified email
- **WHEN** the user creates a map
- **THEN** the map is stored unpublished
- **AND** the client opens the map editor for that map
- **AND** the map is not listed as approved for other users

#### Scenario [SC-MAP-02]: Guest cannot create a map

- **GIVEN** an authenticated anonymous user
- **WHEN** the user attempts to create a map
- **THEN** the system rejects creation
- **AND** the client prompts sign-in or equivalent instead of creating

#### Scenario [SC-MAP-03]: Unverified email cannot create a map

- **GIVEN** an authenticated non-anonymous user with unverified email
- **WHEN** the user attempts to create a map
- **THEN** the system rejects creation
- **AND** the client prompts email confirmation

### Requirement: Map grid and paint editor

A map MUST represent a **10-by-10** grid. Each cell MUST be exactly one of: empty/hole, start, task (brown playable field), or finish. The editor MUST show a lined 10×10 field and a bottom palette of three tools: start, task, finish. Selecting a palette tool MUST outline that tool; subsequent cell clicks MUST paint with the selected tool. Clicking a cell that already has the selected type MUST clear it to empty/hole; clicking with a different selected type MUST replace the cell contents. The editor MUST expose controls for **players** and **tourists per player**, each an integer from **1** to **4** inclusive. The system MUST persist working-copy edits for the creator while the map is never-published (quiet save). Title and description fields MUST NOT be required (maps have none in this capability).

#### Scenario [SC-MAP-04]: Paint places, clears, and replaces

- **GIVEN** a verified creator editing an unpublished map with start tool selected
- **WHEN** the user clicks an empty cell
- **THEN** that cell becomes a start cell
- **WHEN** the user clicks that same start cell again with start still selected
- **THEN** the cell becomes empty/hole
- **WHEN** the user selects the task tool and clicks a start cell
- **THEN** the cell becomes a task cell

#### Scenario [SC-MAP-05]: Players and tourists bounds

- **GIVEN** a verified creator editing an unpublished map
- **WHEN** the user sets players or tourists per player outside 1…4
- **THEN** the system rejects the value (client and server)

### Requirement: Maps list without collection

The client MUST expose a **Maps** section (reachable from the lobby) showing a shared list of **approved in-catalog** maps to any authenticated user (including anonymous). Each list row MUST show a **mini preview** of the grid, the **author** display identity, and the seat config as **players×tourists**. The same Maps page MUST additionally list the **creator’s own unpublished** maps (never approved, or soft-unpublished only for staff visibility rules below) so the author can reopen the editor before and without a moderation submit. Non-authors MUST NOT see another user’s unpublished maps on that list. There MUST be a **Create map** affordance at the top of the Maps section for eligible users. The system MUST NOT offer add-to-collection or remove-from-collection for maps.

#### Scenario [SC-MAP-06]: Approved maps visible to all sessions

- **GIVEN** an approved in-catalog map M and any authenticated user U (including anonymous)
- **WHEN** U opens the Maps list
- **THEN** M appears with mini preview, author, and players×tourists

#### Scenario [SC-MAP-07]: Author sees own unpublished on Maps

- **GIVEN** creator A has unpublished map M that was never approved
- **WHEN** A opens the Maps list
- **THEN** M appears for A and is enterable for edit
- **AND** another non-staff user B does not see M on the Maps list

#### Scenario [SC-MAP-08]: No collection for maps

- **GIVEN** an approved map M
- **WHEN** any user views Maps or the map detail
- **THEN** no collect / uncollect affordance is offered

### Requirement: Submit requires minimum start cells

The creator of a never-published map MUST be able to submit it for moderation when verified. Submit MUST be rejected unless the number of **start** cells on the working grid is greater than or equal to **players × tourists per player**. Submit MUST NOT require connectivity, side distribution, or a fixed finish block. On successful submit the system MUST open a moderation request in **pending** status and the map MUST appear in the author’s **«На модерации»** list. Until first approve, non-staff other than the creator MUST NOT edit the map.

#### Scenario [SC-MAP-09]: Submit rejected when starts below minimum

- **GIVEN** a map with players=2 and tourists=3 and fewer than 6 start cells
- **WHEN** the creator submits for moderation
- **THEN** the system rejects the submit
- **AND** no pending moderation request is created

#### Scenario [SC-MAP-10]: Submit succeeds at minimum starts

- **GIVEN** a map with players=2 and tourists=3 and at least 6 start cells
- **WHEN** the creator submits for moderation
- **THEN** a pending moderation request exists
- **AND** the map appears in the author’s «На модерации» list

### Requirement: Author and staff moderation thread

While a map moderation request is open (**pending** or **needs_revision**), the change author and staff MUST be able to append messages to that thread under the map moderation view. Other users MUST NOT read or write that thread. Staff reject with a non-empty comment MUST set status to **needs_revision** (product: «доработать»), keep the request open, and make the comment visible to the author on the map editor/moderation surface. The author MAY amend the working grid/config and resubmit on the same open request when locks allow. Staff MAY approve a needs_revision request without a new author submit (submitted revision becomes live). Cancel MUST close the open request without publishing.

#### Scenario [SC-MAP-11]: Author and staff exchange thread messages

- **GIVEN** an open map moderation request for author A
- **WHEN** A or staff posts a message on that thread
- **THEN** both A and staff can read the message
- **AND** other non-staff users cannot

#### Scenario [SC-MAP-12]: Staff needs-revision with comment

- **GIVEN** a pending map request and a staff actor
- **WHEN** the actor rejects with a non-empty comment
- **THEN** status is needs_revision
- **AND** the author can read the comment in the thread
- **AND** the map remains in author «На модерации» and staff queue

#### Scenario [SC-MAP-13]: Author resubmits after needs-revision

- **GIVEN** author A has a needs_revision map request and amends the grid so start minima are met
- **WHEN** A resubmits
- **THEN** the request returns to pending with the amended payload for staff review

### Requirement: Staff shared moderation queue includes maps

Moderator and admin MUST see map open requests (**pending** or **needs_revision**) in the **same** staff content moderation queue as pack requests. Each queue row MUST indicate the content type (**pack** vs **map**). Opening a map queue item MUST show the map preview (grid + players×tourists + author) and approve / needs-revision / cancel controls for that map request. Non-staff MUST NOT approve, needs-revision, or cancel map moderation.

#### Scenario [SC-MAP-14]: Staff queue lists map with type badge

- **GIVEN** an open map moderation request and a pack open request
- **WHEN** staff opens the shared moderation queue
- **THEN** both appear
- **AND** the map row is distinguishable as type map

#### Scenario [SC-MAP-15]: Non-staff cannot approve maps

- **GIVEN** a non-staff user and a pending map request
- **WHEN** the user attempts to approve, needs-revision, or cancel
- **THEN** the system rejects the action

### Requirement: Approve publishes into Maps list and freezes author edit

When staff approves a map moderation request, the submitted grid and seat config MUST become the live snapshot, the map MUST become **in catalog**, and it MUST appear on the public Maps list. After the map has been approved at least once, non-staff users including the creator MUST NOT edit the live map or submit further author edits. **Staff** (moderator|admin) MUST be able to Edit any map (in-catalog or soft-unpublished) under an exclusive staff edit lock and save directly to live/working without a new moderation queue cycle for that staff save. Never-approved maps remain editable only by the creator (and staff).

#### Scenario [SC-MAP-16]: Approve lists map publicly

- **GIVEN** a pending map request and a staff actor
- **WHEN** the actor approves
- **THEN** the map is in catalog
- **AND** any authenticated user sees it on the Maps list with mini preview, author, and players×tourists

#### Scenario [SC-MAP-17]: Author cannot edit after approve

- **GIVEN** map M approved into catalog and creator A (non-staff)
- **WHEN** A attempts to edit or submit changes to M
- **THEN** the system rejects the edit/submit for A

#### Scenario [SC-MAP-18]: Staff edits under exclusive lock

- **GIVEN** approved map M and staff S
- **WHEN** S acquires the edit lock and saves grid or seat config changes
- **THEN** the live map reflects those changes without a new author moderation submit
- **AND** a second staff actor cannot acquire the lock while S holds it

### Requirement: Author may delete unpublished maps

The creator MUST be able to hard-delete a map that has **never** been approved into the catalog. Delete MUST fail for maps that have a live/approved snapshot. Staff soft-unpublish is not a substitute for author delete of never-published maps.

#### Scenario [SC-MAP-19]: Creator deletes never-published map

- **GIVEN** creator A and never-approved map M
- **WHEN** A deletes M
- **THEN** M is removed
- **AND** M no longer appears on A’s Maps list or «На модерации»

#### Scenario [SC-MAP-20]: Cannot delete after approve

- **GIVEN** map M that was approved at least once
- **WHEN** the creator attempts to delete M
- **THEN** the system rejects the delete

### Requirement: Staff soft-unpublish and republish maps

Moderator and admin MUST be able to soft-unpublish an in-catalog map and republish it without a new moderation request. Soft-unpublish MUST hide the map from the **public** Maps list while keeping live content stored. Staff MUST still see soft-unpublished maps with a clear unpublished state and MAY Edit under lock. Non-staff MUST NOT open the live/public view of a soft-unpublished map. Soft-unpublish of an in-catalog map MUST cascade-cancel every **open** map moderation request for that map (`pending` or `needs_revision`); those requests MUST leave author «На модерации» and the staff queue. Soft-unpublish when already unpublished MUST be a no-op without re-cancel or re-notify. Republish MUST restore catalog visibility and MUST NOT restore cancelled requests. The staff confirm dialog for unpublish MUST warn that open moderation requests will be cancelled.

#### Scenario [SC-MAP-21]: Soft-unpublish hides from public Maps

- **GIVEN** in-catalog map M and staff S
- **WHEN** S soft-unpublishes M
- **THEN** non-staff users no longer see M on the public Maps list
- **AND** staff still see M as unpublished

#### Scenario [SC-MAP-22]: Soft-unpublish cascade-cancels open requests

- **GIVEN** in-catalog map M with an open moderation request by author A
- **WHEN** staff soft-unpublishes M
- **THEN** that request is cancelled
- **AND** it no longer appears in A’s «На модерации» or the staff queue

#### Scenario [SC-MAP-23]: Republish does not restore cancelled requests

- **GIVEN** map M was soft-unpublished while open requests were cascade-cancelled
- **WHEN** staff republishes M
- **THEN** M is in catalog again
- **AND** the cancelled requests remain cancelled

### Requirement: Author «На модерации» includes maps

A non-staff user’s **«На модерации»** list MUST include maps where the caller is the change author of an open (`pending` or `needs_revision`) map request — one row per map, opening the map editor/moderation surface. Staff MUST NOT be shown the author «На модерации» nav control (same rule as packs). Pack and map open items MAY share the same author list surface with a type distinction.

#### Scenario [SC-MAP-24]: Author sees pending map in my-moderation

- **GIVEN** author A with a pending map request
- **WHEN** A opens «На модерации»
- **THEN** that map appears and opens the map editor/thread

#### Scenario [SC-MAP-25]: Staff does not see author my-moderation nav

- **GIVEN** a staff user on content navigation
- **WHEN** the staff user views nav for author my-moderation
- **THEN** the «На модерации» author control is not shown

### Requirement: Email notifications for map moderation events

When the change author is a non-anonymous user with an email, the system MUST send Russian email notifications (same mail channel as content packs) for: staff approve, staff needs-revision, new staff message on the thread, and soft-unpublish cascade-cancel of the author’s open map request(s) (one email per author per unpublish event, **without** deep-link URLs for cascade-cancel). Approve / needs-revision / staff-message mails MUST deep-link to the client SPA map editor/moderation view. Anonymous authors MUST NOT receive email.

#### Scenario [SC-MAP-26]: Approve notifies author by email

- **GIVEN** a non-anonymous author with email and a pending map request
- **WHEN** staff approves that request
- **THEN** the author receives a Russian email linking to the map moderation/editor view

#### Scenario [SC-MAP-27]: Soft-unpublish cascade-cancel mail has no links

- **GIVEN** non-anonymous author A with an open map request cancelled by soft-unpublish
- **WHEN** staff soft-unpublishes that map
- **THEN** A receives exactly one Russian email stating soft-unpublish and cancellation
- **AND** the email contains no URL

### Requirement: Maps are not used by tourist rooms in this capability

Creating, approving, or listing maps MUST NOT change tourist-room create options, synced board layout, or move validation. The authoritative play layout remains the existing fixed tourist board until a later capability wires map selection.

#### Scenario [SC-MAP-28]: Room create ignores content maps

- **GIVEN** one or more approved maps exist
- **WHEN** a user creates a tourist room
- **THEN** the room still uses the fixed tourist layout
- **AND** no map id is required or applied

### Requirement: Maps UI chrome and errors

Maps pages MUST use loading / empty / error presentation consistent with other content surfaces (store error + banner). The map editor MUST show the moderation thread status and messages when an open request exists. Soft-unpublished labeling for staff MUST be clear («снято с публикации» or equivalent product copy).

#### Scenario [SC-MAP-29]: Editor shows open thread

- **GIVEN** creator A with a pending or needs_revision map request
- **WHEN** A opens the map editor
- **THEN** the request status and thread messages are visible
- **AND** A may reply when the request is open

#### Scenario [SC-MAP-30]: Error banner on list failure

- **GIVEN** the Maps list request fails
- **WHEN** the user views the Maps page
- **THEN** an error banner is shown from the store error
- **AND** the page does not pretend the list succeeded
