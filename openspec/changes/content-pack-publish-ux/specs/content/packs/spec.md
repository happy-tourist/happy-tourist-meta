# content/packs — delta (content-pack-publish-ux)

Базовый канон: `openspec/specs/content/packs/spec.md` (dual submit, D1′, cascade, staff hub). Этот delta добавляет UX публикации, слоты на live/staff, stale-draft pull, staff unpublish/republish.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-85 | pending |
| SC-PACK-86 | pending |
| SC-PACK-87 | pending |
| SC-PACK-88 | pending |
| SC-PACK-89 | pending |
| SC-PACK-90 | pending |
| SC-PACK-91 | pending |
| SC-PACK-92 | pending |
| SC-PACK-93 | pending |
| SC-PACK-94 | pending |
| SC-PACK-95 | pending |
| SC-PACK-96 | pending |
| SC-PACK-81 | pending (row-fill behavior replaced by SC-PACK-85) |

## ADDED Requirements

### Requirement: Approved answers status uses published label

When answers moderation status is **approved** and the pack has a live (catalog) answers revision, the cards editor status under the page title MUST show «Опубликовано» (same vocabulary family as list marks). When tasks moderation status is **approved** but the pack has **no** live answers revision yet, the tasks editor status MUST show «Одобрено». Three-phase labels (awaiting submit / pending / needs revision) MUST remain unchanged.

#### Scenario [SC-PACK-86]: Cards page shows published after answers approve

- **GIVEN** pack P has an approved live answers revision and answers are not dirty
- **WHEN** an editor opens the cards page for P
- **THEN** the status under the page title shows «Опубликовано»

#### Scenario [SC-PACK-87]: Tasks page shows approved before catalog live

- **GIVEN** tasks were approved (live tasks exist) but pack P has no live answers / is not in the catalog
- **AND** tasks are not dirty
- **WHEN** an editor opens the tasks page for P
- **THEN** the status under the page title shows «Одобрено»

### Requirement: Live and staff task lists show slot contents

The public live pack view MUST show each task’s answer slots (filled card content and/or empty), not only a filled-slot count. Staff tasks preview MUST show the same slot presentation (slot chips / slot contents), not a plain joined text line alone.

#### Scenario [SC-PACK-88]: Live pack task rows show slots

- **GIVEN** published pack P has a task with one or more slots (filled or empty)
- **WHEN** any user opens the public live pack page for P
- **THEN** each task row shows those slots’ contents (or empty)
- **AND** the row MUST NOT rely solely on a «слотов: N» count

#### Scenario [SC-PACK-89]: Staff tasks preview shows slots

- **GIVEN** staff opens the nested tasks preview for a moderation request
- **WHEN** the tasks list renders
- **THEN** each task row shows answer slots as slot contents (filled and/or empty)

### Requirement: Cascade gaps use yellow outline only

While cascade gaps remain after an answers content change or delete, the task-set row and the task row MUST use a yellow **outline/border** indication. The client MUST NOT fill the whole row with a solid warning background. Empty slot chips MAY keep a yellow border as well.

#### Scenario [SC-PACK-85]: Yellow border without row fill after cascade

- **GIVEN** cascade from an answers content change or delete emptied slots on task T in set S
- **WHEN** the editor views the task-set list and the tasks list for S
- **THEN** set S and task T are indicated with a yellow border/outline while those cascade gaps remain
- **AND** the row background MUST NOT be a solid warning fill

### Requirement: Stale personal draft offers pull preserving local edits

When an editor’s personal draft is behind the current live (or last published) snapshot — for example after another author’s changes were approved — the client MUST warn that the pack changed and MUST offer a control to pull the new base. Pulling MUST rebase the draft onto the current live snapshot while preserving the editor’s local additions and content changes as **new** entities (new ids); entities that match the incoming live id keep the live version, and the editor’s divergent edit of that id becomes a separate new entity.

#### Scenario [SC-PACK-90]: Banner and pull when draft is stale

- **GIVEN** user B has a personal draft for pack P that does not match the current live base after an approval
- **WHEN** B opens the cards or tasks editor for P
- **THEN** the client shows a warning that the pack changed
- **AND** offers a control to pull the new state

#### Scenario [SC-PACK-91]: Pull keeps B’s edits as new ids

- **GIVEN** B’s draft changed answer card X (same id as live) and also added a new card Y
- **WHEN** B confirms pull
- **THEN** the draft base matches current live (including live’s X)
- **AND** B’s divergent content for X appears as a new card with a new id
- **AND** card Y remains in the draft
- **AND** answers (and tasks if affected) are dirty relative to the last submit snapshots as appropriate

### Requirement: Staff may unpublish and republish a pack from collection

Moderator and admin MUST be able to **unpublish** a catalog-live pack from the **collection** list (affordance next to Edit), with no minimum task-set count. Unpublish MUST: clear live answers and live tasks pointers so the pack leaves the catalog; cancel all open moderation requests for the pack; delete all personal drafts for the pack; retain a last-live snapshot sufficient to restore. While unpublished this way, non-staff users (including the pack creator) MUST only be able to remove the pack from their collection — Edit and submit MUST be unavailable. Staff who have the pack in collection MAY still open Edit. Staff MUST have a separate **republish** control that restores the retained last-live snapshot **directly** into the catalog (no moderation queue). After republish, eligible collection members MAY edit and submit again as for a normal published pack. Block UI remains out of scope (SC-PACK-59 unchanged).

#### Scenario [SC-PACK-92]: Staff unpublish from collection

- **GIVEN** staff S has published pack P in their collection
- **WHEN** S chooses unpublish on P’s collection row
- **THEN** P is absent from the public catalog
- **AND** all open moderation requests for P are closed/cancelled
- **AND** all personal drafts for P are removed
- **AND** a last-live snapshot is retained for republish

#### Scenario [SC-PACK-93]: Non-staff cannot edit after staff unpublish

- **GIVEN** pack P was unpublished by staff and remains in user U’s collection (U is not staff)
- **WHEN** U views the collection row for P
- **THEN** Edit is unavailable
- **AND** U MAY remove P from the collection
- **AND** draft/submit APIs reject U for P

#### Scenario [SC-PACK-94]: Staff republish restores last live into catalog

- **GIVEN** pack P was unpublished by staff and last-live snapshot L exists
- **WHEN** staff chooses republish for P from the collection
- **THEN** P appears in the catalog with content from L
- **AND** no new moderation request is required for that restore
- **AND** eligible non-staff editors MAY Edit P again afterward

### Requirement: Staff may unpublish one live task set when more than one remain

Moderator and admin MUST be able to remove **one** task set from the published live content via a control on the **right** of that task-set row (cards editor task-set list). This MUST be allowed only when the live pack has **two or more** task sets; after removal the live pack MUST still have at least one task set. Unpublishing a task set MUST cancel all open moderation requests for the pack. Personal drafts MUST be left intact (authors may still hold the removed set in draft). Pack-level unpublish has no task-set count precondition.

#### Scenario [SC-PACK-95]: Staff removes a non-last live task set

- **GIVEN** published pack P has live task sets S1 and S2
- **AND** staff S is editing P
- **WHEN** S unpublishes task set S2 from the task-set list
- **THEN** live content for P no longer includes S2
- **AND** S1 remains in live
- **AND** all open moderation requests for P are cancelled
- **AND** personal drafts for P are not deleted solely because of this action

#### Scenario [SC-PACK-96]: Cannot unpublish the last live task set

- **GIVEN** published pack P has exactly one live task set S1
- **WHEN** staff attempts to unpublish S1 from live
- **THEN** the system rejects the attempt
- **AND** S1 remains in live
