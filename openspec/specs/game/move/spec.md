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
| SC-MOVE-04 | covered (server mocha — move no longer advances turn) |
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
| SC-MOVE-28 | covered (server mocha — timeout + open peek = wrong, KEEP tile) |
| SC-MOVE-29 | covered (server mocha) |
| SC-MOVE-30 | covered (server mocha) |
| SC-MOVE-31 | covered (client timeout modal) |
| SC-MOVE-32 | covered (client isInteractive lock) |
| SC-MOVE-33 | covered (server mocha) |
| SC-MOVE-34 | covered (server mocha) |
| SC-MOVE-35 | covered (server mocha) |
| SC-MOVE-36 | covered (server mocha) |
| SC-MOVE-37 | covered (server mocha) |
| SC-MOVE-38 | covered (server mocha — auto-end only without peeks-on-`*`) |
| SC-MOVE-39 | covered (server mocha — multi peek same turn allowed) |
| SC-MOVE-40 | covered (server mocha — solo peeks∞; already-current carries steps) |
| SC-MOVE-41 | covered (server mocha — solo no end-turn) |
| SC-MOVE-42 | covered (server mocha) |
| SC-MOVE-43 | covered (server mocha — move no longer advances turn) |
| SC-MOVE-44 | covered (server mocha) |
| SC-MOVE-45 | covered (server mocha — timer time-expired; distinct from step-loss) |
| SC-MOVE-46 | covered (client UX — keep-focus after move) |
| SC-MOVE-47 | covered (server mocha — keep turn when peeks and on live `*`) |
| SC-MOVE-48 | covered (server mocha — solo step-loss → timeExpired) |
| SC-MOVE-49 | covered (server mocha — move onto removed hole rejected) |
| SC-MOVE-50 | covered (server mocha — become-current into solo grants +1 step) |
| SC-MOVE-51 | covered (server mocha — land traps) |
| SC-MOVE-52 | covered (server mocha — trapped cannot move) |
| SC-MOVE-53 | covered (server mocha — trapped cannot peek) |
| SC-MOVE-54 | covered (server mocha — rescue costs 1 step) |
| SC-MOVE-55 | covered (server mocha — cannot rescue other seat) |
| SC-MOVE-56 | covered (server mocha — rescue needs adjacency + steps) |
| SC-MOVE-57 | covered (server mocha — return costs 1 step) |
| SC-MOVE-58 | covered (server mocha — return ring cells) |
| SC-MOVE-59 | covered (server mocha — return rejected without steps) |
| SC-MOVE-60 | covered (server mocha — all-jail auto reset) |
| SC-MOVE-61 | covered (server mocha — all-jail keeps turn) |
| SC-MOVE-62 | covered (server mocha — auto-end waits for rescue/return) |
| SC-MOVE-63 | covered (client UX — all-jail modal self only) |
| SC-MOVE-64 | covered (server mocha — solo same rules) |
| SC-MOVE-65 | covered (client UX — finish-block click → nearest legal center) |

Related: board peek / removed tiles / grilles — `game/board`; presence counters / end-turn — `game/presence`; solo finish / return — `game/finish`; trapped sync — `game/pieces`.

## Requirements

### Requirement: Private step and peek budgets per seat

While phase is `playing`, the server SHALL maintain for each non-finished seated player a private **steps** budget and a private **peeks** budget that are visible only to that seat’s client (not to other seated players or spectators). At materialize into `playing` both budgets MUST start at **0**. Each time a seat **becomes** current turn:

- while **two or more** eligible seats remain, the server MUST add exactly **1** step and exactly **1** peek to that seat’s budgets before play continues;
- while **exactly one** eligible seat remains (solo), the server MUST add exactly **1** step and MUST **not** add a peek (peeks are already infinite per the solo requirement).

Budgets MUST persist across that seat’s turns (remainder carries over). Becoming solo **without** a change of current turn (another seat permanently left or finished while this seat was already current) MUST **not** add another step solely for that reason. While the same seat remains current under solo rules, successful moves MUST NOT trigger further turn grants. Reconnect within grace MUST restore the same budgets to the reclaiming client. Spectators MUST NOT receive step or peek budgets.

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

