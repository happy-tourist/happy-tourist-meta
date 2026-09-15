## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-09 | pending (client UX) |
| SC-LOBBY-10 | pending (client UX) |
| SC-LOBBY-11 | pending (client UX) |
| SC-LOBBY-12 | pending (client + listing metadata) |

## ADDED Requirements

### Requirement: Create game chooses max seats

When an authenticated user chooses to create a tourist game from the lobby, the system SHALL present a modal choice of max seated players with options 2, 3, and 4, defaulting to 2. Confirming create MUST create a `tourist` room whose maxSeats equals the selected value. Cancelling the modal MUST NOT create a room.

#### Scenario [SC-LOBBY-09]: Default max seats is two

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the selectable max seated counts are 2, 3, and 4
- **AND** 2 is selected by default

#### Scenario [SC-LOBBY-10]: Confirm create uses selected max seats

- **GIVEN** the create modal is open with max seats 3 selected
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with maxSeats 3
- **AND** the user enters that game session

### Requirement: Lobby lists occupied seats over max seats

Each listed `tourist` room in the live lobby listing SHALL show occupied seated count over maxSeats in the form `occupied/maxSeats` (for example `2/4`). Spectators (non-seated clients) MUST NOT increase the occupied numerator. The denominator MUST be that room’s maxSeats.

#### Scenario [SC-LOBBY-11]: Listing shows seats over maxSeats

- **GIVEN** a listed tourist room with maxSeats 4, two seated players, and one spectator
- **WHEN** a lobby subscriber views that room row
- **THEN** the capacity text shows `2/4`
- **AND** does not count the spectator in the numerator

### Requirement: Play shortcut removed from lobby

The lobby screen MUST NOT offer a primary «Играть» / joinOrCreate shortcut that enters an arbitrary tourist room without choosing a listed room. Users MUST create a game (with max seats) or join a specific room from the live listing.

#### Scenario [SC-LOBBY-12]: No play shortcut on lobby

- **GIVEN** an authenticated user on the lobby screen
- **WHEN** the lobby actions are shown
- **THEN** there is no «Играть» action that performs joinOrCreate into an arbitrary room
- **AND** create-game and join-by-listed-room actions remain available
