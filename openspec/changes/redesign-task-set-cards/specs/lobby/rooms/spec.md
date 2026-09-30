## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-23 | unchanged |
| SC-LOBBY-24 | covered (client vitest LobbyCreateWire — no set author; `#{n}` ordinal) |
| SC-LOBBY-25 | unchanged |
| SC-LOBBY-26 | covered (client vitest — listing without set author; `#{n}` ordinal) |
| SC-LOBBY-27 | unchanged |
| SC-LOBBY-30 | unchanged |

Related: task-set card chrome / author removal in content UI — `content/packs` delta. Map author labels unchanged.

## MODIFIED Requirements

### Requirement: Create game chooses task sets from one pack

When an authenticated user opens create-game, the system SHALL require selecting one **in-catalog** content pack and one or more of that pack’s **published** task sets (soft-unpublished sets MUST NOT appear). The picker MUST show each set with the pack title (theme) and a task-set ordinal label **without** the task-set author display name (preferred form «Набор заданий #{n}» or equivalent with a hash before the ordinal). The user MAY select a single set or multiple sets via checkboxes, including selecting all listed published sets of that pack. Task sets from a different pack MUST NOT be combinable in one create. When the selected pack has **exactly one** published task set, the create UI MUST pre-check that set. Confirming create MUST pass the selected pack and task-set identities into the room so the server snapshots questions, difficulties, slots, and answer cards for play. Create MUST be rejected when no published task set is selected.

#### Scenario [SC-LOBBY-23]: Multi-select sets only within one pack

- **GIVEN** the create modal and in-catalog pack P with two published task sets S1 and S2
- **WHEN** the user selects S1 and S2
- **THEN** both remain selected under P
- **AND** the user MUST NOT be able to combine a set from another pack in the same create

#### Scenario [SC-LOBBY-24]: Set picker shows pack theme and author

- **GIVEN** published task set S on pack titled «Математика» authored by a user whose display name is «Иван»
- **WHEN** the create modal lists sets for that pack
- **THEN** the row shows the pack title «Математика» and a task-set ordinal label with a hash before the ordinal and without «Иван» / author text

#### Scenario [SC-LOBBY-30]: Single published set is pre-checked

- **GIVEN** the create modal and in-catalog pack P with exactly one published task set S
- **WHEN** the task-set picker renders
- **THEN** S is pre-checked

#### Scenario [SC-LOBBY-27]: Create rejected without task set

- **GIVEN** the create modal has a valid map selected and no published task set selected
- **WHEN** the user confirms create
- **THEN** create is rejected or blocked until at least one published set is selected

### Requirement: Lobby lists map capacity, preview, and task sets

Each listed `tourist` room in the live lobby listing SHALL show a mini preview of the room’s map grid, the room capacity based on the room’s **`maxSeats`** (chosen at create) and tourists per player from the map snapshot (or equivalent product copy), and which pack/task-set selection is being played (pack title and selected set ordinal labels **without** task-set author names; ordinals prefer «Набор заданий #{n}» or equivalent). Occupied seats over maxSeats (`occupied/maxSeats`) MUST remain as today and MUST use the room’s chosen `maxSeats` (not the map’s full `players` when those differ).

#### Scenario [SC-LOBBY-25]: Listing shows map preview and room capacity

- **GIVEN** a listed tourist room with a map snapshot and chosen maxSeats
- **WHEN** the lobby listing renders that room
- **THEN** the row shows a mini map preview and capacity based on that room’s maxSeats and tourists per player

#### Scenario [SC-LOBBY-26]: Listing shows played task sets

- **GIVEN** a listed tourist room created with pack titled «Математика» and one selected task set by author «Мария»
- **WHEN** the lobby listing renders that room
- **THEN** the row indicates «Математика» and the selected set’s ordinal label (with hash before the ordinal when using the preferred form)
- **AND** MUST NOT show «Мария» / the task-set author as the set identity
