# content/maps Specification

## Purpose

UGC-карты поля 10×10 (старт / игровое / финиш / дыра): создание и paint-редактор, конфиг игроков×туристов, submit на модерацию с тредом автор↔staff, approve в общий список без коллекции; soft-unpublish и staff Edit после freeze автора. Без привязки к tourist-room.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-01 | covered (server mocha) |
| SC-MAP-02 | covered (server mocha) |
| SC-MAP-03 | covered (server mocha) |
| SC-MAP-04 | covered (client vitest paint) |
| SC-MAP-05 | covered (server mocha) |
| SC-MAP-06 | covered (server mocha + client vitest) |
| SC-MAP-07 | covered (server mocha + client vitest) |
| SC-MAP-08 | covered (client vitest) |
| SC-MAP-09 | covered (server mocha) |
| SC-MAP-10 | covered (server mocha) |
| SC-MAP-11 | covered (server mocha) |
| SC-MAP-12 | covered (server mocha) |
| SC-MAP-13 | covered (server mocha) |
| SC-MAP-14 | covered (server mocha + client vitest) |
| SC-MAP-15 | covered (server mocha) |
| SC-MAP-16 | covered (server mocha) |
| SC-MAP-17 | covered (server mocha + client vitest) |
| SC-MAP-18 | covered (server mocha) |
| SC-MAP-19 | covered (server mocha) |
| SC-MAP-20 | covered (server mocha) |
| SC-MAP-21 | covered (server mocha + client vitest) |
| SC-MAP-22 | covered (server mocha) |
| SC-MAP-23 | covered (server mocha) |
| SC-MAP-24 | covered (server mocha + client vitest) |
| SC-MAP-25 | covered (client vitest) |
| SC-MAP-26 | covered (server mocha) |
| SC-MAP-27 | covered (server mocha) |
| SC-MAP-28 | removed (superseded — maps wired at create) |
| SC-MAP-29 | covered (client vitest) |
| SC-MAP-30 | covered (client vitest) |
| SC-MAP-31 | covered (mocha) |
| SC-MAP-32 | covered (mocha) |
| SC-MAP-33 | covered (vitest) |
| SC-MAP-34 | covered (vitest) |
| SC-MAP-35 | covered (mocha) |
| SC-MAP-36 | covered (mocha) |
| SC-MAP-37 | covered (mocha) |
| SC-MAP-38 | covered (mocha) |
| SC-MAP-39 | covered (mocha) |
| SC-MAP-40 | covered (vitest) |
| SC-MAP-41 | covered (mocha/vitest) |
| SC-MAP-42 | covered (mocha/vitest) |
| SC-MAP-43 | covered (vitest) |
| SC-MAP-44 | covered (vitest) |
| SC-MAP-45 | covered (vitest) |
| SC-MAP-46 | covered (vitest) |
| SC-MAP-47 | covered (vitest) |
| SC-MAP-48 | covered (vitest) |
| SC-MAP-49 | covered (vitest) |
| SC-MAP-50 | covered (vitest) |
| SC-MAP-51 | covered (vitest) |
| SC-MAP-52 | covered (vitest) — strengthen: offset zone |
| SC-MAP-53 | covered (client vitest map editor paint/save) |
| SC-MAP-54 | covered (client vitest / page smoke layout) |
| SC-MAP-55 | covered (client vitest maps list cards) |
| SC-MAP-56 | covered (client vitest centered editor seats) |
| SC-MAP-60 | covered (server mocha — soft-unpublished reject) |
| SC-MAP-61 | covered (server mocha — create uses live map) |
| SC-MAP-62 | covered (client vitest staff boot) |
| SC-MAP-63 | covered (client vitest never-published Submit) |
| SC-MAP-64 | covered (server mocha staff-save clear working) |
| SC-MAP-65 | covered (client vitest dirty Submit) |

Related: room create — `lobby/rooms`; runtime board — `game/board`. Moderation patterns — `content/packs`; roles — `support/roles`; branding chrome — `ui/branding`. Staff published path / working clear / dirty Submit — SC-MAP-62…65.

## Requirements

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

When staff approves a map moderation request (after take), the submitted grid and seat config MUST become the live snapshot, the map MUST become **in catalog**, and it MUST appear on the public Maps list. After the map has been approved at least once, non-staff users **other than the creator** MUST NOT edit the live map. The **creator** MUST be able to Edit under the exclusive lock and submit further changes through moderation (see author re-edit). **Staff** (moderator|admin) MUST be able to Edit any map under exclusive lock and save directly to live/working **when** no open author map request blocks staff content edit and the lock is free. Never-approved maps remain editable by the creator (and staff).

