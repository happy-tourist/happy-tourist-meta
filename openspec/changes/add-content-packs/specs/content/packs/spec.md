# content/packs Specification

## Purpose

UGC-наборы карточек-ответов и заданий: создание и правка verified-пользователями, коллекция и публичный каталог одобренных наборов, модерация (очередь, тред, почта), блокировка; задел под будущие peek-награды по сложности 1–3. Без привязки к tourist-room и без замены peek-stub в этом change.

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

## ADDED Requirements

### Requirement: Verified registered user may create a pack

A non-anonymous user with a valid JWT and verified email MUST be able to create a content pack with a non-empty title and a description (description MAY be empty only if the product form allows blank; title MUST be non-empty). On create the pack MUST enter the creator’s collection and MUST start as not publicly listed until approved. Anonymous guests and unverified email/password users MUST NOT create packs; the client MUST prompt them to sign in or confirm email instead.

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

A pack MUST contain answer cards and one or more task sets. Each answer card MUST have textual content and MAY have a textual description. Each task MUST have a textual question, a difficulty of exactly `1`, `2`, or `3`, and one or more answer slots that reference answer cards from the same pack. Task sets MUST be labeled by the contributing user’s display identity (author and, when applicable, co-author labels are display-only and MUST NOT grant extra permissions beyond collection membership).

#### Scenario [SC-PACK-04]: Answer card has content and description

- **GIVEN** a verified user editing a pack in their collection
- **WHEN** the user adds an answer card with non-empty content and an optional description
- **THEN** the card is stored on that pack

#### Scenario [SC-PACK-05]: Task has question, difficulty, and slots

- **GIVEN** a verified user editing a pack that already has answer cards
- **WHEN** the user adds a task with a question, difficulty in `1`|`2`|`3`, and at least one slot filled from those answer cards
- **THEN** the task is stored in a task set attributed to that user

#### Scenario [SC-PACK-06]: Invalid difficulty is rejected

- **GIVEN** a verified user editing a pack
- **WHEN** the user submits a task with difficulty outside `1`|`2`|`3`
- **THEN** the system rejects the write

### Requirement: Answer-slot editing UX semantics

When composing a task, the editor MUST support adding an empty slot, removing a slot, filling the next empty slot by selecting an answer card in order, and clearing a filled slot by selecting that slot. Changing an answer card’s content that is referenced by task slots MUST clear those slot references (MUST NOT silently rewrite slot answers). Submit for moderation MUST be blocked while any task has an empty required slot or while fewer than the minimum answer cards or tasks exist.

#### Scenario [SC-PACK-07]: Changing a referenced answer card clears dependent slots

- **GIVEN** a task whose slots reference answer card A
- **WHEN** the editor changes the content of answer card A
- **THEN** those slots become empty
- **AND** submit for moderation remains blocked until the slots are filled again

#### Scenario [SC-PACK-08]: Submit requires minima and filled slots

- **GIVEN** a pack with fewer than two answer cards, or fewer than two tasks, or any task with an empty slot
- **WHEN** the user attempts to submit the pack for moderation
- **THEN** the system rejects the submit

#### Scenario [SC-PACK-09]: Submit succeeds at minima

- **GIVEN** a pack with at least two answer cards, at least two tasks, every task having at least one filled slot and valid difficulty, and the actor allowed to submit
- **WHEN** the user submits for moderation
- **THEN** the pack enters pending moderation
- **AND** a moderation thread exists for that change author and staff

### Requirement: Collection gates editing; anyone with session may collect approved packs

Only users who have the pack in their collection and who satisfy create/edit identity rules (non-anonymous, verified email) MUST be able to enter edit mode and submit changes. Any authenticated user (including anonymous and unverified) MUST be able to view an approved non-blocked pack in the public catalog and add it to their collection. Unauthenticated callers MUST be rejected for collection mutations.

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

### Requirement: Public catalog shows only latest approved live content

The public catalog and the public pack page MUST list and show only packs that have an approved live revision and are not blocked. Draft and pending content MUST NOT appear as the public live view. While a newer change is pending, the world MUST continue to see the last approved live revision.

#### Scenario [SC-PACK-13]: Pending changes do not replace live public view

- **GIVEN** pack P has an approved live revision and a pending change request
- **WHEN** any user opens the public catalog or public pack page for P
- **THEN** they see the approved live content
- **AND** they do not see the pending draft as live

#### Scenario [SC-PACK-14]: Never-approved pack is absent from catalog

- **GIVEN** a pack that has never been approved
- **WHEN** a user browses the public catalog
- **THEN** that pack is not listed

### Requirement: Moderation lock and single pending submit race

While a pack has an open pending moderation request, users other than that change’s author MUST NOT start a new edit session on the pack. The pending change’s author MUST be allowed to amend and resubmit within the same moderation thread until the request is approved, rejected (with thread continuing for fixes), or cancelled by staff. If two eligible editors race to submit, exactly one submit MUST succeed; the other MUST receive an error and that user’s work MUST remain as a personal draft while the pack stays in moderation.

#### Scenario [SC-PACK-15]: Non-author cannot start edit while pending

- **GIVEN** pack P is pending moderation under change author A
- **WHEN** verified user B (with P in collection) attempts to start editing P
- **THEN** the system rejects starting that edit

#### Scenario [SC-PACK-16]: Pending author may amend and resubmit

- **GIVEN** pack P is pending under author A
- **WHEN** A updates the pending draft and submits again
- **THEN** the same moderation thread remains the active request
- **AND** the pack stays pending

