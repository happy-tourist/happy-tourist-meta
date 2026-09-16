# game/move Specification

## Purpose

Очередь хода и перемещение фигурок туриста на одну соседнюю клетку: авторитетная валидация на сервере, локальные подсказки только ходящему, плавная анимация у всех клиентов. Вход на center завершает фишку (`game/finish`); finished-seats пропускаются в очереди хода. Пока `playing`, сервер ведёт deadline хода (60 с / соло 5 мин) и может сдвинуть ход или заблокировать time-expired без движения фигурок.

Связанные capability: фазы старта и gate ходов — `game/start`; рассадка и pieces — `game/pieces`; геометрия поля — `game/board`; reconnect grace и кольца — `game/presence`; финиш — `game/finish`; leave без confirm для time-expired — `game/leave`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-01 | covered-by-reuse (server mocha) |
| SC-MOVE-02 | covered (server mocha — skip finished via SC-MOVE-21) |
| SC-MOVE-03 | covered-by-reuse (server mocha — solo; finished skip via SC-MOVE-21) |
| SC-MOVE-04 | covered (server mocha) |
| SC-MOVE-05 | covered (server mocha) |
| SC-MOVE-06 | covered (server mocha) |
| SC-MOVE-07 | covered (server mocha) |
| SC-MOVE-08 | covered (server mocha — center finishes piece / SC-FINISH-01) |
| SC-MOVE-09 | covered (server mocha) |
| SC-MOVE-10 | covered (server mocha) |
| SC-MOVE-11 | covered (client UX GamePage) |
| SC-MOVE-12 | covered (client UX GamePage) |
| SC-MOVE-13 | covered (client UX GamePage) |
| SC-MOVE-14 | covered (client UX GamePage) |
| SC-MOVE-15 | covered (client UX GamePage) |
| SC-MOVE-16 | covered-by-reuse (server mocha) |
| SC-MOVE-17 | covered-by-reuse (server mocha — active offline) |
| SC-MOVE-18 | covered (server mocha) |
| SC-MOVE-19 | covered (server mocha — playing join is spectator; turn order unchanged) |
| SC-MOVE-20 | covered (client UX) |
| SC-MOVE-21 | covered (server mocha) |
| SC-MOVE-22 | covered (server mocha) |
| SC-MOVE-23 | covered (server mocha) |
| SC-MOVE-24 | covered (client UX) |
| SC-MOVE-25 | covered (server mocha) |
| SC-MOVE-26 | covered (server mocha) |
| SC-MOVE-27 | covered (server mocha) |
| SC-MOVE-28 | covered (server mocha) |
| SC-MOVE-29 | covered (server mocha) |
| SC-MOVE-30 | covered (server mocha) |
| SC-MOVE-31 | covered (client timeout modal) |
| SC-MOVE-32 | covered (client isInteractive lock) |

## Requirements

### Requirement: Turn order among seated players

As soon as at least one seated player exists in the tourist room, the server SHALL maintain a synchronized current-turn seat among seated players that are eligible to move. A seat with a finish place (`game/finish`) MUST NOT be eligible to hold or receive the current turn while any non-finished seated player remains. Turn order MUST follow join order of seated players (earliest seated first), skipping finished seats. After a successful move by the current player, the turn MUST pass immediately to the next non-finished seated player in that order, wrapping to the earliest non-finished when the end is reached. When only one non-finished seated player remains, that player MUST receive the turn again after their own successful move. New seats are assigned only while phase is `waiting` (`game/pieces`); therefore no seat is appended to the turn order from a join during `countdown` or `playing`. When the seated player whose turn it is permanently leaves (consented leave or reconnect grace timeout), the turn MUST advance immediately to the next remaining non-finished seated player. While a non-finished seated player is offline within reconnect grace, the current turn MUST remain on that seat until they reconnect and move or the seat is removed. While a finished seat is offline within reconnect grace, the current turn MUST NOT remain on that finished seat: the turn MUST advance to the next non-finished seated player if any exist. When every remaining seated player is finished, the server MUST NOT require a move-capable current turn for gameplay. Having a current-turn seat MUST NOT by itself allow moves before start phase `playing` (see authoritative move requirement and `game/start`).

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
- **WHEN** a third authenticated client joins
- **THEN** that client receives no seat
- **AND** the turn order among the existing two seats remains unchanged
- **AND** the current turn does not change solely because of that join

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

