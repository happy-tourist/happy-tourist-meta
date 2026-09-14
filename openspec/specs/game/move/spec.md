# game/move Specification

## Purpose

Очередь хода и перемещение фигурок туриста на одну соседнюю клетку: авторитетная валидация на сервере, локальные подсказки только ходящему, плавная анимация у всех клиентов.

Связанные capability: рассадка и pieces — `game/pieces`; геометрия поля — `game/board`; reconnect grace — `game/presence` / pieces.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-01 | covered (server mocha) |
| SC-MOVE-02 | covered (server mocha) |
| SC-MOVE-03 | covered (server mocha) |
| SC-MOVE-04 | covered (server mocha) |
| SC-MOVE-05 | covered (server mocha) |
| SC-MOVE-06 | covered (server mocha) |
| SC-MOVE-07 | covered (server mocha) |
| SC-MOVE-08 | covered (server mocha) |
| SC-MOVE-09 | covered (server mocha) |
| SC-MOVE-10 | covered (server mocha) |
| SC-MOVE-11 | covered (client UX GamePage) |
| SC-MOVE-12 | covered (client UX GamePage) |
| SC-MOVE-13 | covered (client UX GamePage) |
| SC-MOVE-14 | covered (client UX GamePage) |
| SC-MOVE-15 | covered (client UX GamePage) |
| SC-MOVE-16 | covered (server mocha) |
| SC-MOVE-17 | covered (server mocha) |

## Requirements

### Requirement: Turn order among seated players

As soon as at least one seated player exists in the tourist room, the server SHALL maintain a synchronized current-turn seat among seated players. Turn order MUST follow join order of seated players (earliest seated first). After a successful move by the current player, the turn MUST pass immediately to the next seated player in that order, wrapping to the earliest when the end is reached. When only one seated player remains, that player MUST receive the turn again after their own successful move. When a new player becomes seated before the room seating lock (`started`), they MUST be appended to the end of the turn order. When the seated player whose turn it is permanently leaves (consented leave or reconnect grace timeout), the turn MUST advance immediately to the next remaining seated player. While a seated player is offline within reconnect grace, the current turn MUST remain on that seat until they reconnect and move or the seat is removed.

#### Scenario [SC-MOVE-01]: First seated player holds the turn

- **GIVEN** a tourist room with no seated players
- **WHEN** the first authenticated client joins and receives a seat
- **THEN** that seat is the current turn
- **AND** every client in the room observes the synchronized current-turn indicator for that seat

#### Scenario [SC-MOVE-02]: Join order defines the rotation

- **GIVEN** a tourist room with two or more seated players in known join order
- **WHEN** the earliest seated player completes a successful move
- **THEN** the current turn becomes the next seated player in join order
- **AND** after the last seated player in that order moves successfully, the turn returns to the earliest seated player

#### Scenario [SC-MOVE-03]: Solo seated player keeps the turn after moving

- **GIVEN** a tourist room with exactly one seated player whose turn it is
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

### Requirement: Authoritative one-step tourist move

The server SHALL accept a move message from the seated client whose turn it is that identifies one of that client’s own pieces and a target cell. A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, task/field, or center cell), and MUST NOT land on a cell occupied by any piece in the room (including the mover’s other pieces). Holes and cells outside the playable layout MUST be rejected. Moves from a client that is not seated, not the current turn, or that target another player’s piece MUST be rejected without changing piece positions or the current turn. On acceptance the server MUST update the piece’s synchronized row and column and advance the turn per the turn-order requirement. There is no pass action in this change.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position and turn

- **GIVEN** it is a seated player’s turn and one of that player’s pieces has a free orthogonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that neighbor cell
- **THEN** the server updates that piece’s synchronized row and column to the target
- **AND** the turn advances according to turn order
- **AND** every client observes the new piece position

#### Scenario [SC-MOVE-05]: Diagonal step is allowed

- **GIVEN** it is a seated player’s turn and one of that player’s pieces has a free diagonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that diagonal neighbor
- **THEN** the server accepts the move and updates the piece’s synchronized position

#### Scenario [SC-MOVE-06]: Occupied cell is rejected

- **GIVEN** it is a seated player’s turn and a neighbor playable cell is occupied by any tourist piece (own or another player’s)
- **WHEN** that player submits a move onto that occupied cell
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-07]: Non-playable cell is rejected

- **GIVEN** it is a seated player’s turn
- **WHEN** that player submits a move onto a hole or otherwise non-playable cell adjacent in grid coordinates
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-08]: Start and center cells are walkable

- **GIVEN** it is a seated player’s turn and a piece has a free neighbor that is a start cell or a center cell of the tourist layout
- **WHEN** that player submits a move onto that cell
- **THEN** the server accepts the move and updates the piece’s synchronized position

#### Scenario [SC-MOVE-09]: Move out of turn is rejected

- **GIVEN** a seated player whose turn it is not
- **WHEN** that player submits a move for one of their pieces
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-10]: Spectator cannot move

- **GIVEN** a spectator client in a tourist room that has at least one seated player
- **WHEN** that spectator attempts to submit a move
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

### Requirement: Local selection and move hints for the current player only

While it is a seated client’s own turn, that client SHALL be able to select one of their own pieces by activating it on the board or the matching slot in their personal strip. The selected piece’s cell MUST show a white selection outline, and every currently legal destination for that piece MUST show a red outline. Selection and red destination hints MUST be local UX only (not authoritative truth) and MUST NOT be shown to other seated players or spectators. The current player MAY change selection among their own pieces until a move is submitted. Clients that are not the current-turn seated player MUST NOT show white or red move chrome and MUST NOT submit a move as a result of board activation. Activating a red destination cell MUST submit the corresponding move for the selected piece.

#### Scenario [SC-MOVE-11]: Current player sees white selection chrome

- **GIVEN** it is the user’s turn as a seated player with at least one piece
- **WHEN** the user selects one of their pieces on the board or in the personal strip
- **THEN** that piece’s cell shows a white selection outline on that user’s client
- **AND** other clients do not show that white selection outline as a shared requirement

#### Scenario [SC-MOVE-12]: Current player sees red legal destinations

- **GIVEN** it is the user’s turn and one of their pieces is selected
- **WHEN** legal destinations exist for that piece under the move rules
- **THEN** those destination cells show a red outline on that user’s client only
- **AND** occupied or non-playable neighbors are not outlined as destinations

#### Scenario [SC-MOVE-13]: Reselection before commit

- **GIVEN** it is the user’s turn and one of their pieces is already selected
- **WHEN** the user selects a different own piece before submitting a move
- **THEN** the white outline moves to the newly selected piece’s cell
- **AND** red destination outlines update for the newly selected piece

### Requirement: Smooth move animation on all clients

When a piece’s synchronized cell changes because of an accepted move, every client that displays the board MUST animate that piece smoothly from its previous cell to the new cell. While the current-turn client’s own piece is animating after they submitted the move, that client MUST ignore further board and strip activations for selection or move submission until the animation finishes. Other clients MUST still observe the animated travel of the piece.

#### Scenario [SC-MOVE-14]: All clients animate the piece travel

- **GIVEN** a successful move updates a piece’s synchronized cell
- **WHEN** seated players and spectators view the board
- **THEN** each of those clients shows the piece sliding from the old cell to the new cell

#### Scenario [SC-MOVE-15]: Current player input ignored during animation

- **GIVEN** the current player has just submitted a move and that piece’s travel animation is still in progress on their client
- **WHEN** the user activates a board tile, piece, or strip slot during that animation
- **THEN** the client does not change selection, show new hints, or send another move as a result of that activation