#### Scenario [SC-MOVE-50]: Becoming current as the solo leftover grants one step

- **GIVEN** a tourist room in phase `playing` with two eligible seated players A then B, it is A’s turn, and B still has steps 0 and peeks 0 (B has not yet received a turn grant this game)
- **WHEN** A permanently leaves **or** A receives a finish place so that B is the only remaining eligible seat and becomes current turn
- **THEN** B’s peeks are infinite
- **AND** B’s steps become exactly 1
- **AND** B’s peeks counter is not increased by a finite +1 (infinite mode applies instead)
- **AND** B may legally move with that granted step

### Requirement: Turn order among seated players

As soon as at least one seated player exists in the tourist room, the server SHALL maintain a synchronized current-turn seat among seated players that are eligible to move. A seat with a finish place (`game/finish`) MUST NOT be eligible to hold or receive the current turn while any non-finished seated player remains. Turn order MUST follow join order of seated players (earliest seated first), skipping finished seats. After a successful **end-turn**, an **auto end-turn** (no available actions), or a multiplayer **turn timeout**, the turn MUST pass to the next non-finished seated player in that order, wrapping to the earliest non-finished when the end is reached. A successful **move alone MUST NOT** advance the turn. When only one non-finished seated player remains, that player keeps the turn under solo peeks-infinite / finite-steps rules without end-turn rotation. New seats are assigned only while phase is `waiting` (`game/pieces`); therefore no seat is appended to the turn order from a join during `countdown` or `playing`. When the seated player whose turn it is permanently leaves (consented leave or reconnect grace timeout), the turn MUST advance immediately to the next remaining non-finished seated player. While a non-finished seated player is offline within reconnect grace, the current turn MUST remain on that seat until they reconnect and act or the seat is removed. While a finished seat is offline within reconnect grace, the current turn MUST NOT remain on that finished seat: the turn MUST advance to the next non-finished seated player if any exist. When every remaining seated player is finished, the server MUST NOT require a move-capable current turn for gameplay. Having a current-turn seat MUST NOT by itself allow moves before start phase `playing` (see authoritative move requirement and `game/start`).

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

### Requirement: Moves consume steps and do not advance the turn

A legal one-step move in phase `playing` MUST decrement the mover’s steps budget by exactly **1** when the seat’s steps budget is finite (including solo). The server MUST reject a move that would require a step while the seat’s steps budget is **0** without changing piece positions or the current turn. After an accepted move the current turn MUST **remain** on that seat; the turn MUST NOT advance solely because a move succeeded. Orthogonal/diagonal adjacency, occupancy, playable cells (**excluding** removed task cells as landing targets per `game/board`), center finish, and finished/time-expired gates remain as previously specified except where this change modifies turn advancement.

#### Scenario [SC-MOVE-35]: Successful move spends one step and keeps the turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1, and a legal neighbor cell exists for one unfinished piece
- **WHEN** that player submits a legal move to that cell
- **THEN** the server updates the piece position
- **AND** that seat’s steps budget decreases by 1
- **AND** the current turn remains that same seat

#### Scenario [SC-MOVE-36]: Move with zero steps is rejected

- **GIVEN** it is a seated player’s turn with steps 0
- **WHEN** that player submits a move message
- **THEN** the server rejects the move
- **AND** piece positions, peeks budget, and the current turn remain unchanged

### Requirement: Player end-turn and auto end-turn when no actions remain

While phase is `playing`, at least two eligible seats remain, and it is a non-finished non-time-expired seat’s turn, that client MUST be able to submit an end-turn action. On acceptance the server MUST advance the turn to the next eligible seat in join order and grant that seat +1 step and +1 peek per the budget requirement. The server MUST **not** auto-advance while the current seat still has peeks ≥ 1 **and** at least one own unfinished piece stands on a still-present task cell (a legal peek remains available), even if steps are 0. The server MUST also treat a legal own rescue or a legal own return (when finish place is 0) as available actions. When the current seat has **no available actions** — cannot legally move (steps 0 or no legal destinations including no landing on removed holes) **and** does not have (peeks ≥ 1 and an own unfinished piece on a still-present task cell) **and** has no legal rescue or return — the server MUST auto-advance the turn the same way without waiting for the deadline. Solo mode MUST NOT offer or require end-turn advancement among seats.

