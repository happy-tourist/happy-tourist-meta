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

Moderator and admin MUST be able to open Edit on any pack (published or not) **without** requiring the pack in their collection. Staff content changes (including add card / add task set / edit / delete within policy) MUST apply **directly to live or to the single working copy** without entering the moderation queue. The client MUST NOT show staff a «submit for moderation» control for their own staff edits. While staff A holds an edit lock on pack P, staff B’s Edit attempt MUST fail with an error. The lock MUST release when A leaves the edit session or after a short timeout.

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

## MODIFIED Requirements

### Requirement: Collection gates editing; anyone with session may collect approved packs

Only the **creator** MAY edit an **unpublished** pack’s working copy (cards and task sets) and submit first publish, subject to verified non-anonymous identity. After the pack is in the public catalog, non-staff users MUST NOT enter full Edit of existing cards or task sets from collection or live; verified collectors MAY only use the **add task set** flow. **Staff** (moderator/admin) MUST be able to Edit any pack without collection membership, subject to the exclusive staff lock. Any authenticated user (including anonymous and unverified) MUST be able to view an approved non-blocked pack in the public catalog and add it to their collection. The live pack response MUST report `inCollection`. Removing a pack from the user’s own collection MUST use a clear trash affordance and confirmation. Unauthenticated callers MUST be rejected for collection mutations.

#### Scenario [SC-PACK-10]: User without collection cannot edit

- **GIVEN** a verified non-staff user who does not have published pack P in their collection
- **WHEN** the user attempts to edit existing content or submit an add-task-set for P
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

- **GIVEN** an authenticated user on the collection list with pack P
- **WHEN** the list renders
- **THEN** remove-from-collection uses a trash/delete icon (not a minus-only glyph)
- **AND WHEN** the user activates remove
- **THEN** confirmation is required before the API call
- **AND WHEN** the user clicks the row (not the action icons)
- **THEN** they navigate to the live view if P has live content (else creator editor when unpublished and caller is creator)
- **AND** activating action icons MUST NOT navigate via the row link instead of the intended action

#### Scenario [SC-PACK-66]: Collection Edit reaches editor when pack has live

- **GIVEN** a verified user with published pack P in their collection
- **WHEN** the user activates the collection contribution control for P (add-task-set; not full Edit of live cards)
- **THEN** the client opens the add-task-set flow
- **AND** MUST NOT open a full cards/tasks editor of existing live content for non-staff

## REMOVED Requirements

### Requirement: Dual pending locks and races

**Reason:** Product removes concurrent personal drafts and dual answers|tasks submit races among collection members. First publish and add-task-set use single-request flows with needs-revision / Cancel instead.

**Migration:** Delete dual dirty/pending lock rules and scenarios SC-PACK-15/16/17/42/43/57/58 behavior; replace with ADDED first-submit / add-task-set / staff-lock requirements. Drop per-user `content_user_drafts` and any retain-draft-on-losing-submit behavior.
