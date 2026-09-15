## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-01 | covered (server mocha — reaffirm under maxSeats) |
| SC-PIECE-05 | covered (server mocha) |
| SC-PIECE-06 | covered (server mocha) |
| SC-PIECE-08 | covered (server mocha) |
| SC-PIECE-19 | covered (server mocha) |
| SC-PIECE-20 | covered-by-reuse (server mocha SC-PIECE-15) |

## MODIFIED Requirements

### Requirement: Four pieces per seated player

Until the tourist room has no free seat under its maxSeats, when an authenticated client joins and fewer seated players exist than maxSeats, the server SHALL assign that client a unique tourist kind from `1`…`4` not used by any current seat, and SHALL place exactly four pieces for that client — one on each board side `N`, `E`, `S`, and `W`. For each side the start cell MUST be chosen uniformly at random from the start cells of that side that are not already occupied by any piece in the room. All four pieces of the same player MUST use that player’s tourist kind. The assignment MUST be synchronized to all clients in the room. This seating rule applies in any start phase (`waiting`, `countdown`, or `playing`) whenever a free seat slot exists.

#### Scenario [SC-PIECE-01]: First join receives four pieces on all sides

- **GIVEN** a tourist room that has no seated players and has free seat capacity
- **WHEN** an authenticated client joins the room
- **THEN** the client is assigned exactly one tourist kind from `1`…`4`
- **AND** the client has exactly four pieces, one on each side `N`, `E`, `S`, and `W`
- **AND** each piece’s cell is one of the four start cells of its side
- **AND** every client in the room observes those pieces in synced state

#### Scenario [SC-PIECE-02]: Tourist kinds stay unique among players

- **GIVEN** a tourist room that already has one or more seated players and still has free seat capacity
- **WHEN** another authenticated client joins and receives a seat
- **THEN** the new tourist kind is not equal to any currently seated player’s kind
- **AND** all four of the new player’s pieces use that same new kind

#### Scenario [SC-PIECE-03]: Start cells lie on the assigned side and stay free

- **GIVEN** a tourist room with free seat capacity
- **WHEN** a client is assigned pieces on the four sides
- **THEN** each piece’s row and column match a start cell of that piece’s side on the agreed tourist layout
- **AND** no two pieces in the room share the same row and column

#### Scenario [SC-PIECE-04]: Second player uses remaining cells on each side

- **GIVEN** a tourist room that already has one seated player with four pieces and still has free seat capacity
- **WHEN** a second authenticated client joins and receives a seat
- **THEN** the second player also has one piece on each side `N`, `E`, `S`, and `W`
- **AND** none of the second player’s cells equal any cell already occupied by the first player

### Requirement: Spectators and hard stop after four seated players

The room MUST allow clients beyond maxSeats connections. The room’s maxSeats MUST be one of 2, 3, or 4 as set at create. While the number of seated players is strictly less than maxSeats, joining authenticated clients MUST receive a seat and pieces (including during `countdown` and `playing`). When seated count equals maxSeats, joining clients MUST NOT receive a seat or pieces (spectators / guests). Filling the last free seat while phase is `waiting` triggers start countdown per `game/start` (this requirement does not itself define countdown). The legacy meaning of `started` as a permanent seating lock after exactly four seats MUST NOT apply.

#### Scenario [SC-PIECE-05]: Fourth seated player starts the room

- **GIVEN** a tourist room in phase `waiting` whose maxSeats equals 4 and that has three seated players
- **WHEN** a fourth authenticated client joins and receives a seat with four pieces
- **THEN** seated count equals maxSeats
- **AND** subsequent joining clients receive no seat and no pieces
- **AND** start countdown begins per `game/start`

#### Scenario [SC-PIECE-06]: Fifth connection is a spectator

- **GIVEN** a tourist room that already has seated count equal to maxSeats
- **WHEN** another authenticated client joins
- **THEN** that client receives no tourist kind and no pieces
- **AND** existing seats and pieces remain unchanged

#### Scenario [SC-PIECE-19]: Mid-game join takes a free seat

- **GIVEN** a tourist room in phase `playing` with maxSeats 4 and two seated players
- **WHEN** another authenticated client joins
- **THEN** that client receives a seat and four pieces on free start cells
- **AND** every client observes the new seat and pieces

### Requirement: Leave before and after start

A **consented** leave by a seated player (intentional exit from the match UI back to the lobby) SHALL permanently remove that player’s seat and all four pieces. The tourist kind and occupied cells MUST return to the available pools whenever a seat is permanently removed, regardless of start phase. After that removal, if seated count is again below maxSeats, a subsequent joiner MUST be allowed to receive a seat and pieces. An **unexpected** disconnect is governed by the reconnect grace requirement and MUST NOT be treated as an immediate consented leave. Ready marks of remaining seats are not cleared by this leave (see `game/start`). If seated count reaches zero, the room is disposed even if spectators remain (unchanged).

#### Scenario [SC-PIECE-07]: Leave before start frees kind and cells

- **GIVEN** a tourist room in phase `waiting` with at least one seated player
- **WHEN** that seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner MAY receive the freed tourist kind and MAY occupy formerly taken start cells

#### Scenario [SC-PIECE-08]: Leave after full table reopens seating

- **GIVEN** a tourist room in phase `playing` with maxSeats 4 and four seated players
- **WHEN** one seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner receives a seat and pieces while seated count is below maxSeats

#### Scenario [SC-PIECE-20]: Empty seated disposes room

- **GIVEN** a tourist room with exactly one seated player and any number of spectators
- **WHEN** that seated player permanently leaves
- **THEN** the room is disposed
- **AND** spectators do not keep the room alive