#### Scenario [SC-MOVE-37]: End-turn advances and grants the next seat

- **GIVEN** a tourist room in phase `playing` with two eligible seated players A then B, it is A’s turn, and A still has steps or peeks remaining
- **WHEN** A submits end-turn
- **THEN** the current turn becomes B
- **AND** B’s private budgets increase by 1 step and 1 peek
- **AND** no piece positions change solely because of end-turn

#### Scenario [SC-MOVE-38]: Auto end-turn when no actions remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0 and peeks 0 (or peeks remain but no own unfinished piece stands on a still-present task cell), so no legal move and no available peek remain
- **WHEN** the server evaluates available actions
- **THEN** the turn advances to the next eligible seated player
- **AND** that next seat receives +1 step and +1 peek

#### Scenario [SC-MOVE-47]: Auto end-turn does not fire while peeks and a live task tile remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0, peeks ≥ 1, and an own unfinished piece on a still-present task cell
- **WHEN** the server evaluates available actions
- **THEN** the current turn remains that seat
- **AND** the player may open another peek while peeks remain

### Requirement: Multiple peeks per turn while peeks remain

While it is a seat’s turn in phase `playing`, that seat MAY successfully open as many peeks as its peeks budget (or solo infinite peeks) allows, each time an own unfinished piece stands on a still-present task cell (see `game/board`). The server MUST NOT enforce a one-peek-per-turn limit. Each successful open MUST still consume one peek in finite peeks mode when the peek resolves.

#### Scenario [SC-MOVE-39]: Second peek in the same turn is allowed

- **GIVEN** it is a multiplayer turn and the current seat has already resolved one peek this turn, still has peeks ≥ 1, and an own unfinished piece stands on a still-present task cell
- **WHEN** that seat attempts another peek
- **THEN** the server accepts the peek open
- **AND** the peek modal is shown to that client with the cell’s hidden reward

### Requirement: Authoritative one-step tourist move

The server SHALL accept a move message only when the room start phase is `playing`, and only from a non-finished seated client who is not time-expired, whose turn it is, that identifies one of that client’s own unfinished pieces and a target cell, and who has steps ≥ 1 in finite steps mode. A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, still-present task cell, or center cell), and MUST NOT land on a **removed** task cell, and MUST NOT land on a cell occupied by any unfinished piece in the room (including the mover’s other unfinished pieces). Finished pieces MUST NOT occupy cells and MUST NOT be move targets as pieces. Layout holes that were never playable, removed task cells, and cells outside the playable layout MUST be rejected as landing targets. Moves from a client that is not seated, is finished, is time-expired, not the current turn, not in phase `playing`, that target another player’s piece, that target an already finished own piece, or that lack a step MUST be rejected without changing piece positions or the current turn. On acceptance onto a non-center playable cell the server MUST update the piece’s synchronized row and column, decrement steps by 1, and **MUST NOT** advance the turn. On acceptance onto a center cell the server MUST finish the piece per `game/finish` (piece leaves board occupancy), decrement steps by 1, and **MUST NOT** advance the turn solely because of that move. A piece already standing on a removed task cell MAY leave that cell onto a legal neighbor. Player-initiated end-turn and auto end-turn / timeout rules of this capability advance the turn without requiring a move.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position without advancing turn

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
- **WHEN** that player submits a move onto a layout hole or otherwise non-playable cell adjacent in grid coordinates
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-49]: Landing on a removed task hole is rejected

- **GIVEN** the room phase is `playing`, it is a seated player’s turn with steps ≥ 1, and a neighbor cell is a removed task hole
- **WHEN** that player submits a move onto that removed cell
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

### Requirement: Authoritative turn deadline while playing

While the start phase is `playing` and there is an eligible current-turn seat that is not time-expired, the server SHALL maintain a synchronized turn deadline for that seat. When two or more seated players are eligible to move (non-finished and not time-expired), each time that seat becomes current turn the deadline MUST be exactly **60 seconds** from that moment. The deadline MUST keep counting down while the current-turn seat is offline within reconnect grace. While the phase is `waiting` or `countdown`, the server MUST NOT run a turn deadline that auto-advances the turn. When the turn advances because of end-turn, auto end-turn, a permanent leave, or a normal 60-second timeout with another eligible seat remaining, the new current-turn seat MUST receive a fresh deadline per this requirement (or the solo budget requirement when applicable).

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

