## MODIFIED Requirements

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
- **WHEN** the user successfully enters a `tourist` game (play / create / join)
- **THEN** the lobby listing subscription is closed
- **AND** the client navigates to the game screen with an active game room connection

#### Scenario [SC-LOBBY-06]: Resubscribe on return to lobby

- **GIVEN** the user left the lobby listing subscription when entering a game
- **WHEN** the user returns to the lobby screen without an active need for the game session listing
- **THEN** the client establishes a new lobby listing subscription and shows the current `tourist` room list

## RENAMED Requirements

- FROM: `### Requirement: Canonical checkers room name`
- TO: `### Requirement: Canonical tourist room name`
