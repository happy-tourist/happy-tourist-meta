## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-231 | covered (client vitest staff boot) |
| SC-PACK-232 | covered (client vitest never-published Submit) |
| SC-PACK-233 | covered (server mocha staff-save clear working) |
| SC-PACK-234 | covered (client vitest dirty Submit) |
| SC-PACK-235 | covered (client vitest live view no CSV) |
| SC-PACK-236 | covered (client vitest staff Edit CSV) |
| SC-PACK-237 | covered (client vitest compose slot chrome) |
| SC-PACK-238 | covered (client vitest PackTaskTile slot chrome) |

Related: staff false-draft SC-PACK-230; CSV SC-PACK-213…220; main `content/packs`.

## ADDED Requirements

### Requirement: Staff edit published packs without moderation queue

When a moderator or admin opens Edit on a pack that already has live content (`hasLive`), the client MUST enter the staff direct-edit path (staff-save) even if that staff user is the pack `createdBy` or a task-set author. The staff Edit surface MUST NOT show «На модерацию» / Submit. Successful staff-save MUST update live content without creating a moderation request. For a **never-published** pack (`!hasLive`), staff who is the creator MUST retain the creator Submit-to-moderation path (draft badge and queue submit remain as today).

#### Scenario [SC-PACK-231]: Staff creator on published pack has no Submit

- **GIVEN** a published in-catalog pack created by user S who is staff (moderator|admin)
- **WHEN** S opens Edit on that pack
- **THEN** the editor is in staff direct-edit mode
- **AND** the «На модерацию» control is not shown
- **AND** saves use staff-save into live without opening a moderation request

#### Scenario [SC-PACK-232]: Staff creator on never-published pack keeps Submit

- **GIVEN** a never-published pack created by staff user S
- **WHEN** S opens the pack editor
- **THEN** the «На модерацию» control remains available subject to existing minima and lock rules
- **AND** successful submit creates or updates an author moderation request as for non-staff creators

### Requirement: Staff-save clears retained working copy

When staff saves a pack to live via staff-save and there is **no** open author moderation request (`pending` or `needs_revision`) on that pack, the system MUST clear the pack’s retained working revision pointer so list/editor status for authors and staff MUST NOT show author-facing **draft** solely from a leftover working≠live copy. Open author pending/needs_revision MUST still block staff-save as today. After such a clear, staff and the pack creator MUST see published/live status for that pack (unless another open request applies). Extends SC-PACK-230 beyond twin-only clear.

#### Scenario [SC-PACK-233]: Staff-save removes hanging draft after diverging working

- **GIVEN** a published pack with no open author request
- **AND** a retained working revision that differs from live (e.g. after cancel-to-draft)
- **WHEN** staff holds the edit lock and saves via staff-save
- **THEN** live reflects the staff save
- **AND** the retained working pointer is cleared (or otherwise no longer yields draft)
- **AND** subsequent list/status for staff and creator does not show draft solely from that prior working copy

### Requirement: Author Submit disabled until content is dirty

On pack creator / task-set-author editing surfaces that expose «На модерацию» (cards editor, task-set editor, add-task-set), Submit MUST be disabled while the in-memory editor content is unchanged relative to the content loaded for that edit session, even when submit minima are already met. After the user changes any editable field that participates in the working copy, Submit MUST become enabled when minima and existing lock/pending-other rules allow. Staff direct-edit surfaces without Submit are unaffected.

#### Scenario [SC-PACK-234]: Submit disabled on open without edits

- **GIVEN** a verified author opens pack Edit (or add-task-set / task-set editor) with content that already meets submit minima
- **AND** the author has not changed the loaded content
- **WHEN** the client renders «На модерацию»
- **THEN** the control is disabled
- **AND GIVEN** the author changes at least one editable field in the working copy
- **WHEN** minima and lock rules still allow submit
- **THEN** «На модерацию» is enabled

### Requirement: Hide task CSV on live task-set view

On the live pack **view** drill-in into a task set (read-only live view for non-staff), the client MUST NOT show the framed task CSV import/export controls. On task **editing** surfaces (add-task-set, creator/staff task-set editor, including staff Edit of a live set) the existing CSV affordances MUST remain subject to SC-PACK-213…220.

#### Scenario [SC-PACK-235]: Live view has no CSV frame

- **GIVEN** a non-staff user opens a published pack’s live task-set drill-in (view-only)
- **WHEN** the task-set page is shown
- **THEN** the task CSV controls frame is not present

#### Scenario [SC-PACK-236]: Staff Edit keeps CSV

- **GIVEN** staff opens staff Edit for a published pack’s task set
- **WHEN** the task-set editor is shown
- **THEN** the task CSV import/export controls remain available subject to existing read-only and answer-context rules

### Requirement: Task answer slots match peek slot chrome

On task composing surfaces (slot row while filling a question) and on task tiles that display slot labels (`PackTaskTile` and equivalent list rows), answer slots MUST use the same approximate size and visual chrome as peek answer slots on the game board (min dimensions, padding, dashed empty / solid filled border treatment). Dense Quasar chips that read as a different scale from peek MUST NOT be the primary slot presentation for these surfaces.

#### Scenario [SC-PACK-237]: Compose slots match peek scale

- **GIVEN** a user is composing a task and filling answer slots
- **WHEN** the slot row is rendered
- **THEN** each slot uses peek-comparable min size and empty/filled border chrome

#### Scenario [SC-PACK-238]: Task tile slots match peek scale

- **GIVEN** a task list or card tile showing slot labels for a task
- **WHEN** the slots are rendered
- **THEN** those slots use the same peek-comparable chrome as the compose slot row
