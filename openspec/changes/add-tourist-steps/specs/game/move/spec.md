## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-33 | pending (server mocha) |
| SC-MOVE-34 | pending (server mocha) |
| SC-MOVE-35 | pending (server mocha) |
| SC-MOVE-36 | pending (server mocha) |
| SC-MOVE-37 | pending (server mocha) |
| SC-MOVE-38 | pending (server mocha) |
| SC-MOVE-39 | pending (server mocha) |
| SC-MOVE-40 | pending (server mocha) |
| SC-MOVE-41 | pending (server mocha) |
| SC-MOVE-42 | pending (server mocha) |
| SC-MOVE-43 | pending (server mocha) |
| SC-MOVE-44 | pending (server mocha) |
| SC-MOVE-45 | pending (client UX) |
| SC-MOVE-46 | pending (client UX — keep-focus after move) |
| SC-MOVE-04 | pending-update (server mocha — move no longer advances turn) |
| SC-MOVE-28 | pending-update (server mocha — timeout + open peek = wrong, KEEP tile) |

Related: board peek / removed tiles — `game/board`; presence counters / end-turn — `game/presence`; solo finish — `game/finish`.

## ADDED Requirements

### Requirement: Private step and peek budgets per seat

While phase is `playing`, the server SHALL maintain for each non-finished seated player a private **steps** budget and a private **peeks** budget that are visible only to that seat’s client (not to other seated players or spectators). At materialize into `playing` both budgets MUST start at **0**. Each time a seat becomes current turn while **two or more** eligible seats remain, the server MUST add exactly **1** step and exactly **1** peek to that seat’s budgets before play continues. Budgets MUST persist across that seat’s turns (remainder carries over). Reconnect within grace MUST restore the same budgets to the reclaiming client. Spectators MUST NOT receive step or peek budgets.

#### Scenario [SC-MOVE-33]: Budgets start at zero until first turn grant

- **GIVEN** a tourist room that has just entered phase `playing` with at least two non-finished seated players
- **WHEN** seated clients observe their private budgets before any turn grant animation settles
- **THEN** each non-current seat that has not yet been granted a turn this game has steps 0 and peeks 0
- **AND** the seat that becomes current turn receives +1 step and +1 peek as part of becoming current

#### Scenario [SC-MOVE-34]: Remainder carries and next turn adds one each

- **GIVEN** a non-finished seated player ends a multiplayer turn with 2 steps and 1 peek remaining
- **WHEN** that seat becomes current turn again while two or more eligible seats remain
- **THEN** that seat’s private budgets become 3 steps and 2 peeks
- **AND** other clients do not observe those budget values

### Requirement: Moves consume steps and do not advance the turn

A legal one-step move in phase `playing` MUST decrement the mover’s steps budget by exactly **1** when the seat is not in solo infinite-budget mode. The server MUST reject a move that would require a step while the seat’s steps budget is **0** (finite mode) without changing piece positions or the current turn. After an accepted move the current turn MUST **remain** on that seat; the turn MUST NOT advance solely because a move succeeded. Orthogonal/diagonal adjacency, occupancy, playable cells (including removed task cells that remain walkable per `game/board`), center finish, and finished/time-expired gates remain as previously specified except where this change modifies turn advancement.

#### Scenario [SC-MOVE-35]: Successful move spends one step and keeps the turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1 (finite mode), and a legal neighbor cell exists for one unfinished piece
- **WHEN** that player submits a legal move to that cell
- **THEN** the server updates the piece position
- **AND** that seat’s steps budget decreases by 1
- **AND** the current turn remains that same seat

#### Scenario [SC-MOVE-36]: Move with zero steps is rejected

- **GIVEN** it is a seated player’s turn in finite-budget mode with steps 0
- **WHEN** that player submits a move message
- **THEN** the server rejects the move
- **AND** piece positions, peeks budget, and the current turn remain unchanged

### Requirement: Player end-turn and auto end-turn when no actions remain

While phase is `playing`, at least two eligible seats remain, and it is a non-finished non-time-expired seat’s turn, that client MUST be able to submit an end-turn action. On acceptance the server MUST advance the turn to the next eligible seat in join order and grant that seat +1 step and +1 peek per the budget requirement. When the current seat has **no available actions** — cannot legally move (steps 0 or no legal destinations) **and** cannot legally peek (peeks 0, or already used the one peek this turn, or no own unfinished piece stands on a still-present task cell) — the server MUST auto-advance the turn the same way without waiting for the deadline. Solo infinite-budget mode MUST NOT offer or require end-turn advancement among seats.