When phase is `playing`, at least two seated players remain eligible to move, and the current-turn seat’s 60-second deadline elapses, the server SHALL advance the turn to the next eligible seated player in join order without requiring a move, without moving any piece. An open peek MUST be force-resolved as incorrect without removing the task tile (`game/board`). The newly current seat MUST receive a fresh 60-second deadline and the +1 step / +1 peek grant. This timeout pass MUST apply even if the timed-out seat is still offline within reconnect grace.

#### Scenario [SC-MOVE-28]: Sixty-second timeout passes the turn

- **GIVEN** a tourist room in phase `playing` with two or more eligible seated players and a current-turn seat whose 60-second deadline has just elapsed
- **WHEN** the server applies the turn timeout
- **THEN** the current turn becomes the next eligible seated player in join order
- **AND** no piece positions change solely because of that timeout
- **AND** an open peek, if any, is resolved as incorrect without removing that task tile
- **AND** the new current-turn seat has a fresh 60-second deadline and +1 step / +1 peek

### Requirement: Turn timeout forces open peek as incorrect

When a multiplayer 60-second turn deadline elapses, the server SHALL advance the turn as previously specified for timeout among eligible seats. If that seat had an unresolved peek modal open, the server MUST resolve that peek as **incorrect** (no step reward) and MUST NOT remove the peeked task tile (same as «Неправильно» in `game/board`) before or as part of advancing the turn.

#### Scenario [SC-MOVE-42]: Timeout with open peek is incorrect and keeps the tile

- **GIVEN** it is a multiplayer turn, the current seat has an unresolved peek on a task cell, and the 60-second deadline elapses
- **WHEN** the server applies the turn timeout
- **THEN** that peek is resolved as incorrect with no steps added
- **AND** that task tile remains present for every client
- **AND** the peeks budget decreases by 1 in finite mode
- **AND** the turn advances to the next eligible seat

### Requirement: Solo infinite peeks and finite steps

When phase is `playing` and exactly one seated player remains eligible to move, the server SHALL set that seat’s **peeks** budget to an **infinite** mode, MUST keep **steps** as a finite number (not infinite), MUST NOT accept or require end-turn to keep playing, and MUST NOT apply further turn grants while that same seat remains current under solo (successful moves do not re-grant). When another seat’s permanent leave or finish causes this seat to **become** current turn as the sole eligible seat, the private-budget grant for becoming current under solo (+1 step, no peek increment) MUST apply. When the seat was **already** current and merely becomes the sole eligible seat, steps MUST carry unchanged (no extra step solely for entering solo). The five-minute solo deadline remains; additionally, when steps reach **0** and no own unfinished piece stands on a still-present task cell, the server MUST mark that seat **time-expired** the same way as solo timer expiry (moves and peeks rejected). Finishing all pieces follows `game/finish`. The solo client MUST be informed via a modal that peeks are unlimited while steps remain finite. Other clients MUST NOT see that seat’s budget values.

#### Scenario [SC-MOVE-40]: Becoming solo while already current carries steps

- **GIVEN** a tourist room in phase `playing` where it is non-finished seated player A’s turn with a finite steps budget greater than 0, and every other seated player either receives a finish place or permanently leaves, leaving A as the only eligible seat **without** changing current turn away from A
- **WHEN** that condition becomes true
- **THEN** A’s peeks are infinite and steps remain the prior finite value (no extra +1 step solely for entering solo)
- **AND** that client is shown the solo peeks-unlimited modal
- **AND** the synchronized five-minute solo deadline applies as previously specified

#### Scenario [SC-MOVE-41]: Solo needs no end-turn

- **GIVEN** a non-finished seated player under solo infinite peeks
- **WHEN** that player views Game controls for ending a turn
- **THEN** the end-turn control is not offered
- **AND** successful moves do not pass the turn to another seat
- **AND** successful moves do not grant an additional +1 step solely because the seat remains solo current

