## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-07 | covered (client — initial subscribe fail still surfaces) |
| SC-LOBBY-08 | covered (client — lobby drop / reservation noise quiet) |

## MODIFIED Requirements

### Requirement: Lobby listing errors

If establishing the lobby listing subscription fails on lobby entry (or the listing remains unavailable after a quiet resubscribe attempt), the system SHALL surface an error on the lobby screen so the user can understand that the room list is unavailable, without blocking unrelated navigation such as logout. Transient disconnects of an already-established lobby listing subscription, automatic reconnect failures, and matchmaking messages such as seat reservation expired that arise from lobby reconnect attempts MUST NOT be treated as that user-facing listing error; the client MAY quietly clear the subscription and resubscribe while the lobby screen is still active.

#### Scenario [SC-LOBBY-07]: Failed lobby subscribe

- **GIVEN** the user is on the lobby screen
- **WHEN** the lobby listing subscription fails to connect on entry (or remains unavailable after a quiet resubscribe attempt)
- **THEN** the lobby UI shows an error indication for the listing
- **AND** the user can still leave the lobby (for example log out)

## ADDED Requirements

### Requirement: Lobby subscription has no reconnect hold

The lobby listing room MUST NOT hold a disconnected subscriber via reconnection grace. The client MUST NOT persist a lobby reconnection token in localStorage or sessionStorage for page reload. A dropped lobby listing connection while the user remains on the lobby screen MAY be followed by a new listing subscription without showing seat-reservation or reconnect failure text to the user.

#### Scenario [SC-LOBBY-08]: Lobby drop does not surface reservation expired

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **WHEN** that subscription drops unexpectedly or a lobby reconnect/reservation attempt fails
- **THEN** the UI does not show a seat reservation expired (or equivalent reconnect) error for that listing drop
- **AND** the client MAY establish a fresh lobby listing subscription while the lobby screen stays active
- **AND** room list updates resume after a successful resubscribe
