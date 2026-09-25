# content/maps — delta: list statuses/filters, author re-edit, moderation take

Базовый канон: `openspec/specs/content/maps/spec.md`. Паритет с packs: статусы/фильтры в общем списке, без author my-moderation nav у non-staff, author re-edit через очередь, edit lock для автора, staff «взять в модерацию». Избранного у карт нет.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-31 | pending |
| SC-MAP-32 | pending |
| SC-MAP-33 | pending |
| SC-MAP-34 | pending |
| SC-MAP-35 | pending |
| SC-MAP-36 | pending |
| SC-MAP-37 | pending |
| SC-MAP-38 | pending |
| SC-MAP-39 | pending |
| SC-MAP-40 | pending |

Related: `content/packs` (unified list / moderation take / author re-edit). Tourist-room wiring still out of scope.

## ADDED Requirements

### Requirement: Maps list shows moderation statuses

The Maps list MUST show status on each visible row distinguishing at least: in catalog; never-published draft without open request; open moderation **pending**; open moderation **needs_revision**; soft-unpublished (staff). A map with an open request MUST show pending or needs_revision rather than only a generic draft badge. Non-authors still MUST NOT see another user’s never-published maps.

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

## MODIFIED Requirements

### Requirement: Approve publishes into Maps list and freezes author edit

When staff approves a map moderation request (after take), the submitted grid and seat config MUST become the live snapshot, the map MUST become **in catalog**, and it MUST appear on the public Maps list. After the map has been approved at least once, non-staff users **other than the creator** MUST NOT edit the live map. The **creator** MUST be able to Edit under the exclusive lock and submit further changes through moderation (see ADDED). **Staff** (moderator|admin) MUST be able to Edit any map under exclusive lock and save directly to live/working **when** no open author map request blocks staff content edit and the lock is free. Never-approved maps remain editable by the creator (and staff).

#### Scenario [SC-MAP-16]: Approve lists map publicly

- **GIVEN** a pending map request and staff who has taken the request
- **WHEN** the actor approves
- **THEN** the map is in catalog
- **AND** any authenticated user sees it on the Maps list with mini preview, author, and players×tourists

#### Scenario [SC-MAP-17]: Author cannot edit after approve

- **GIVEN** map M approved into catalog and creator A (non-staff)
- **WHEN** A attempts a **direct live** edit/save without going through moderation submit
- **THEN** the system rejects applying those changes to live without moderation
- **AND** A MAY still acquire the edit lock, amend a working copy, and submit for moderation (see ADDED author re-edit)

#### Scenario [SC-MAP-18]: Staff edits under exclusive lock

- **GIVEN** approved map M with no open author request and free edit lock, and staff S
- **WHEN** S acquires the edit lock and saves grid or seat config changes
- **THEN** the live map reflects those changes without a new author moderation submit
- **AND** a second staff actor cannot acquire the lock while S holds it