#### Scenario [SC-MOVE-37]: End-turn advances and grants the next seat

- **GIVEN** a tourist room in phase `playing` with two eligible seated players A then B, it is A’s turn, and A still has steps or peeks remaining
- **WHEN** A submits end-turn
- **THEN** the current turn becomes B
- **AND** B’s private budgets increase by 1 step and 1 peek
- **AND** no piece positions change solely because of end-turn

#### Scenario [SC-MOVE-38]: Auto end-turn when no actions remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0 and peeks 0 (or peeks remain but peek is already used this turn or no piece stands on a present task cell), so no legal move or peek remains
- **WHEN** the server evaluates available actions
- **THEN** the turn advances to the next eligible seated player
- **AND** that next seat receives +1 step and +1 peek

### Requirement: One peek attempt per multiplayer turn

While two or more eligible seats remain, a current-turn seat MAY successfully open at most **one** peek attempt per turn (see `game/board`). After that attempt resolves (correct, incorrect, or timeout-forced incorrect), further peek attempts by that seat MUST be rejected until the seat’s next turn, even if the peeks budget is still greater than zero. Moves with remaining steps remain allowed after a peek resolves.

#### Scenario [SC-MOVE-39]: Second peek in the same turn is rejected

- **GIVEN** it is a multiplayer turn and the current seat has already resolved one peek this turn and still has peeks ≥ 1
- **WHEN** that seat attempts another peek
- **THEN** the server rejects the peek
- **AND** budgets and removed tiles remain unchanged by that attempt

### Requirement: Solo infinite steps and peeks

When phase is `playing` and exactly one seated player remains eligible to move, the server SHALL set that seat’s steps and peeks budgets to an **infinite** mode (no numeric exhaustion), MUST NOT apply the per-turn +1/+1 grant, MUST NOT enforce the one-peek-per-turn limit, and MUST NOT accept or require end-turn to keep playing. The five-minute solo deadline and time-expired lock of this capability remain the time limit; finishing all pieces follows `game/finish`. The solo client MUST be informed via a modal that steps and peeks are unlimited for the remainder of the match. Other clients MUST NOT see that seat’s budget values.

#### Scenario [SC-MOVE-40]: Becoming solo unlocks infinite budgets

- **GIVEN** a tourist room in phase `playing` where every other seated player has either a finish place or has permanently left, leaving exactly one non-finished seated player
- **WHEN** that condition becomes true
- **THEN** that seat’s steps and peeks are infinite
- **AND** that client is shown the solo unlimited-resources modal
- **AND** the synchronized five-minute solo deadline applies as previously specified

#### Scenario [SC-MOVE-41]: Solo needs no end-turn

- **GIVEN** a non-finished seated player under solo infinite budgets
- **WHEN** that player views Game controls for ending a turn
- **THEN** the end-turn control is not offered
- **AND** successful moves do not pass the turn to another seat

### Requirement: Turn timeout forces open peek as incorrect

When a multiplayer 60-second turn deadline elapses, the server SHALL advance the turn as previously specified for timeout among eligible seats. If that seat had an unresolved peek modal open, the server MUST resolve that peek as **incorrect** (no step reward) and MUST NOT remove the peeked task tile (same as «Неправильно» in `game/board`) before or as part of advancing the turn.

#### Scenario [SC-MOVE-42]: Timeout with open peek is incorrect and keeps the tile

- **GIVEN** it is a multiplayer turn, the current seat has an unresolved peek on a task cell, and the 60-second deadline elapses
- **WHEN** the server applies the turn timeout
- **THEN** that peek is resolved as incorrect with no steps added
- **AND** that task tile remains present for every client
- **AND** the peeks budget decreases by 1 in finite mode
- **AND** the turn advances to the next eligible seat

### Requirement: Keep local selection after a successful non-finishing move

After the current-turn client successfully submits a move that leaves that piece unfinished (not a center finish), that client’s Game UI MUST keep local selection on the same piece side when the turn remains theirs. After the piece move animation finishes, if the seat still has steps remaining (or infinite budgets), red legal destination hints MUST show for that selected piece from its new cell without requiring the user to re-select it. Selection MUST still clear when the piece finishes, when the seat loses the turn, when phase leaves `playing`, or when time-expired / finished-seat rules clear interaction. Other clients MUST NOT observe this selection.

