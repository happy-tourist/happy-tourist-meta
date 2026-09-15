## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-01 | covered-by-reuse (server mocha) |
| SC-MOVE-02 | covered (server mocha — skip finished via SC-MOVE-21) |
| SC-MOVE-03 | covered-by-reuse (server mocha — solo; finished skip via SC-MOVE-21) |
| SC-MOVE-08 | covered (server mocha — center finishes piece / SC-FINISH-01) |
| SC-MOVE-16 | covered-by-reuse (server mocha) |
| SC-MOVE-17 | covered-by-reuse (server mocha — active offline) |
| SC-MOVE-19 | covered-by-reuse (server mocha) |
| SC-MOVE-21 | covered (server mocha) |
| SC-MOVE-22 | covered (server mocha) |
| SC-MOVE-23 | covered (server mocha) |
| SC-MOVE-24 | covered (client UX) |

Related: piece finish side-effects — `game/finish`.

## MODIFIED Requirements

### Requirement: Turn order among seated players

As soon as at least one seated player exists in the tourist room, the server SHALL maintain a synchronized current-turn seat among seated players that are eligible to move. A seat with a finish place (`game/finish`) MUST NOT be eligible to hold or receive the current turn while any non-finished seated player remains. Turn order MUST follow join order of seated players (earliest seated first), skipping finished seats. After a successful move by the current player, the turn MUST pass immediately to the next non-finished seated player in that order, wrapping to the earliest non-finished when the end is reached. When only one non-finished seated player remains, that player MUST receive the turn again after their own successful move. When a new player becomes seated while a free seat slot exists (including during `countdown` or `playing`), they MUST be appended to the end of the turn order. When the seated player whose turn it is permanently leaves (consented leave or reconnect grace timeout), the turn MUST advance immediately to the next remaining non-finished seated player. While a non-finished seated player is offline within reconnect grace, the current turn MUST remain on that seat until they reconnect and move or the seat is removed. While a finished seat is offline within reconnect grace, the current turn MUST NOT remain on that finished seat: the turn MUST advance to the next non-finished seated player if any exist. When every remaining seated player is finished, the server MUST NOT require a move-capable current turn for gameplay. Having a current-turn seat MUST NOT by itself allow moves before start phase `playing` (see authoritative move requirement and `game/start`).

#### Scenario [SC-MOVE-01]: First seated player holds the turn

- **GIVEN** a tourist room with no seated players
- **WHEN** the first authenticated client joins and receives a seat
- **THEN** that seat is the current turn
- **AND** every client in the room observes the synchronized current-turn indicator for that seat

#### Scenario [SC-MOVE-02]: Join order defines the rotation

- **GIVEN** a tourist room with two or more non-finished seated players in known join order and phase `playing`
- **WHEN** the earliest non-finished seated player completes a successful move
- **THEN** the current turn becomes the next non-finished seated player in join order
- **AND** after the last non-finished seated player in that order moves successfully, the turn returns to the earliest non-finished seated player

#### Scenario [SC-MOVE-03]: Solo seated player keeps the turn after moving

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player whose turn it is (other seats may be finished)
- **WHEN** that player completes a successful move that does not finish their last piece
- **THEN** the current turn remains that same non-finished seated player

#### Scenario [SC-MOVE-16]: Permanent leave advances the turn

- **GIVEN** a tourist room with at least two seated players and the current turn belongs to one of them
- **WHEN** that current-turn player permanently leaves (consented leave or grace timeout)
- **THEN** the current turn advances to the next remaining non-finished seated player in join order
- **AND** the departed seat is no longer in the turn rotation

#### Scenario [SC-MOVE-17]: Offline grace keeps the turn waiting

- **GIVEN** a non-finished seated player whose turn it is and who disconnects unexpectedly within reconnect grace
- **WHEN** other clients remain in the room during the grace
- **THEN** the synchronized current turn stays on that offline non-finished seat
- **AND** no other seated player becomes current turn solely because of that disconnect

#### Scenario [SC-MOVE-19]: Mid-game seat appends to turn order

- **GIVEN** a tourist room in phase `playing` with two seated players in known turn order
- **WHEN** a third authenticated client joins and receives a seat
- **THEN** that new seat is appended to the end of the turn order
- **AND** the current turn does not jump to the newcomer solely because of the join

#### Scenario [SC-MOVE-21]: Finished seats are skipped in turn rotation