#### Scenario [SC-MOVE-48]: Solo steps exhaustion without a live task tile time-locks

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under solo infinite peeks, steps 0, and no own unfinished piece on a still-present task cell
- **WHEN** the server evaluates solo end conditions
- **THEN** that seat becomes time-expired and further moves and peeks are rejected
- **AND** the client is shown the steps-exhausted end copy (distinct from the five-minute timer copy)

### Requirement: Solo endgame five-minute budget

When phase is `playing` and exactly one seated player remains eligible to move (all other seats either hold a finish place or have permanently left), the server SHALL give that seat a synchronized turn deadline of exactly **five minutes** from the moment that condition becomes true, replacing any prior multiplayer turn deadline, and SHALL apply solo infinite peeks with finite steps as specified above. While that solo budget runs, the seat remains current turn after its own successful moves without resetting the remaining solo budget to a new five minutes on each move and without +1/+1 turn grants; if the player finishes all four pieces before the deadline, normal finish rules apply (`game/finish`) and the turn deadline MUST clear. Spectators and other clients MUST observe the same synchronized solo deadline.

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

#### Scenario [SC-MOVE-45]: Solo timer expiration time-locks with timer copy

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under the five-minute solo budget with infinite peeks and remaining steps ≥ 1
- **WHEN** the five-minute deadline elapses without finishing all pieces
- **THEN** that seat becomes time-expired and further moves and peeks are rejected
- **AND** the client is shown the timer-expired end copy (distinct from the steps-exhausted copy)

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

While it is a seated client’s own turn, that client SHALL be able to select one of their own pieces by activating it on the board or the matching slot in their personal strip. The selected piece’s cell MUST show a white selection outline, and every currently legal destination for that piece MUST show a red outline. Legal **return-from-finish** ring cells shown while return-mode is active MUST use that **same** red destination outline presentation (not a distinct orange-only chrome). Selection and red destination hints MUST be local UX only (not authoritative truth) and MUST NOT be shown to other seated players or spectators. The current player MAY change selection among their own pieces until a move is submitted; changing selection MUST cancel return-mode without spending a step. Clients that are not the current-turn seated player MUST NOT show white or red move chrome and MUST NOT submit a move as a result of board activation. Activating a red destination cell MUST submit the corresponding move for the selected piece (or the return when return-mode is active).

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
- **AND** when return-mode is active with legal ring cells, those cells show the same red outline presentation

#### Scenario [SC-MOVE-13]: Reselection before commit

- **GIVEN** it is the user’s turn and one of their pieces is already selected
- **WHEN** the user selects a different own piece before submitting a move
- **THEN** the white outline moves to the newly selected piece’s cell
- **AND** red destination outlines update for the newly selected piece
- **AND** any prior return-mode is cancelled without spending a step

### Requirement: Finish-block click resolves to nearest legal center cell

The central finish area MAY be presented as one visual 2×2 block. While the client is submitting a move (not return-mode) and the selected piece has at least one legal destination among the four center cells, activating **any** point on that finish block MUST submit a move to the **nearest** of those **legal** center cells relative to the selected piece’s current cell (Chebyshev distance; ties broken by lower row, then lower column). The client MUST NOT require the user to hit a specific quadrant of the block matching that cell. If none of the four center cells is a legal destination for the selected piece, activating the finish block MUST NOT submit a move.

#### Scenario [SC-MOVE-65]: Any click on finish picks nearest legal center

- **GIVEN** it is the user’s turn, an own piece is selected, and exactly one of the four center cells is a legal one-step destination for that piece
- **WHEN** the user activates any point on the visual finish 2×2 block
- **THEN** the client submits a move to that legal center cell
- **AND** the submission does not depend on which quadrant of the block was clicked

### Requirement: Keep local selection after a successful non-finishing move

After the current-turn client successfully submits a move that leaves that piece unfinished (not a center finish), that client’s Game UI MUST keep local selection on the same piece side when the turn remains theirs. After the piece move animation finishes, if the seat still has steps remaining, red legal destination hints MUST show for that selected piece from its new cell without requiring the user to re-select it, and MUST NOT outline removed-task holes as destinations. Selection MUST still clear when the piece finishes, when the seat loses the turn, when phase leaves `playing`, or when time-expired / finished-seat rules clear interaction. Other clients MUST NOT observe this selection.