#### Scenario [SC-MOVE-46]: Selection and step hints remain after a multi-step move

- **GIVEN** it is the user’s turn with steps ≥ 2 (finite) or infinite budgets, and one own unfinished piece is selected
- **WHEN** the user submits a successful non-center move for that piece and the move animation finishes while the turn remains theirs
- **THEN** that same piece remains selected with white selection chrome
- **AND** if steps remain (or infinite), red legal destination outlines show from the new cell without a second selection click
- **AND** other clients do not show that selection chrome

## MODIFIED Requirements

### Requirement: Turn order among seated players

As soon as at least one seated player exists in the tourist room, the server SHALL maintain a synchronized current-turn seat among seated players that are eligible to move. A seat with a finish place (`game/finish`) MUST NOT be eligible to hold or receive the current turn while any non-finished seated player remains. Turn order MUST follow join order of seated players (earliest seated first), skipping finished seats. After a successful **end-turn**, an **auto end-turn** (no available actions), or a multiplayer **turn timeout**, the turn MUST pass to the next non-finished seated player in that order, wrapping to the earliest non-finished when the end is reached. A successful **move alone MUST NOT** advance the turn. When only one non-finished seated player remains, that player keeps the turn under solo infinite-budget rules without end-turn rotation. New seats are assigned only while phase is `waiting` (`game/pieces`); therefore no seat is appended to the turn order from a join during `countdown` or `playing`. When the seated player whose turn it is permanently leaves (consented leave or reconnect grace timeout), the turn MUST advance immediately to the next remaining non-finished seated player. While a non-finished seated player is offline within reconnect grace, the current turn MUST remain on that seat until they reconnect and act or the seat is removed. While a finished seat is offline within reconnect grace, the current turn MUST NOT remain on that finished seat: the turn MUST advance to the next non-finished seated player if any exist. When every remaining seated player is finished, the server MUST NOT require a move-capable current turn for gameplay. Having a current-turn seat MUST NOT by itself allow moves before start phase `playing` (see authoritative move requirement and `game/start`).

#### Scenario [SC-MOVE-01]: First seated player holds the turn

- **GIVEN** a tourist room with no seated players
- **WHEN** the first authenticated client joins and receives a seat
- **THEN** that seat is the current turn
- **AND** every client in the room observes the synchronized current-turn indicator for that seat

#### Scenario [SC-MOVE-02]: Join order defines the rotation

- **GIVEN** a tourist room with two or more non-finished seated players in known join order and phase `playing`
- **WHEN** the earliest non-finished seated player completes end-turn (or auto end-turn / timeout)
- **THEN** the current turn becomes the next non-finished seated player in join order
- **AND** after the last non-finished seated player in that order ends their turn, the turn returns to the earliest non-finished seated player

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
- **WHEN** B completes end-turn (or auto end-turn / timeout)
- **THEN** the current turn returns to B (A is skipped)
- **AND** A never becomes current turn while finished and B remains non-finished

#### Scenario [SC-MOVE-22]: Finished offline seat does not hold the turn

- **GIVEN** a finished seated player would be next in join order and that finished seat is offline within reconnect grace, and at least one non-finished seated player remains
- **WHEN** the previous non-finished player completes end-turn (or auto end-turn / timeout)
- **THEN** the current turn advances to the next non-finished seated player
- **AND** the offline finished seat does not hold the current turn

#### Scenario [SC-MOVE-43]: Move does not rotate the turn in multiplayer

- **GIVEN** a tourist room in phase `playing` with two non-finished seated players and it is the earliest player’s turn with steps ≥ 1
- **WHEN** that player completes a successful non-center move
- **THEN** the current turn remains that same player
- **AND** the next player does not become current solely because of that move

### Requirement: Authoritative one-step tourist move

