## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-28 | removed (superseded — maps wired at create) |
| SC-MAP-60 | pending (server mocha — soft-unpublished reject) |
| SC-MAP-61 | pending (server mocha — create uses live map) |

Related: room create — `lobby/rooms`; runtime board — `game/board`.

## REMOVED Requirements

### Requirement: Maps are not used by tourist rooms in this capability

**Reason:** This change wires in-catalog maps into tourist room create and play layout.
**Migration:** Use the added requirement below. Scenario SC-MAP-28 from main specs is superseded by SC-MAP-61.

## ADDED Requirements

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