#### Scenario [SC-MAP-16]: Approve lists map publicly

- **GIVEN** a pending map request and staff who has taken the request
- **WHEN** the actor approves
- **THEN** the map is in catalog
- **AND** any authenticated user sees it on the Maps list with mini preview, author, and players×tourists

#### Scenario [SC-MAP-17]: Author cannot edit after approve

- **GIVEN** map M approved into catalog and creator A (non-staff)
- **WHEN** A attempts a **direct live** edit/save without going through moderation submit
- **THEN** the system rejects applying those changes to live without moderation
- **AND** A MAY still acquire the edit lock, amend a working copy, and submit for moderation (see author re-edit)

#### Scenario [SC-MAP-18]: Staff edits under exclusive lock

- **GIVEN** approved map M with no open author request and free edit lock, and staff S
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

### Requirement: In-catalog maps drive tourist room create

A new `tourist` room create MUST accept an in-catalog map id and snapshot that map’s live grid and seat config (`players`, `touristsPerPlayer`) into the room per `lobby/rooms` / `game/board` / `game/pieces`. Soft-unpublished or never-published maps MUST NOT be usable for new creates. Creating, approving, or listing maps MUST NOT by itself mutate an already-running room’s snapshot.

#### Scenario [SC-MAP-61]: Room create applies an in-catalog map

- **GIVEN** an approved in-catalog map M
- **WHEN** a user creates a tourist room selecting M with valid task sets
- **THEN** the room’s play layout and capacity come from M’s live snapshot
- **AND** a map id is required at create

#### Scenario [SC-MAP-60]: Soft-unpublished map cannot create a room

- **GIVEN** map M is soft-unpublished
- **WHEN** a user attempts to create a tourist room selecting M
- **THEN** the system rejects create

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

### Requirement: Maps list shows moderation statuses

The Maps list MUST show status badges for **pending**, **needs_revision**, **draft** (never-published without open request or author draft after cancel), and soft-**unpublished** (staff) when applicable. Rows that are simply published / in catalog MUST **NOT** show an «В каталоге» / in_catalog status badge. A map with an open request MUST show pending or needs_revision rather than only a generic draft badge. Non-authors still MUST NOT see another user’s never-published maps.

#### Scenario [SC-MAP-31]: Pending map shows pending on Maps list

- **GIVEN** creator A submitted map M and the request is pending
- **WHEN** A opens the Maps list
- **THEN** M appears with a pending status indication

#### Scenario [SC-MAP-32]: Needs-revision map shows needs-revision on Maps list

- **GIVEN** creator A has map M with an open needs_revision request
- **WHEN** A opens the Maps list
- **THEN** M appears with a needs_revision status indication

### Requirement: Maps list filters

The Maps list MUST support filters: **all** (default); **on moderation** (caller’s own open pending or needs_revision map requests); **drafts** (caller’s never-published maps without an open request); **mine** (maps where the caller is `createdBy`). There MUST NOT be a favorites filter or star for maps. Guests MUST only see in-catalog maps; identity filters MUST be empty or unavailable for guests.

#### Scenario [SC-MAP-33]: Filter on moderation shows own pending maps

- **GIVEN** author A has map M pending and user B has map N pending
- **WHEN** A applies the on-moderation filter
- **THEN** A sees M and MUST NOT see N solely via that filter

#### Scenario [SC-MAP-34]: Filter mine shows own maps including drafts

- **GIVEN** creator A has in-catalog map M1 and never-published map M2
- **WHEN** A applies the mine filter
- **THEN** both M1 and M2 appear for A

### Requirement: Author may re-edit published map through moderation

After map M has been approved at least once, the creator MUST be able to acquire the exclusive edit lock, amend the working grid/config, and submit changes into a new or resumed map moderation request. Author changes MUST NOT apply directly to live without staff approve. While an open author map request exists, staff MUST NOT acquire content Edit or staff-save on M. Other non-staff MUST NOT edit M.

#### Scenario [SC-MAP-35]: Creator submits post-publish map edits

- **GIVEN** in-catalog map M created by A with no open author request
- **WHEN** A acquires the edit lock, amends the grid meeting start minima, and submits
- **THEN** an open map moderation request exists
- **AND** live catalog content remains the last approved snapshot until staff approve

