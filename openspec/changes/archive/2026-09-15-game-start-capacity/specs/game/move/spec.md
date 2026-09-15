## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-01 | covered (server mocha) |
| SC-MOVE-04 | covered (server mocha — additionally requires playing) |
| SC-MOVE-18 | covered (server mocha) |
| SC-MOVE-19 | covered (server mocha) |
| SC-MOVE-20 | covered (client UX) |

## MODIFIED Requirements

### Requirement: Turn order among seated players

As soon as at least one seated player exists in the tourist room, the server SHALL maintain a synchronized current-turn seat among seated players. Turn order MUST follow join order of seated players (earliest seated first). After a successful move by the current player, the turn MUST pass immediately to the next seated player in that order, wrapping to the earliest when the end is reached. When only one seated player remains, that player MUST receive the turn again after their own successful move. When a new player becomes seated while a free seat slot exists (including during `countdown` or `playing`), they MUST be appended to the end of the turn order. When the seated player whose turn it is permanently leaves (consented leave or reconnect grace timeout), the turn MUST advance immediately to the next remaining seated player. While a seated player is offline within reconnect grace, the current turn MUST remain on that seat until they reconnect and move or the seat is removed. Having a current-turn seat MUST NOT by itself allow moves before start phase `playing` (see authoritative move requirement and `game/start`).

#### Scenario [SC-MOVE-01]: First seated player holds the turn

- **GIVEN** a tourist room with no seated players
- **WHEN** the first authenticated client joins and receives a seat
- **THEN** that seat is the current turn
- **AND** every client in the room observes the synchronized current-turn indicator for that seat

#### Scenario [SC-MOVE-02]: Join order defines the rotation

- **GIVEN** a tourist room with two or more seated players in known join order and phase `playing`
- **WHEN** the earliest seated player completes a successful move
- **THEN** the current turn becomes the next seated player in join order
- **AND** after the last seated player in that order moves successfully, the turn returns to the earliest seated player

#### Scenario [SC-MOVE-03]: Solo seated player keeps the turn after moving

- **GIVEN** a tourist room in phase `playing` with exactly one seated player whose turn it is
- **WHEN** that player completes a successful move
- **THEN** the current turn remains that same seated player

#### Scenario [SC-MOVE-16]: Permanent leave advances the turn

- **GIVEN** a tourist room with at least two seated players and the current turn belongs to one of them
- **WHEN** that current-turn player permanently leaves (consented leave or grace timeout)
- **THEN** the current turn advances to the next remaining seated player in join order
- **AND** the departed seat is no longer in the turn rotation

#### Scenario [SC-MOVE-17]: Offline grace keeps the turn waiting

- **GIVEN** a seated player whose turn it is and who disconnects unexpectedly within reconnect grace
- **WHEN** other clients remain in the room during the grace
- **THEN** the synchronized current turn stays on that offline seat
- **AND** no other seated player becomes current turn solely because of that disconnect

#### Scenario [SC-MOVE-19]: Mid-game seat appends to turn order

- **GIVEN** a tourist room in phase `playing` with two seated players in known turn order
- **WHEN** a third authenticated client joins and receives a seat
- **THEN** that new seat is appended to the end of the turn order
- **AND** the current turn does not jump to the newcomer solely because of the join

### Requirement: Authoritative one-step tourist move

The server SHALL accept a move message only when the room start phase is `playing`, and only from the seated client whose turn it is that identifies one of that client’s own pieces and a target cell. A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, task/field, or center cell), and MUST NOT land on a cell occupied by any piece in the room (including the mover’s other pieces). Holes and cells outside the playable layout MUST be rejected. Moves from a client that is not seated, not the current turn, not in phase `playing`, or that target another player’s piece MUST be rejected without changing piece positions or the current turn. On acceptance the server MUST update the piece’s synchronized row and column and advance the turn per the turn-order requirement. There is no pass action in this change.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position and turn

- **GIVEN** the room phase is `playing`, it is a seated player’s turn, and one of that player’s pieces has a free orthogonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that neighbor cell
- **THEN** the server updates that piece’s synchronized row and column to the target
- **AND** the turn advances according to turn order
- **AND** every client observes the new piece position

#### Scenario [SC-MOVE-05]: Diagonal step is allowed

- **GIVEN** the room phase is `playing`, it is a seated player’s turn, and one of that player’s pieces has a free diagonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that diagonal neighbor
- **THEN** the server accepts the move and updates the piece’s synchronized position

#### Scenario [SC-MOVE-06]: Occupied cell is rejected

- **GIVEN** the room phase is `playing`, it is a seated player’s turn, and a neighbor playable cell is occupied by any tourist piece (own or another player’s)
- **WHEN** that player submits a move onto that occupied cell
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-07]: Non-playable cell is rejected

- **GIVEN** the room phase is `playing` and it is a seated player’s turn
- **WHEN** that player submits a move onto a hole or otherwise non-playable cell adjacent in grid coordinates
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-08]: Start and center cells are walkable

- **GIVEN** the room phase is `playing`, it is a seated player’s turn, and a piece has a free neighbor that is a start cell or a center cell of the tourist layout
- **WHEN** that player submits a move onto that cell
- **THEN** the server accepts the move and updates the piece’s synchronized position

#### Scenario [SC-MOVE-09]: Move out of turn is rejected

- **GIVEN** the room phase is `playing` and a seated player whose turn it is not
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

## ADDED Requirements

### Requirement: Client move chrome only in playing

While the start phase is not `playing`, the current-turn seated client MUST NOT show white selection / red destination move chrome as a way to commit moves, and MUST NOT submit moves from board or strip activation. When the phase becomes `playing`, existing local selection and hint rules of this capability apply.

#### Scenario [SC-MOVE-20]: No move chrome before playing

- **GIVEN** it would be the user’s turn as a seated player but the room phase is `waiting` or `countdown`
- **WHEN** the user views the board
- **THEN** white selection and red destination chrome for committing a move are not available
- **AND** board activation does not submit a move