#### Scenario [SC-MOVE-46]: Selection and step hints remain after a multi-step move

- **GIVEN** it is the user’s turn with steps ≥ 2 and one own unfinished piece is selected
- **WHEN** the user submits a successful non-center move for that piece and the move animation finishes while the turn remains theirs
- **THEN** that same piece remains selected with white selection chrome
- **AND** if steps remain, red legal destination outlines show from the new cell without a second selection click and do not include removed-task holes
- **AND** other clients do not show that selection chrome

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

### Requirement: Landing on an unspent grille traps the piece

When phase is `playing` and the server accepts a legal move whose target cell still holds an unspent grille, the server SHALL consume the move’s step as usual, reveal and clear the hidden status of that grille, and mark that piece trapped. The piece MUST remain on that cell and MUST continue to occupy it for other pieces. The grille MUST become visible to all clients per `game/board`.

#### Scenario [SC-MOVE-51]: Land on grille traps and spends the step

- **GIVEN** it is a seated player’s turn with steps ≥ 1 and an unfinished free piece adjacent to a task cell that still holds an unspent grille
- **WHEN** that player submits a legal move onto that cell
- **THEN** that seat’s steps budget decreases by 1
- **AND** that piece is marked trapped in synced state
- **AND** the piece remains on that cell

### Requirement: Trapped pieces cannot move or peek

While a piece is trapped, the server MUST reject move and peek intents that target that piece. Other free unfinished pieces of the same seat MAY still move or peek under existing turn and budget rules.

#### Scenario [SC-MOVE-52]: Move rejected for trapped piece

- **GIVEN** it is a seated player’s turn and one of that player’s pieces is trapped
- **WHEN** that player submits a move identifying the trapped piece
- **THEN** the server rejects the move
- **AND** piece positions and steps do not change from that attempt

#### Scenario [SC-MOVE-53]: Peek rejected for trapped piece

- **GIVEN** it is a seated player’s turn with peeks remaining and a trapped own piece stands on a still-present task cell
- **WHEN** that player attempts to open a peek for that trapped piece
- **THEN** the server rejects the peek
- **AND** no reward is revealed

### Requirement: Rescue own adjacent trapped piece for one step

While it is a seated client’s own turn with steps ≥ 1, that client MAY rescue exactly one of their own trapped pieces when at least one of their own free unfinished pieces stands at Chebyshev distance 1 from the trapped piece’s cell (orthogonal or diagonal). A successful rescue MUST decrease steps by 1, clear trapped on the rescued piece, and clear the holding grille (rise and vanish ~1000 ms per `game/board`). Rescue of another seat’s piece MUST be rejected. Rescue MUST NOT permanently relocate the rescuer (approach animation is client presentation only).

#### Scenario [SC-MOVE-54]: Rescue spends one step and frees the piece

- **GIVEN** it is a seated player’s turn with steps ≥ 1, one own piece trapped on a cell, and another own free unfinished piece on a Chebyshev-1 neighbor of that cell
- **WHEN** that player submits a rescue for the trapped piece
- **THEN** steps decrease by 1
- **AND** the trapped piece becomes free
- **AND** the grille on that cell is cleared for all clients

#### Scenario [SC-MOVE-55]: Cannot rescue another player’s tourist

- **GIVEN** it is seat A’s turn with steps ≥ 1 and seat A has a free piece adjacent to seat B’s trapped piece
- **WHEN** seat A submits a rescue targeting seat B’s piece
- **THEN** the server rejects the rescue
- **AND** no steps are spent and no grille is cleared

#### Scenario [SC-MOVE-56]: Rescue rejected without adjacency or steps

- **GIVEN** it is a seated player’s turn and either steps are 0 or no own free unfinished piece is Chebyshev-1 from the trapped own piece
- **WHEN** that player submits a rescue for that trapped piece
- **THEN** the server rejects the rescue

### Requirement: Return a finished piece onto the center ring for one step