The server SHALL accept a move message only when the room start phase is `playing`, and only from a non-finished seated client who is not time-expired, whose turn it is, that identifies one of that client’s own unfinished pieces and a target cell. A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, task/field, or center cell), and MUST NOT land on a cell occupied by any unfinished piece in the room (including the mover’s other unfinished pieces). Finished pieces MUST NOT occupy cells and MUST NOT be move targets as pieces. Holes and cells outside the playable layout MUST be rejected. Moves from a client that is not seated, is finished, is time-expired, not the current turn, not in phase `playing`, that target another player’s piece, or that target an already finished own piece MUST be rejected without changing piece positions or the current turn. On acceptance onto a non-center playable cell the server MUST update the piece’s synchronized row and column and advance the turn per the turn-order requirement. On acceptance onto a center cell the server MUST finish the piece per `game/finish` (piece leaves board occupancy) and advance the turn per the turn-order requirement. There is no player-initiated pass action; only the authoritative turn-timeout rules of this capability advance the turn without a move.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position and turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn, and one of that player’s unfinished pieces has a free orthogonal neighbor playable non-center cell
- **WHEN** that player submits a move for that piece to that neighbor cell
- **THEN** the server updates that piece’s synchronized row and column to the target
- **AND** the turn advances according to turn order
- **AND** every client observes the new piece position

#### Scenario [SC-MOVE-05]: Diagonal step is allowed

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn, and one of that player’s unfinished pieces has a free diagonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that diagonal neighbor
- **THEN** the server accepts the move and applies the non-center or center outcome as defined above

#### Scenario [SC-MOVE-06]: Occupied cell is rejected

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn, and a neighbor playable cell is occupied by any unfinished tourist piece (own or another player’s)
- **WHEN** that player submits a move onto that occupied cell
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-07]: Non-playable cell is rejected

- **GIVEN** the room phase is `playing` and it is a seated non-finished non-time-expired player’s turn
- **WHEN** that player submits a move onto a hole or otherwise non-playable cell adjacent in grid coordinates
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-08]: Start and center cells are walkable

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn, and an unfinished piece has a free neighbor that is a start cell or a center cell of the tourist layout
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

### Requirement: Authoritative turn deadline while playing

While the start phase is `playing` and there is an eligible current-turn seat that is not time-expired, the server SHALL maintain a synchronized turn deadline for that seat. When two or more seated players are eligible to move (non-finished and not time-expired), each time that seat becomes current turn the deadline MUST be exactly **60 seconds** from that moment. The deadline MUST keep counting down while the current-turn seat is offline within reconnect grace. While the phase is `waiting` or `countdown`, the server MUST NOT run a turn deadline that auto-advances the turn. When the turn advances because of a successful move, a permanent leave, or a normal 60-second timeout with another eligible seat remaining, the new current-turn seat MUST receive a fresh deadline per this requirement (or the solo budget requirement when applicable).

#### Scenario [SC-MOVE-25]: Sixty-second deadline on each multiplayer turn

- **GIVEN** a tourist room in phase `playing` with at least two non-finished seated players and it becomes one of their turns
- **WHEN** every client observes the synchronized turn state
- **THEN** that current-turn seat has a turn deadline 60 seconds from becoming current
- **AND** every client in the room observes that synchronized deadline

#### Scenario [SC-MOVE-26]: Deadline ticks during reconnect grace

- **GIVEN** a non-finished current-turn seat in phase `playing` with a remaining turn deadline and at least one other eligible seat
- **WHEN** that current-turn player disconnects unexpectedly within reconnect grace
- **THEN** the synchronized turn remains on that offline seat
- **AND** the turn deadline continues to count down without pausing for the disconnect

#### Scenario [SC-MOVE-27]: No turn auto-pass before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with a current-turn seated player
- **WHEN** more than 60 seconds elapse without a move
- **THEN** the current turn does not advance solely because of elapsed wall time
- **AND** no turn-deadline auto-pass applies until phase `playing`

### Requirement: Turn timeout advances among eligible seats

When phase is `playing`, at least two seated players remain eligible to move, and the current-turn seat’s 60-second deadline elapses without a successful move, the server SHALL advance the turn to the next eligible seated player in join order exactly as after a successful move, without moving any piece. The newly current seat MUST receive a fresh 60-second deadline. This timeout pass MUST apply even if the timed-out seat is still offline within reconnect grace.

#### Scenario [SC-MOVE-28]: Sixty-second timeout passes the turn