The server SHALL accept a move message only when the room start phase is `playing`, and only from a non-finished seated client who is not time-expired, whose turn it is, that identifies one of that client’s own unfinished pieces and a target cell, and who has an available step in finite mode (or infinite solo mode). A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, task/field including removed-but-walkable task cells, or center cell), and MUST NOT land on a cell occupied by any unfinished piece in the room (including the mover’s other unfinished pieces). Finished pieces MUST NOT occupy cells and MUST NOT be move targets as pieces. Holes that were never playable and cells outside the playable layout MUST be rejected. Moves from a client that is not seated, is finished, is time-expired, not the current turn, not in phase `playing`, that target another player’s piece, that target an already finished own piece, or that lack a step in finite mode MUST be rejected without changing piece positions or the current turn. On acceptance onto a non-center playable cell the server MUST update the piece’s synchronized row and column, decrement steps by 1 in finite mode, and **MUST NOT** advance the turn. On acceptance onto a center cell the server MUST finish the piece per `game/finish` (piece leaves board occupancy), decrement steps by 1 in finite mode, and **MUST NOT** advance the turn solely because of that move. Player-initiated end-turn and auto end-turn / timeout rules of this capability advance the turn without requiring a move.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position and turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1, and one of that player’s unfinished pieces has a free orthogonal neighbor playable non-center cell
- **WHEN** that player submits a move for that piece to that neighbor cell
- **THEN** the server updates that piece’s synchronized row and column to the target
- **AND** the turn does not advance solely because of that move
- **AND** every client observes the new piece position

#### Scenario [SC-MOVE-05]: Diagonal step is allowed

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1, and one of that player’s unfinished pieces has a free diagonal neighbor playable cell
- **WHEN** that player submits a move for that piece to that diagonal neighbor
- **THEN** the server accepts the move and applies the non-center or center outcome as defined above
- **AND** the turn does not advance solely because of that move

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

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1, and an unfinished piece has a free neighbor that is a start cell or a center cell of the tourist layout
- **WHEN** that player submits a move onto that cell
- **THEN** the server accepts the move
- **AND** if the target is a start cell, the piece’s synchronized position updates to that cell
- **AND** if the target is a center cell, the piece is finished per `game/finish` and leaves board occupancy
- **AND** the turn does not advance solely because of that move

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

#### Scenario [SC-MOVE-44]: Center finish spends a step without rotating turn

- **GIVEN** the room phase is `playing`, it is a multiplayer turn with steps ≥ 1, and an unfinished piece has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** the piece is finished per `game/finish`
- **AND** steps decrease by 1 in finite mode
- **AND** the current turn remains that seat until end-turn, auto end-turn, or timeout

### Requirement: Turn timeout advances among eligible seats

When phase is `playing`, at least two seated players remain eligible to move, and the current-turn seat’s 60-second deadline elapses, the server SHALL advance the turn to the next eligible seated player in join order without requiring a move, without moving any piece. An open peek MUST be force-resolved as incorrect without removing the task tile (`game/board`). The newly current seat MUST receive a fresh 60-second deadline and the +1 step / +1 peek grant. This timeout pass MUST apply even if the timed-out seat is still offline within reconnect grace.

#### Scenario [SC-MOVE-28]: Sixty-second timeout passes the turn

- **GIVEN** a tourist room in phase `playing` with two or more eligible seated players and a current-turn seat whose 60-second deadline has just elapsed
- **WHEN** the server applies the turn timeout
- **THEN** the current turn becomes the next eligible seated player in join order
- **AND** no piece positions change solely because of that timeout
- **AND** an open peek, if any, is resolved as incorrect without removing that task tile
- **AND** the new current-turn seat has a fresh 60-second deadline and +1 step / +1 peek

### Requirement: Solo endgame five-minute budget

When phase is `playing` and exactly one seated player remains eligible to move (all other seats either hold a finish place or have permanently left), the server SHALL give that seat a synchronized turn deadline of exactly **five minutes** from the moment that condition becomes true, replacing any prior multiplayer turn deadline, and SHALL apply solo infinite steps/peeks as specified above. While that solo budget runs, the seat remains current turn after its own successful moves without resetting the remaining solo budget to a new five minutes on each move and without +1/+1 turn grants; if the player finishes all four pieces before the deadline, normal finish rules apply (`game/finish`) and the turn deadline MUST clear. Spectators and other clients MUST observe the same synchronized solo deadline.

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

#### Scenario [SC-MOVE-45]: Solo expiration still only time-locks

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under the five-minute solo budget with infinite steps/peeks
- **WHEN** the five-minute deadline elapses without finishing all pieces
- **THEN** that seat becomes time-expired and further moves and peeks are rejected
- **AND** exhaustion of steps or peeks is not used as a solo end condition