#### Scenario [SC-MAP-36]: Staff cannot content-edit while author map request open

- **GIVEN** map M has an open author pending or needs_revision request
- **WHEN** staff S attempts to acquire content Edit lock or staff-save on M
- **THEN** the system rejects the action

### Requirement: Exclusive map edit lock includes creator

The map exclusive edit lock MUST be acquirable by the creator when Edit is allowed and by staff when staff Edit is allowed. While held (non-expired), a second actor MUST NOT acquire it. TTL MUST match the packs edit-lock policy. Staff MUST NOT acquire the lock while an open author request blocks staff content edit.

#### Scenario [SC-MAP-37]: Creator lock blocks staff Edit

- **GIVEN** creator A holds a non-expired edit lock on map M
- **WHEN** staff S attempts to acquire the lock
- **THEN** the system rejects with an edit-locked outcome

### Requirement: Staff must take a map moderation request before moderating

Moderator and admin MUST take an open map moderation request before approve, needs-revision, or cancel. Take MUST be available on the shared staff queue row and on the map request detail. While staff A holds a non-expired take, other staff MUST NOT take or moderate that request. Take TTL MUST match edit-lock TTL. Author amend/resubmit MUST remain allowed while take is held.

#### Scenario [SC-MAP-38]: Cannot approve map without take

- **GIVEN** open map request R and staff S who has not taken R
- **WHEN** S attempts to approve R
- **THEN** the system rejects the action

#### Scenario [SC-MAP-39]: Second staff cannot take map request

- **GIVEN** staff A holds take on open map request R
- **WHEN** staff B attempts to take or approve R
- **THEN** the system rejects the action

### Requirement: Non-staff have no author my-moderation navigation for maps

Non-staff users MUST NOT be shown a separate «На модерации» navigation entry from the Maps section. Their open map items MUST be reachable via the Maps list **on moderation** filter. Staff retain the shared staff queue (pack|map).

#### Scenario [SC-MAP-40]: Maps chrome hides my-moderation for non-staff

- **GIVEN** a non-staff authenticated user on the Maps section
- **WHEN** the user views Maps navigation chrome
- **THEN** the author «На модерации» control is not shown
- **AND** the on-moderation filter is available

### Requirement: Cancel returns author map work to draft on the Maps list

When staff or the change author cancels an open map moderation request, the system MUST keep the working grid/config and MUST NOT hard-delete the map. For the author, the map MUST appear as **draft** on the Maps list (including the drafts filter). Other users MUST continue to see the last approved **live** in-catalog snapshot when one exists. Author hard-delete of a never-published map removes it.

#### Scenario [SC-MAP-41]: Cancel yields author draft on Maps list

- **GIVEN** author A has an open pending map request on map M
- **WHEN** staff (after take) or A cancels that request
- **THEN** A sees M as draft on the Maps list
- **AND** working edits remain available for Edit

#### Scenario [SC-MAP-42]: Others keep live snapshot after cancel of post-publish map edits

- **GIVEN** in-catalog map M, author A’s open pending request cancelled, and non-author user B
- **WHEN** B opens the Maps list
- **THEN** B sees M with the live in-catalog snapshot
- **AND** A sees M as draft for retained working edits

### Requirement: Staff moderation entry is not on Maps list chrome

Moderator and admin MUST open the staff moderation queue from the shared application header, not from an embedded control on the Maps list chrome.

#### Scenario [SC-MAP-43]: Maps list has no staff moderation link

- **GIVEN** an authenticated moderator or admin on the Maps list
- **WHEN** the user views Maps list chrome
- **THEN** an embedded staff «Модерация» control is not shown on that list chrome

### Requirement: Maps breadcrumbs

Maps section surfaces MUST show breadcrumbs **below** the shared elevated header chrome in the page/layout zone that receives header offset (e.g. Lobby / Maps / map title), consistent with packs chrome — not inside the elevated header bar and not covered by the fixed header.

#### Scenario [SC-MAP-44]: Breadcrumbs on Maps list and editor

- **GIVEN** an authenticated user on the Maps list or map editor
- **WHEN** the page renders
- **THEN** breadcrumbs include a path back toward Lobby and Maps

#### Scenario [SC-MAP-52]: Map breadcrumbs sit below elevated header

- **GIVEN** an authenticated user on a Maps list or editor route with breadcrumbs
- **WHEN** the page renders
- **THEN** breadcrumbs are outside the elevated shared header bar
- **AND** breadcrumbs sit in the page/layout zone offset for the header (not covered by the fixed header)