- **GIVEN** a tourist room in phase `playing` with two or more eligible seated players and a current-turn seat whose 60-second deadline has just elapsed without a move
- **WHEN** the server applies the turn timeout
- **THEN** the current turn becomes the next eligible seated player in join order
- **AND** no piece positions change solely because of that timeout
- **AND** the new current-turn seat has a fresh 60-second deadline

### Requirement: Solo endgame five-minute budget

When phase is `playing` and exactly one seated player remains eligible to move (all other seats either hold a finish place or have permanently left), the server SHALL give that seat a synchronized turn deadline of exactly **five minutes** from the moment that condition becomes true, replacing any prior multiplayer turn deadline. While that solo budget runs, the seat remains current turn after its own successful moves without resetting the remaining solo budget to a new five minutes on each move; if the player finishes all four pieces before the deadline, normal finish rules apply (`game/finish`) and the turn deadline MUST clear. Spectators and other clients MUST observe the same synchronized solo deadline.

#### Scenario [SC-MOVE-29]: Solo remaining player gets five minutes

- **GIVEN** a tourist room in phase `playing` where every other seated player has either a finish place or has permanently left, leaving exactly one non-finished seated player
- **WHEN** that condition becomes true
- **THEN** that seat’s synchronized turn deadline is exactly five minutes from that moment
- **AND** every client observes that five-minute deadline

#### Scenario [SC-MOVE-30]: Successful solo finish clears the deadline

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under the five-minute solo budget
- **WHEN** that player finishes their last piece and receives a finish place
- **THEN** the turn deadline for that seat is cleared
- **AND** the seat follows finished-seat rules without a time-expired lock

### Requirement: Solo budget expiry locks moves

When the five-minute solo budget elapses without that seat finishing all pieces, the server SHALL mark that seat time-expired in synchronized state, MUST clear the active turn deadline, and MUST reject further move messages from that seat without changing piece positions. The room MUST NOT dispose or force-leave solely because of that expiry; the seat remains until consented leave or reconnect grace timeout as for other seats. The time-expired client’s Game UI MUST show a modal whose product Russian sense is that the tourists were not finished in time; closing the modal MUST leave the player in the room. That client MUST clear any local piece selection and move-destination hints and MUST NOT offer board/strip move interaction while time-expired. Other clients MUST NOT show that modal. Say and presence for the seat remain allowed as for a finished seat without a finish place.

#### Scenario [SC-MOVE-31]: Solo timeout locks and shows modal only to that player

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player whose five-minute solo budget has just elapsed, and at least one spectator or finished seated client also in the room
- **WHEN** the server applies solo budget expiry
- **THEN** that seat is marked time-expired in synced state
- **AND** move messages from that seat are rejected without changing pieces
- **AND** that player’s client shows the timeout modal
- **AND** other clients do not show that modal
- **AND** the room remains open with that seat still present after the modal is closed

#### Scenario [SC-MOVE-32]: Time-expired clears local move chrome

- **GIVEN** a seated client who had a piece selected with destination hints and whose seat has just become time-expired
- **WHEN** that client observes the time-expired state
- **THEN** selection and destination hints are cleared
- **AND** further board or strip activations do not submit moves

### Requirement: No move chrome for finished pieces or finished seats

While a seated client’s seat has a finish place, or while selecting among pieces, that client MUST NOT treat finished pieces or finished strip slots as selectable for moves, and MUST NOT show white selection / red destination chrome for finished pieces. Unfinished pieces of a non-finished current-turn seat keep existing local selection and hint rules.

#### Scenario [SC-MOVE-24]: Finished strip slot has no move chrome

- **GIVEN** it is the user’s turn as a non-finished seated player and one of their pieces is already finished
- **WHEN** the user views the board and personal strip
- **THEN** the finished piece is not on the board as a movable piece
- **AND** activating the finished strip slot does not show white/red move chrome or submit a move

### Requirement: Client move chrome only in playing

While the start phase is not `playing`, the current-turn seated client MUST NOT show white selection / red destination move chrome as a way to commit moves, and MUST NOT submit moves from board or strip activation. When the phase becomes `playing`, existing local selection and hint rules of this capability apply.

#### Scenario [SC-MOVE-20]: No move chrome before playing

- **GIVEN** it would be the user’s turn as a seated player but the room phase is `waiting` or `countdown`
- **WHEN** the user views the board
- **THEN** white selection and red destination chrome for committing a move are not available
- **AND** board activation does not submit a move

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
