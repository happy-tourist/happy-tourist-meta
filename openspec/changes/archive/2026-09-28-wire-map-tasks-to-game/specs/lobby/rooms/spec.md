## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-09 | removed (superseded by map ceiling + seats picker) |
| SC-LOBBY-10 | removed (superseded by map ceiling + seats picker) |
| SC-LOBBY-21 | covered (client — map required; capacity in options) |
| SC-LOBBY-22 | covered (revised — chosen maxSeats ≤ map.players) |
| SC-LOBBY-23 | covered (client + server) |
| SC-LOBBY-24 | covered (client UX) |
| SC-LOBBY-25 | covered (revised — listing uses room maxSeats) |
| SC-LOBBY-26 | covered (client UX + lobby metadata) |
| SC-LOBBY-27 | covered (server mocha create reject) |
| SC-LOBBY-28 | covered (client — seats picker 1…map.players) |
| SC-LOBBY-29 | covered (client — no duplicate capacity caption) |
| SC-LOBBY-30 | covered (client — auto-check single published set) |

Related: map snapshot / task deck — `game/board`; seating capacity — `game/pieces` / `game/start`. Grille/catapult density unchanged (SC-LOBBY-13…18).

## REMOVED Requirements

### Requirement: Create game chooses max seats

**Reason:** Legacy free-form 2/3/4 without a map is gone. Capacity is now **chosen seats capped by the selected map’s `players`**.
**Migration:** Create MUST require an in-catalog map; `maxSeats` MUST be an integer in `1…map.players`. Scenarios SC-LOBBY-09 and SC-LOBBY-10 no longer apply as unbounded seat radios without a map.

## ADDED Requirements

### Requirement: Create game chooses an in-catalog map

When an authenticated user opens create-game from the lobby, the system SHALL require selecting exactly one **in-catalog** content map. Soft-unpublished or never-published maps MUST NOT be selectable. Cancelling MUST NOT create a room. The map’s `players` value is the **ceiling** for seat selection (see seats requirement). Tourists per player and play layout MUST come from that map’s live snapshot at create. The create UI MUST expose each map’s players × tourists in the map **option** list (or equivalent picker chrome) and MUST NOT repeat the same capacity string as a separate caption under the closed map select after selection.

#### Scenario [SC-LOBBY-21]: Create requires map and shows capacity in options

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the user MUST select an in-catalog map before confirm is allowed
- **AND** each map option shows that map’s players × tourists per player

#### Scenario [SC-LOBBY-29]: No duplicate capacity caption under map select

- **GIVEN** the create modal and an in-catalog map is selected
- **WHEN** the map select is closed
- **THEN** the UI MUST NOT show a second caption line under the select that only repeats the same players × tourists text already conveyed by the picker options / selection

### Requirement: Create game chooses seat count up to map players

When an authenticated user has selected an in-catalog map, the create UI SHALL require choosing **maxSeats** as an integer from **1** through that map’s **players** inclusive. The default after map selection MUST be **`min(2, map.players)`**. Confirming create MUST create a `tourist` room whose synced `maxSeats` equals the chosen value (not necessarily the map’s full `players`). The server MUST reject create when `maxSeats` is missing/invalid or greater than the map’s `players` (or less than 1). When `maxSeats` is omitted by a legacy client, the server MAY default to `min(2, map.players)`.

#### Scenario [SC-LOBBY-28]: Seats picker respects map ceiling

- **GIVEN** the create modal has in-catalog map M with players=4 selected
- **WHEN** the seats control is shown
- **THEN** the user may choose 1, 2, 3, or 4
- **AND** the default selection is 2

#### Scenario [SC-LOBBY-22]: Confirm create uses chosen maxSeats

- **GIVEN** the create modal has in-catalog map M with players=3 and touristsPerPlayer=2 selected and the user chose maxSeats=2 with valid task-set selection
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with maxSeats 2
- **AND** each seated player will receive 2 pieces at playing materialize per `game/pieces`

### Requirement: Create game chooses task sets from one pack

When an authenticated user opens create-game, the system SHALL require selecting one **in-catalog** content pack and one or more of that pack’s **published** task sets (soft-unpublished sets MUST NOT appear). The picker MUST show each set with the pack title (theme) and the task-set author display label. The user MAY select a single set or multiple sets via checkboxes, including selecting all listed published sets of that pack. Task sets from a different pack MUST NOT be combinable in one create. When the selected pack has **exactly one** published task set, the create UI MUST pre-check that set. Confirming create MUST pass the selected pack and task-set identities into the room so the server snapshots questions, difficulties, slots, and answer cards for play. Create MUST be rejected when no published task set is selected.

#### Scenario [SC-LOBBY-23]: Multi-select sets only within one pack

- **GIVEN** the create modal and in-catalog pack P with two published task sets S1 and S2
- **WHEN** the user selects P and checks both S1 and S2
- **THEN** confirm create is allowed
- **AND** the new room’s task pool includes tasks from both S1 and S2

#### Scenario [SC-LOBBY-24]: Set picker shows pack theme and author

- **GIVEN** published task set S on pack titled «Математика» authored by a user whose display name is «Иван»
- **WHEN** the create task-set picker lists S
- **THEN** the row shows the pack title «Математика» and an author label for Иван (product sense: набор заданий от Ивана)

#### Scenario [SC-LOBBY-30]: Single published set is pre-checked

- **GIVEN** the create modal and in-catalog pack P with exactly one published task set S
- **WHEN** the user selects P
- **THEN** S is already checked
- **AND** confirm create is allowed without a further set click (given a valid map and seats)

#### Scenario [SC-LOBBY-27]: Create rejected without task set

- **GIVEN** the create modal has a valid map selected and no published task set selected
- **WHEN** the user attempts to confirm create
- **THEN** the system rejects create
- **AND** no `tourist` room is created

### Requirement: Lobby lists map capacity, preview, and task sets

Each listed `tourist` room in the live lobby listing SHALL show a mini preview of the room’s map grid, the room capacity based on the room’s **`maxSeats`** (chosen at create) and tourists per player from the map snapshot (or equivalent product copy), and which pack/task-set selection is being played (pack title and selected set author labels). Occupied seats over maxSeats (`occupied/maxSeats`) MUST remain as today and MUST use the room’s chosen `maxSeats` (not the map’s full `players` when those differ).

#### Scenario [SC-LOBBY-25]: Listing shows map preview and room capacity

- **GIVEN** a listed tourist room created from map M with players=4 and touristsPerPlayer=3 where the creator chose maxSeats=2
- **WHEN** a lobby subscriber views that room row
- **THEN** the row shows a mini preview of M’s grid
- **AND** shows capacity reflecting maxSeats 2 (not 4) and 3 tourists per player
- **AND** occupied/maxSeats uses maxSeats 2

#### Scenario [SC-LOBBY-26]: Listing shows played task sets

- **GIVEN** a listed tourist room created with pack titled «Математика» and one selected task set by author «Мария»
- **WHEN** a lobby subscriber views that room row
- **THEN** the row indicates «Математика» and the selected set’s author (Мария)