### Requirement: No in_catalog status badge on Maps list

Published in-catalog maps MUST appear on the Maps list without an «В каталоге» / in_catalog status badge. Other status badges remain when applicable.

#### Scenario [SC-MAP-45]: Published map has no in_catalog badge

- **GIVEN** an approved in-catalog map M with no open author-facing draft/pending/needs_revision mark for the caller
- **WHEN** the caller opens the Maps list
- **THEN** M has no «В каталоге» / in_catalog status badge

### Requirement: Published map opens View without paint tools

Choosing a **clean** published / in-catalog map (no author-facing draft / pending / needs_revision for the caller) MUST open a **View** surface first. View MUST show the author and players×tourists (and MAY show a read-only grid preview). View MUST NOT show map paint-tool controls or other bottom edit chrome. **Edit** MUST be available from the Maps list row and from inside View in the **title row** (same placement pattern as pack live Edit); activating Edit acquires the exclusive edit lock and enters edit mode. While a user holds a non-expired edit lock, another eligible editor MUST NOT acquire it.

#### Scenario [SC-MAP-46]: Published map row opens View without tools

- **GIVEN** an in-catalog map M with no author-facing draft/pending/needs_revision for the caller
- **WHEN** a user chooses M from the Maps list (row open, not Edit)
- **THEN** View opens
- **AND** paint-tool controls are not shown

#### Scenario [SC-MAP-47]: Map View shows author and seat meta

- **GIVEN** user U is on View for in-catalog map M
- **WHEN** the View renders
- **THEN** M’s author and players×tourists are shown

#### Scenario [SC-MAP-48]: Edit from list or View enters locked edit

- **GIVEN** in-catalog map M with a free edit lock and eligible editor E
- **WHEN** E activates Edit from the list or from View
- **THEN** edit mode opens under E’s exclusive lock
- **AND** paint tools are available to E as allowed by edit rules

#### Scenario [SC-MAP-51]: View Edit control is in the title row

- **GIVEN** eligible editor E is on View for in-catalog map M
- **WHEN** View renders
- **THEN** the Edit control appears in the title/actions row (not below author/seats meta alone)

### Requirement: Never-published map opens Edit directly

Choosing a never-published map from the Maps list MUST open Edit directly (not View-first).

#### Scenario [SC-MAP-49]: Never-published map row opens Edit

- **GIVEN** creator A has never-published map M
- **WHEN** A chooses M from the Maps list
- **THEN** Edit opens (not View-first)

### Requirement: Author map draft or open moderation opens Edit

Choosing a map from the Maps list when the caller’s author-facing status is **draft**, **pending**, or **needs_revision** MUST open **Edit** directly (not View-first), including after Cancel-to-draft and while open moderation is in progress.

#### Scenario [SC-MAP-50]: Author pending map opens Edit from list

- **GIVEN** creator A has map M with author-facing pending or needs_revision (or draft after cancel)
- **WHEN** A chooses M from the Maps list
- **THEN** Edit opens (not View-first)

### Requirement: Map paint does not lose cells during quiet save

While the map editor is applying a quiet or staff save of the working grid, the client MUST NOT allow a later paint to be discarded when an in-flight save response returns. The client MUST either ignore further cell paints until the in-flight save settles, or apply save responses only when they are not older than the local grid the user has painted since that request started. Rapid consecutive paints MUST leave every accepted paint visible after saves complete.

#### Scenario [SC-MAP-53]: Second paint survives first save round-trip

- **GIVEN** an editable map editor with an in-progress quiet or staff save after painting cell A
- **WHEN** the user paints cell B before that save response is applied
- **THEN** after saves settle both cell A and cell B remain painted as the user left them
- **AND** the editor MUST NOT replace the local grid with an older server snapshot that omits cell B

### Requirement: Map editor field size and under-map palette

On the map **edit** surface (not View-only), the lined 10×10 field MUST be presented at a size comparable to the in-game board (a large square usable for painting, not a small side thumbnail). The three paint tools (start, task, finish) MUST appear as a palette **below** the field. Each tool MUST show a tile-like control with its label **under** that tile. Seat config controls MAY remain nearby but MUST NOT displace the under-map palette as the primary tool chrome.

#### Scenario [SC-MAP-54]: Palette under board-sized editor field

