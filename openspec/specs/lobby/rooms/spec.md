# lobby/rooms Specification

## Purpose

Live-список доступных игровых комнат на экране лобби: авторизованный пользователь (включая гостя) видит актуальные комнаты `tourist` в реальном времени и может создать или присоединиться к партии.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-01 | partial (client subscribe on mount; full check → task 3.2 manual) |
| SC-LOBBY-02 | covered (server mocha, task 1.5) |
| SC-LOBBY-03 | covered (server mocha, task 1.5) |
| SC-LOBBY-04 | covered (registration + tests/loadtest `tourist`, tasks 1.2/1.4) |
| SC-LOBBY-05 | partial (client `_enterRoom` unsubscribe; full check → task 3.2) |
| SC-LOBBY-06 | pending (manual, task 3.2) |
| SC-LOBBY-07 | covered (client — initial subscribe fail still surfaces) |
| SC-LOBBY-08 | covered (client — lobby drop / reservation noise quiet) |
| SC-LOBBY-09 | covered (client UX) |
| SC-LOBBY-10 | covered (client UX) |
| SC-LOBBY-11 | covered (client UX) |
| SC-LOBBY-12 | covered (client UX) |

## Requirements

### Requirement: Live lobby listing subscription

The system SHALL provide an authorized user on the lobby screen with a live listing of available `tourist` game rooms over a realtime Colyseus connection to the lobby listing room, without periodic HTTP polling of the room list while the lobby screen is active.

#### Scenario [SC-LOBBY-01]: Subscribe on lobby entry

- **GIVEN** the user is authenticated (registered or anonymous guest) and navigates to the lobby screen
- **WHEN** the lobby screen becomes active
- **THEN** the client establishes a realtime subscription to the lobby listing filtered to room name `tourist`
- **AND** the UI receives an initial snapshot of available `tourist` rooms
- **AND** the client does not rely on a repeating HTTP poll interval to refresh that list while the subscription is active

#### Scenario [SC-LOBBY-02]: Room appears in listing

- **GIVEN** at least one client is subscribed to the lobby listing for `tourist`
- **WHEN** a new `tourist` room becomes available for listing
- **THEN** every subscribed lobby client SHALL receive an update that adds that room to the list without requiring a manual refresh

#### Scenario [SC-LOBBY-03]: Room leaves listing

- **GIVEN** a `tourist` room is visible in the live lobby listing
- **WHEN** that room is disposed or otherwise removed from the public listing
- **THEN** every subscribed lobby client SHALL receive an update that removes that room from the list

### Requirement: Canonical tourist room name

The server SHALL register the playable game room under the name `tourist`, matching the client contract for create, join, joinOrCreate, and lobby filter.

#### Scenario [SC-LOBBY-04]: Create and join by tourist name

- **GIVEN** the server is running with the game room registered as `tourist`
- **WHEN** an authenticated client creates or joins a game via the lobby actions (create / joinOrCreate / join by id of a listed `tourist` room)
- **THEN** the operation targets the `tourist` room type
- **AND** newly created rooms are eligible for the live lobby listing of `tourist`

### Requirement: Leave lobby listing when entering a game

While the user is on the game screen in an active `tourist` session, the client SHALL NOT keep an active lobby listing subscription. The lobby listing subscription MUST be closed before or when entering a game room, and MAY be re-established when the user returns to the lobby screen.

#### Scenario [SC-LOBBY-05]: Unsubscribe before or on game enter

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **WHEN** the user successfully enters a `tourist` game (create / join by listed room)
- **THEN** the lobby listing subscription is closed
- **AND** the client navigates to the game screen with an active game room connection

#### Scenario [SC-LOBBY-06]: Resubscribe on return to lobby

- **GIVEN** the user left the lobby listing subscription when entering a game
- **WHEN** the user returns to the lobby screen without an active need for the game session listing
- **THEN** the client establishes a new lobby listing subscription and shows the current `tourist` room list

### Requirement: Lobby listing errors

If establishing the lobby listing subscription fails on lobby entry (or the listing remains unavailable after a quiet resubscribe attempt), the system SHALL surface an error on the lobby screen so the user can understand that the room list is unavailable, without blocking unrelated navigation such as logout. Transient disconnects of an already-established lobby listing subscription, automatic reconnect failures, and matchmaking messages such as seat reservation expired that arise from lobby reconnect attempts MUST NOT be treated as that user-facing listing error; the client MAY quietly clear the subscription and resubscribe while the lobby screen is still active.

#### Scenario [SC-LOBBY-07]: Failed lobby subscribe

- **GIVEN** the user is on the lobby screen
- **WHEN** the lobby listing subscription fails to connect on entry (or remains unavailable after a quiet resubscribe attempt)
- **THEN** the lobby UI shows an error indication for the listing
- **AND** the user can still leave the lobby (for example log out)

### Requirement: Lobby subscription has no reconnect hold

The lobby listing room MUST NOT hold a disconnected subscriber via reconnection grace. The client MUST NOT persist a lobby reconnection token in localStorage or sessionStorage for page reload. A dropped lobby listing connection while the user remains on the lobby screen MAY be followed by a new listing subscription without showing seat-reservation or reconnect failure text to the user.

#### Scenario [SC-LOBBY-08]: Lobby drop does not surface reservation expired

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **WHEN** that subscription drops unexpectedly or a lobby reconnect/reservation attempt fails
- **THEN** the UI does not show a seat reservation expired (or equivalent reconnect) error for that listing drop
- **AND** the client MAY establish a fresh lobby listing subscription while the lobby screen stays active
- **AND** room list updates resume after a successful resubscribe

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
