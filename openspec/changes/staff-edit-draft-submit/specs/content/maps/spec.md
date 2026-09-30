## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-62 | covered (client vitest staff boot) |
| SC-MAP-63 | covered (client vitest never-published Submit) |
| SC-MAP-64 | covered (server mocha staff-save clear working) |
| SC-MAP-65 | covered (client vitest dirty Submit) |

Related: SC-MAP-36 staff blocked by open request; main `content/maps`.

## ADDED Requirements

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
