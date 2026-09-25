## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-09 | removed (superseded by map capacity) |
| SC-LOBBY-10 | removed (superseded by map capacity) |
| SC-LOBBY-21 | pending (client + server mocha) |
| SC-LOBBY-22 | pending (client + server mocha) |
| SC-LOBBY-23 | pending (client + server mocha) |
| SC-LOBBY-24 | pending (client UX) |
| SC-LOBBY-25 | pending (client UX) |
| SC-LOBBY-26 | pending (client UX + lobby metadata) |
| SC-LOBBY-27 | pending (server mocha create reject) |

Related: map snapshot / task deck — `game/board`; seating capacity — `game/pieces` / `game/start`. Grille/catapult density unchanged (SC-LOBBY-13…18).

## REMOVED Requirements

### Requirement: Create game chooses max seats

**Reason:** Max seated players now comes from the selected content map (`players`), including maps for one player.
**Migration:** Create MUST require an in-catalog map; `maxSeats` MUST equal that map’s `players`. Scenarios SC-LOBBY-09 and SC-LOBBY-10 no longer apply.

## ADDED Requirements

### Requirement: Create game chooses an in-catalog map

When an authenticated user opens create-game from the lobby, the system SHALL require selecting exactly one **in-catalog** content map. The create UI MUST show the map’s capacity as **players × tourists per player** before confirm and after selection. Confirming create MUST create a `tourist` room whose `maxSeats` equals the map’s `players` and whose play layout and tourist count per seat come from that map’s live snapshot. Soft-unpublished or never-published maps MUST NOT be selectable. Cancelling MUST NOT create a room.

#### Scenario [SC-LOBBY-21]: Create requires map and shows capacity

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the user MUST select an in-catalog map before confirm is allowed
- **AND** after selecting a map the UI shows that map’s players × tourists per player

#### Scenario [SC-LOBBY-22]: Confirm create uses map players as maxSeats

- **GIVEN** the create modal has in-catalog map M with players=3 and touristsPerPlayer=2 selected
- **WHEN** the user confirms create with valid task-set selection
- **THEN** a new `tourist` room is created with maxSeats 3
- **AND** each seated player will receive 2 pieces at playing materialize per `game/pieces`

### Requirement: Create game chooses task sets from one pack

When an authenticated user opens create-game, the system SHALL require selecting one **in-catalog** content pack and one or more of that pack’s **published** task sets (soft-unpublished sets MUST NOT appear). The picker MUST show each set with the pack title (theme) and the task-set author display label. The user MAY select a single set or multiple sets via checkboxes, including selecting all listed published sets of that pack. Task sets from a different pack MUST NOT be combinable in one create. Confirming create MUST pass the selected pack and task-set identities into the room so the server snapshots questions, difficulties, slots, and answer cards for play. Create MUST be rejected when no published task set is selected.

#### Scenario [SC-LOBBY-23]: Multi-select sets only within one pack

- **GIVEN** the create modal and in-catalog pack P with two published task sets S1 and S2
- **WHEN** the user selects P and checks both S1 and S2
- **THEN** confirm create is allowed
- **AND** the new room’s task pool includes tasks from both S1 and S2

#### Scenario [SC-LOBBY-24]: Set picker shows pack theme and author

- **GIVEN** published task set S on pack titled «Математика» authored by a user whose display name is «Иван»
- **WHEN** the create task-set picker lists S
- **THEN** the row shows the pack title «Математика» and an author label for Иван (product sense: набор заданий от Ивана)

#### Scenario [SC-LOBBY-27]: Create rejected without task set

- **GIVEN** the create modal has a valid map selected and no published task set selected
- **WHEN** the user attempts to confirm create
- **THEN** the system rejects create
- **AND** no `tourist` room is created

### Requirement: Lobby lists map capacity, preview, and task sets

Each listed `tourist` room in the live lobby listing SHALL show a mini preview of the room’s map grid, the capacity as players × tourists (or equivalent product copy consistent with create), and which pack/task-set selection is being played (pack title and selected set author labels). Occupied seats over maxSeats (`occupied/maxSeats`) MUST remain as today.

#### Scenario [SC-LOBBY-25]: Listing shows map preview and capacity

- **GIVEN** a listed tourist room created from map M with players=2 and touristsPerPlayer=3
- **WHEN** a lobby subscriber views that room row
- **THEN** the row shows a mini preview of M’s grid
- **AND** shows capacity reflecting 2 players and 3 tourists per player

#### Scenario [SC-LOBBY-26]: Listing shows played task sets

- **GIVEN** a listed tourist room created with pack titled «Математика» and one selected task set by author «Мария»
- **WHEN** a lobby subscriber views that room row
- **THEN** the row indicates «Математика» and the selected set’s author (Мария)