While it is a seated client’s own turn, that seat has finish place **0**, steps ≥ 1, and at least one own piece is finished, the client MAY return one finished piece onto a legal ring cell: any playable cell at Chebyshev distance 1 from the central 2×2 block (including diagonal corners relative to the block), that is not a removed-task hole and not occupied by an unfinished piece. A successful return MUST decrease steps by 1, mark that piece unfinished at the chosen cell, and place it on the board for all clients. Return MUST be offered per finished strip slot whenever steps and at least one legal ring cell exist (not only in a jail deadlock).

#### Scenario [SC-MOVE-57]: Return spends one step and places on ring

- **GIVEN** it is a seated player’s turn with steps ≥ 1, finish place 0, one finished own piece, and a free non-hole ring cell around the center block
- **WHEN** that player submits a return of that piece onto that cell
- **THEN** steps decrease by 1
- **AND** the piece is unfinished at that cell in synced state
- **AND** every client shows that piece on the board

#### Scenario [SC-MOVE-58]: Ring includes diagonal cells around the center block

- **GIVEN** it is a seated player’s turn eligible to return and a corner-adjacent playable cell just outside the central 2×2 is free and not a removed hole
- **WHEN** that player chooses that cell as the return target
- **THEN** the server accepts the return onto that cell

#### Scenario [SC-MOVE-59]: No return affordance without steps

- **GIVEN** it is a seated player’s turn with steps 0 and at least one finished own piece
- **WHEN** the client renders return controls
- **THEN** return is not offered as an actionable control
- **AND** a return message is rejected if sent

### Requirement: All four own pieces trapped resets to starts

When a seated player’s four pieces are all trapped (none finished), the server SHALL immediately clear trapped on those four pieces, clear the four holding grilles, and place each piece on a uniformly random **free** start cell of that piece’s own side (`N`/`E`/`S`/`W`). Other seats’ grilles MUST NOT be cleared by this reset. The affected client MUST see a warning modal (product Russian sense: tourists are sent to start) that does not block other clients; closing it is informational only. If that seat is still current turn with remaining steps and turn time, the seat MUST keep the turn after reset.

#### Scenario [SC-MOVE-60]: Fourth trap triggers immediate start reset

- **GIVEN** a seated player has three pieces already trapped and one free unfinished piece that lands on an unspent grille as a legal move
- **WHEN** that land traps the fourth piece
- **THEN** all four of that seat’s pieces are free on start cells of their respective sides
- **AND** the four grilles that held them are cleared
- **AND** other hidden or revealed grilles unrelated to those four remain unchanged

#### Scenario [SC-MOVE-61]: Same turn continues after all-jail reset

- **GIVEN** the same setup as SC-MOVE-60 and after the trapping move the seat still has steps ≥ 1 and turn time remaining
- **WHEN** the all-jail reset completes
- **THEN** that seat remains the current turn
- **AND** that seat MAY move a free piece with remaining steps

#### Scenario [SC-MOVE-63]: All-jail warning modal is only for the affected seat

- **GIVEN** an all-jail reset occurs for seat A while seat B and a spectator are connected
- **WHEN** clients update
- **THEN** only seat A’s client shows the start-warning modal
- **AND** seat B and the spectator do not see that modal

### Requirement: Auto end-turn waits for available rescue or return

While evaluating whether the current seat has no available actions for auto end-turn, the server MUST treat a legal own rescue and a legal own return (when finish place is 0) as available actions. Auto end-turn MUST NOT fire solely because no board move remains if rescue or return is still legal under steps and geometry rules.

#### Scenario [SC-MOVE-62]: Auto-end deferred while rescue is legal

- **GIVEN** it is a seated player’s turn with steps ≥ 1, no legal board move for any free piece, and a legal own rescue exists
- **WHEN** the server evaluates auto end-turn
- **THEN** the turn does not auto-end solely for lack of board moves
- **AND** the seat may still submit that rescue

### Requirement: Solo uses the same grille rules

When exactly one eligible seat remains (solo), grille land/trap, rescue, return, and all-jail reset MUST follow the same rules as multiplayer, subject to existing solo budget and timer rules (`game/move` main).

#### Scenario [SC-MOVE-64]: Solo rescue costs one finite step

- **GIVEN** a solo eligible seat with finite steps ≥ 1, one own trapped piece, and one own free piece at Chebyshev-1
- **WHEN** that seat submits a rescue
- **THEN** steps decrease by 1
- **AND** the trapped piece becomes free