- **GIVEN** a verified editor on the map edit surface
- **WHEN** the user views the paint chrome
- **THEN** the map field is shown at a large board-comparable size
- **AND** start, task, and finish tools appear below the field
- **AND** each tool has a visible label under its tile

### Requirement: Maps list uses card tiles with mini preview

The Maps section list MUST render each map as a card in a wrapping row. Each card MUST show a **mini preview** of the grid in the upper area and the seat config as **players×tourists** below that preview. Author identity and status chrome MAY appear on the card. Action controls (Edit, soft-unpublish, republish, and similar) MUST appear at the **bottom** as stacked full-width **text** buttons when available. Existing open/navigation and visibility rules for unpublished maps MUST remain.

#### Scenario [SC-MAP-55]: Maps list renders cards with mini preview and capacity

- **GIVEN** an authenticated user on the Maps section with at least one listed map
- **WHEN** the user views the list
- **THEN** each map is shown as a card
- **AND** the card shows a mini grid preview above the players×tourists capacity
- **AND** Edit when available is a bottom full-width text control

### Requirement: Map editor column is centered with usable seat selects

On the map **edit** surface, the editor column that contains the field, under-map palette, and seat-count controls MUST be **horizontally centered** on the page (comparable to how the in-game board is presented). The players and tourists-per-player selects MUST remain usable at a readable width and MUST NOT appear collapsed into unusably narrow controls.

#### Scenario [SC-MAP-56]: Editor column centered and seat selects usable

- **GIVEN** a verified editor on the map edit surface
- **WHEN** the user views the editor chrome
- **THEN** the field-plus-palette-plus-seat column is centered horizontally
- **AND** the players and tourists selects are wide enough to read and operate

### Requirement: Staff edit published maps without moderation queue

When a moderator or admin opens Edit on a map that already has live content (`hasLive`), the client MUST enter the staff direct-edit path (staff-save) even if that staff user is the map `createdBy`. The staff Edit surface MUST NOT show «На модерацию» / Submit. Successful staff-save MUST update live content without creating a moderation request. For a **never-published** map (`!hasLive`), staff who is the creator MUST retain the creator Submit-to-moderation path (draft badge and queue submit remain as today).

#### Scenario [SC-MAP-62]: Staff creator on published map has no Submit

- **GIVEN** a published in-catalog map created by user S who is staff (moderator|admin)
- **WHEN** S opens staff Edit on that map
- **THEN** the editor is in staff direct-edit mode
- **AND** the «На модерацию» control is not shown
- **AND** saves use staff-save into live without opening a moderation request

#### Scenario [SC-MAP-63]: Staff creator on never-published map keeps Submit

- **GIVEN** a never-published map created by staff user S
- **WHEN** S opens the map editor
- **THEN** the «На модерацию» control remains available subject to existing start-cell minima and lock rules
- **AND** successful submit creates or updates an author moderation request as for non-staff creators

### Requirement: Map staff-save clears retained working copy

When staff saves a map to live via staff-save and there is **no** open author map moderation request (`pending` or `needs_revision`) on that map, the system MUST clear the map’s retained working revision pointer so list status for the creator MUST NOT show author-facing **draft** solely from a leftover working≠live copy. Open author pending/needs_revision MUST still block staff content Edit as today (SC-MAP-36).

#### Scenario [SC-MAP-64]: Staff-save removes hanging map draft

- **GIVEN** a published map with no open author request
- **AND** a retained working revision that differs from live
- **WHEN** staff holds the edit lock and saves via staff-save
- **THEN** live reflects the staff save
- **AND** the retained working pointer is cleared (or otherwise no longer yields draft)
- **AND** subsequent Maps list status for the creator does not show draft solely from that prior working copy

### Requirement: Map author Submit disabled until content is dirty

On the map editor when the creator (non-staff path) sees «На модерацию», Submit MUST be disabled while the in-memory grid/seats are unchanged relative to the content loaded for that edit session, even when start-cell minima are already met. After the user paints or changes seats, Submit MUST become enabled when minima and existing lock/pending-other rules allow. Staff direct-edit surfaces without Submit are unaffected.

#### Scenario [SC-MAP-65]: Map Submit disabled on open without edits

- **GIVEN** a verified creator opens map Edit with a grid that already meets start-cell minima
- **AND** the creator has not changed the loaded grid or seats
- **WHEN** the client renders «На модерацию»
- **THEN** the control is disabled
- **AND GIVEN** the creator paints a cell or changes seats
- **WHEN** minima and lock rules still allow submit
- **THEN** «На модерацию» is enabled