#### Scenario [SC-PACK-17]: Losing submit keeps personal draft during moderation

- **GIVEN** two eligible editors both prepared changes and the first successfully submitted P into pending
- **WHEN** the second attempts to submit
- **THEN** the system rejects the second submit
- **AND** the second user’s draft remains available until moderation ends (approve or staff cancel)

### Requirement: Staff moderation queue, decisions, and thread

Moderator and admin MUST be able to list packs awaiting moderation, open a read-only preview of the submitted content (full preview, not diff-only), approve, reject with a staff comment in the thread, or cancel the pending request (returning the pack to non-pending so others may edit again). The moderation thread MUST be visible only to the change author and to moderator|admin. After approval, accepted contributors MAY receive a co-author display label on relevant task sets; labels MUST NOT alone grant edit rights. A later improvement cycle MUST open a new moderation thread.

#### Scenario [SC-PACK-18]: Staff approves pending pack into catalog

- **GIVEN** a pending pack change and an actor with role moderator or admin
- **WHEN** the actor approves the change
- **THEN** the submitted content becomes the live approved revision
- **AND** the pack appears in the public catalog (unless blocked)
- **AND** the pack is no longer pending

#### Scenario [SC-PACK-19]: Staff rejects with comment

- **GIVEN** a pending pack change and staff actor
- **WHEN** the actor rejects with a non-empty comment
- **THEN** the change author can read the comment in the thread
- **AND** the author may amend and resubmit on the same thread

#### Scenario [SC-PACK-20]: Staff cancels pending

- **GIVEN** a pending pack change and staff actor
- **WHEN** the actor cancels the pending request
- **THEN** the pack is no longer pending
- **AND** other collection members may start editing again

#### Scenario [SC-PACK-21]: Non-staff cannot approve or cancel

- **GIVEN** an authenticated user with role `user`
- **WHEN** the user attempts to approve, reject, cancel, or block a pack
- **THEN** the system rejects the request

#### Scenario [SC-PACK-22]: New improvement opens a new thread

- **GIVEN** an approved pack and an eligible editor who submits a new change after a prior approval
- **WHEN** moderation starts for that new change
- **THEN** a new moderation thread is created (not a continuation of the old approved thread)

### Requirement: Author and staff may exchange messages on an open thread

While a moderation request is not cancelled and not finally closed by approval without further work, the change author and staff MUST be able to append messages to that thread (support-like chat). Other users MUST NOT read or write that thread.

#### Scenario [SC-PACK-23]: Change author posts a reply in the thread

- **GIVEN** an open moderation thread for author A
- **WHEN** A posts a non-empty message
- **THEN** the message appears in the thread for A and for staff

#### Scenario [SC-PACK-24]: Unrelated user cannot read the thread

- **GIVEN** an open moderation thread for author A
- **WHEN** another non-staff user requests the thread
- **THEN** the system rejects or returns no thread content

### Requirement: Staff may block a pack for everyone

Moderator and admin MUST be able to block a pack. A blocked pack MUST remain visible in catalog and collections with a clear blocked state and MUST NOT be editable or submittable. Unblock MUST be restricted to moderator|admin. Hard delete is out of scope.

#### Scenario [SC-PACK-25]: Blocked pack shows as blocked everywhere

- **GIVEN** an approved pack that staff blocks
- **WHEN** any user views the catalog, collection, or pack page
- **THEN** the pack is shown as blocked
- **AND** edit and submit are rejected

#### Scenario [SC-PACK-26]: Only staff may unblock

- **GIVEN** a blocked pack and a role `user` actor
- **WHEN** the actor attempts to unblock
- **THEN** the system rejects the attempt

### Requirement: Email notifications for moderation events

When the change author is a non-anonymous user with an email, the system MUST send Russian email notifications (same mail channel as support) for: staff approve, staff reject, new staff message on the thread, and staff block of the pack. Anonymous authors MUST NOT receive email. Links in mail MUST open the client SPA hash route for the pack / moderation view.

#### Scenario [SC-PACK-27]: Approve notifies the change author by email

- **GIVEN** a pending change by a registered author with email
- **WHEN** staff approves the change
- **THEN** the author receives an email about the approval
- **AND** the email links to the client SPA pack view

#### Scenario [SC-PACK-28]: Anonymous change author gets no email

- **GIVEN** moderation would notify but the identity is anonymous
- **WHEN** a notifiable moderation event occurs
- **THEN** no email is sent for that author

### Requirement: Client surfaces for catalog, collection, editor, and staff queue

The client MUST expose: a public catalog of approved non-secret packs; a collection list for the current user; a pack view (read-only live content for the public; editor with fields when allowed); and a staff moderation queue for moderator|admin. Loading, empty, and error states MUST be visible (banner/error pattern consistent with the app). Create/edit entry points MUST show the auth/verify modal when the user is ineligible.

#### Scenario [SC-PACK-29]: Catalog lists approved packs

- **GIVEN** at least one approved non-blocked pack exists
- **WHEN** an authenticated user opens the catalog
- **THEN** those packs are listed for browsing

#### Scenario [SC-PACK-30]: Staff queue lists pending packs

- **GIVEN** at least one pending pack and a moderator or admin session
- **WHEN** staff opens the moderation queue
- **THEN** pending packs are listed
- **AND** a role `user` session MUST NOT access that queue

#### Scenario [SC-PACK-31]: Ineligible create shows auth or verify prompt

- **GIVEN** a guest or unverified user on a create-pack entry point
- **WHEN** the user attempts to create
- **THEN** the client shows a prompt to sign in or confirm email
- **AND** no pack is created

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
