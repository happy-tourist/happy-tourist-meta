# content/packs — delta: unified list, favorites, author re-edit, moderation take, set statuses, cancel→draft, crumbs, ghost add-task-set, author draft→Edit

Базовый канон: `openspec/specs/content/packs/spec.md`. Change: коллекция→единый список, избранное, author re-edit, take; follow-up: статусы сетов без fan-out, ghost never-live set для автора, author draft→Edit, крошки под elevated header, staff из шапки (`ui/branding`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
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
| SC-PACK-179 | covered (mocha/vitest) |
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

Related: `content/maps`; `ui/branding` (header/crumbs); `game/leave`; roles — `support/roles`. Tourist-room wiring still out of scope.

## ADDED Requirements

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

## MODIFIED Requirements

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

## REMOVED Requirements

### Requirement: Default published packs are granted once to every user

**Reason:** Collection membership is removed as a product concept; default auto-grant into collections is obsolete.

**Migration:** Stop startup/create-user grants; remove or ignore `DEFAULT_CONTENT_PACK_IDS` grant behavior. In-catalog packs remain visible on the unified list to everyone. Existing `content_pack_collections` rows MAY be dropped or left unused; UI MUST NOT expose collection.

#### Scenario [SC-PACK-170]: No auto-grant on user create

- **GIVEN** any default-pack configuration that previously granted into collections
- **WHEN** a new user is created
- **THEN** the system MUST NOT add collection memberships solely by default-grant
- **AND** in-catalog packs remain listed on the unified packs list without membership