- **GIVEN** a tourist room in phase `playing` with seated players A then B in join order, where A has a finish place and B does not, and it is B’s turn
- **WHEN** B completes a successful move
- **THEN** the current turn returns to B (A is skipped)
- **AND** A never becomes current turn while finished and B remains non-finished

#### Scenario [SC-MOVE-22]: Finished offline seat does not hold the turn

- **GIVEN** a finished seated player would be next in join order and that finished seat is offline within reconnect grace, and at least one non-finished seated player remains
- **WHEN** the previous non-finished player completes a successful move
- **THEN** the current turn advances to the next non-finished seated player
- **AND** the offline finished seat does not hold the current turn

### Requirement: Authoritative one-step tourist move

The server SHALL accept a move message only when the room start phase is `playing`, and only from a non-finished seated client whose turn it is that identifies one of that client’s own unfinished pieces and a target cell. A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, task/field, or center cell), and MUST NOT land on a cell occupied by any unfinished piece in the room (including the mover’s other unfinished pieces). Finished pieces MUST NOT occupy cells and MUST NOT be move targets as pieces. Holes and cells outside the playable layout MUST be rejected. Moves from a client that is not seated, is finished, not the current turn, not in phase `playing`, that target another player’s piece, or that target an already finished own piece MUST be rejected without changing piece positions or the current turn. On acceptance onto a non-center playable cell the server MUST update the piece’s synchronized row and column and advance the turn per the turn-order requirement. On acceptance onto a center cell the server MUST finish the piece per `game/finish` (piece leaves board occupancy) and advance the turn per the turn-order requirement. There is no pass action in this change.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position and turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free orthogonal neighbor playable non-center cell
- **WHEN** that player submits a move for that piece to that neighbor cell
- **THEN** the server updates that piece’s synchronized row and column to the target
- **AND** the turn advances according to turn order
- **AND** every client observes the new piece position

#### Scenario [SC-MOVE-05]: Diagonal step is allowed

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free diagonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that diagonal neighbor
- **THEN** the server accepts the move and applies the non-center or center outcome as defined above

#### Scenario [SC-MOVE-06]: Occupied cell is rejected

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and a neighbor playable cell is occupied by any unfinished tourist piece (own or another player’s)
- **WHEN** that player submits a move onto that occupied cell
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-07]: Non-playable cell is rejected

- **GIVEN** the room phase is `playing` and it is a seated non-finished player’s turn
- **WHEN** that player submits a move onto a hole or otherwise non-playable cell adjacent in grid coordinates
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-08]: Start and center cells are walkable

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and an unfinished piece has a free neighbor that is a start cell or a center cell of the tourist layout
- **WHEN** that player submits a move onto that cell
- **THEN** the server accepts the move
- **AND** if the target is a start cell, the piece’s synchronized position updates to that cell
- **AND** if the target is a center cell, the piece is finished per `game/finish` and leaves board occupancy
- **AND** the turn advances according to turn order skipping finished seats

#### Scenario [SC-MOVE-09]: Move out of turn is rejected

- **GIVEN** the room phase is `playing` and a seated non-finished player whose turn it is not
- **WHEN** that player submits a move for one of their pieces
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-10]: Spectator cannot move

- **GIVEN** a spectator client in a tourist room that has at least one seated player
- **WHEN** that spectator attempts to submit a move
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-18]: Move before playing is rejected

- **GIVEN** a tourist room in phase `waiting` or `countdown` with a seated player who holds the current turn
- **WHEN** that player submits a legal one-step move
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-23]: Finished seat cannot move

- **GIVEN** a seated player who has a finish place in phase `playing`
- **WHEN** that player submits a move message
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

## ADDED Requirements

### Requirement: No move chrome for finished pieces or finished seats

While a seated client’s seat has a finish place, or while selecting among pieces, that client MUST NOT treat finished pieces or finished strip slots as selectable for moves, and MUST NOT show white selection / red destination chrome for finished pieces. Unfinished pieces of a non-finished current-turn seat keep existing local selection and hint rules.

#### Scenario [SC-MOVE-24]: Finished strip slot has no move chrome

- **GIVEN** it is the user’s turn as a non-finished seated player and one of their pieces is already finished
- **WHEN** the user views the board and personal strip
- **THEN** the finished piece is not on the board as a movable piece
- **AND** activating the finished strip slot does not show white/red move chrome or submit a move
