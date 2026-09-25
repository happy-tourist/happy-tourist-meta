# content/maps — delta: list statuses/filters, author re-edit, moderation take, cancel→draft, crumbs

Базовый канон: `openspec/specs/content/maps/spec.md`. Паритет с packs + follow-up: cancel→draft, убрать staff-link из списка, крошки. Избранного у карт нет.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
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
| SC-MAP-52 | covered (vitest) |

Related: `content/packs`; `ui/branding`. Tourist-room wiring still out of scope.

## ADDED Requirements

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

Maps section surfaces MUST show breadcrumbs **below** the shared elevated header chrome (e.g. Lobby / Maps / map title), consistent with packs chrome — not inside the elevated header bar.

#### Scenario [SC-MAP-44]: Breadcrumbs on Maps list and editor

- **GIVEN** an authenticated user on the Maps list or map editor
- **WHEN** the page renders
- **THEN** breadcrumbs include a path back toward Lobby and Maps

#### Scenario [SC-MAP-52]: Map breadcrumbs sit below elevated header

- **GIVEN** an authenticated user on a Maps list or editor route with breadcrumbs
- **WHEN** the page renders
- **THEN** breadcrumbs are outside the elevated shared header bar (below header chrome)

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
