## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-07 | covered (server mocha — consented leave before start) |
| SC-PIECE-08 | covered (server mocha — consented leave after start) |
| SC-PIECE-11 | covered (server mocha — unexpected hold before start) |
| SC-PIECE-12 | covered (server mocha — unexpected hold after start) |
| SC-PIECE-13 | covered (server mocha — reconnect within grace) |
| SC-PIECE-14 | covered (server mocha — grace timeout removes seat) |
| SC-PIECE-15 | covered (server mocha — zero seated disposes room) |
| SC-PIECE-16 | covered (server mocha — sync offline + deadline) |

## MODIFIED Requirements

### Requirement: Leave before and after start

A **consented** leave by a seated player (intentional exit from the match UI back to the lobby) SHALL permanently remove that player’s seat and all four pieces. If the room has not started, the tourist kind and occupied cells MUST return to the available pools. If the room has started, the server MUST NOT assign seats to new joiners after that removal. An **unexpected** disconnect is governed by the reconnect grace requirement and MUST NOT be treated as an immediate consented leave.

#### Scenario [SC-PIECE-07]: Leave before start frees kind and cells

- **GIVEN** a tourist room that has not started and has at least one seated player
- **WHEN** that seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner MAY receive the freed tourist kind and MAY occupy formerly taken start cells

#### Scenario [SC-PIECE-08]: Leave after start does not reopen seating

- **GIVEN** a started tourist room with four seated players
- **WHEN** one seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner still receives no seat

## ADDED Requirements

### Requirement: Unexpected disconnect grace and reconnect

When a seated player disconnects unexpectedly (for example page reload or network drop), the server MUST keep that seat and all four pieces in synced state for a grace period of exactly **30 seconds** and MUST allow the same client session to reconnect into that seat within the grace. During the grace the seat MUST be marked offline with a synchronized reconnect deadline visible to all clients in the room. The same grace policy applies whether or not the room has started.

#### Scenario [SC-PIECE-11]: Unexpected disconnect before start holds the seat

- **GIVEN** a tourist room that has not started and has at least one seated player who is marked connected
- **WHEN** that seated player disconnects unexpectedly
- **THEN** that player’s seat and four pieces remain in synced state
- **AND** the seat is marked offline with a reconnect deadline 30 seconds from the disconnect
- **AND** other clients in the room observe the offline mark and deadline

#### Scenario [SC-PIECE-12]: Unexpected disconnect after start holds the seat

- **GIVEN** a started tourist room with four seated players
- **WHEN** one seated player disconnects unexpectedly
- **THEN** that player’s seat and four pieces remain in synced state
- **AND** the seat is marked offline with a reconnect deadline 30 seconds from the disconnect
- **AND** the room remains started and does not assign a seat to a new joiner solely because of that disconnect

#### Scenario [SC-PIECE-13]: Reconnect within grace restores the same seat

- **GIVEN** a seated player whose seat is held offline within the 30-second grace
- **WHEN** that client successfully reconnects before the deadline
- **THEN** the client occupies the same seat with the same tourist kind and pieces
- **AND** the seat is marked connected again
- **AND** other clients observe the seat as online

#### Scenario [SC-PIECE-14]: Grace timeout removes the seat

- **GIVEN** a seated player whose seat is held offline within the grace
- **WHEN** the 30-second grace expires without a successful reconnect
- **THEN** that player’s seat and four pieces are removed from synced state
- **AND** if the room has not started, kind and cells return to the available pools
- **AND** if the room has started, seating remains closed to new joiners

### Requirement: Dispose when no seated players remain

After any permanent seat removal (consented leave or grace timeout), if the room has **zero** seated players left, the server SHALL close the room even if spectator clients are still connected. If at least one seated player remains, the room MUST continue and MUST NOT reopen seating solely because another seated player left after start.

#### Scenario [SC-PIECE-15]: Last seated removal closes the room with spectators present

- **GIVEN** a tourist room with exactly one seated player and at least one spectator
- **WHEN** that seated player permanently leaves (consented leave or grace timeout)
- **THEN** the room is closed
- **AND** remaining spectator clients are disconnected from that room

#### Scenario [SC-PIECE-16]: Seat connectivity is synchronized

- **GIVEN** a tourist room with at least one seated player
- **WHEN** that seat’s connected or offline-with-deadline status changes
- **THEN** every client in the room observes the updated connectivity fields for that seat in synced state
