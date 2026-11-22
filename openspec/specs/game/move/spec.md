# game/move Specification

## Purpose

Очередь хода и перемещение фигурок туриста на одну соседнюю клетку: авторитетная валидация на сервере, локальные подсказки только ходящему, плавная анимация у всех клиентов. Вход на center завершает фишку (`game/finish`); finished-seats пропускаются в очереди хода. Пока `playing`, сервер ведёт deadline хода (60 с / соло 5 мин) и может сдвинуть ход или заблокировать time-expired без движения фигурок.

Связанные capability: фазы старта и gate ходов — `game/start`; рассадка и pieces — `game/pieces`; геометрия поля — `game/board`; reconnect grace и кольца — `game/presence`; финиш — `game/finish`; leave без confirm для time-expired — `game/leave`.

## Traceability

| Scenario ID | Coverage |
|-------------|---------|
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
| SC-MOVE-43 | covered (server mocha — move no longer advances turn) |
| SC-MOVE-44 | covered (server mocha) |
| SC-MOVE-46 | covered (client UX — keep-focus after move) |
| SC-MOVE-49 | covered (server mocha — move onto removed hole rejected) |
| SC-MOVE-51 | covered (server mocha — land traps) |
| SC-MOVE-52 | covered (server mocha — trapped cannot move) |
| SC-MOVE-53 | covered (server mocha — trapped cannot peek) |
| SC-MOVE-57 | covered (server mocha — return costs 1 step) |
| SC-MOVE-58 | covered (server mocha — return ring cells) |
| SC-MOVE-59 | covered (server mocha — return rejected without steps) |
| SC-MOVE-60 | covered (server mocha — all-jail auto reset) |
| SC-MOVE-61 | covered (server mocha — all-jail keeps turn) |
| SC-MOVE-63 | covered (client UX — all-jail modal self only) |
| SC-MOVE-64 | covered (server mocha — solo same rules) |
| SC-MOVE-65 | covered (client vitest GameBoardWire — per-cell finish click / outline) |
| SC-MOVE-77 | covered (client UX — 36×36 rescue; hit clear of tourist figure) |
| SC-MOVE-80 | covered (server mocha — move onto catapult flings to ring-2 when free) |
| SC-MOVE-81 | covered (server mocha — fling prefers ring-2 then ring-1) |
| SC-MOVE-82 | covered (server mocha — no dest → broken consume, piece stays) |
| SC-MOVE-83 | covered (server mocha — push onto catapult triggers same as move) |
| SC-MOVE-84 | covered (server mocha — return onto catapult triggers same as move) |
| SC-MOVE-85 | covered (server mocha — after fling, dest land effects apply) |
| SC-MOVE-86 | covered (server mocha — rescue then remaining catapult flings) |
| SC-MOVE-87 | covered (server mocha — stacked traps resolve in random order) |
| SC-MOVE-88 | covered (server mocha — catapult one-shot consumed) |
| SC-MOVE-89 | covered (server mocha — fling onto center finishes — see game/finish) |
| SC-MOVE-90 | covered (server mocha — paced: next hop only after presentation budget) |
| SC-MOVE-91 | covered (server mocha — auto-end deferred until trap pipeline idle) |
| SC-MOVE-92 | covered (server mocha — deadline advance deferred until trap pipeline idle; pending absorb if eligibility repair already moved current) |
| SC-MOVE-93 | covered (server mocha — idle re-eval auto-end after grille removes peek) |
| SC-MOVE-96 | covered (server mocha — finish place via catapult→center clears current turn; pending absorb) |
| SC-MOVE-33…50 | removed/superseded — steps/peeks budgets & solo ∞ |
| SC-MOVE-25…32 | modified — restore multi turn deadline 60s; no solo leftover timeExpired |
| SC-MOVE-37…41 | removed — end-turn / peeks gate |
| SC-MOVE-51…53 | modified — grille land sets remainingTurns; no steps/peek |
| SC-MOVE-54…56 | removed — rescue for steps; see SC-MOVE-113 free rescue |
| SC-MOVE-57…59 | modified — return spends the turn, no steps |
| SC-MOVE-60…61 | modified — all-jail no teleport; advance turn |
| SC-MOVE-124 | pending — all-jail skip whenever nobody can move + room toast (if eligible remain) |
| SC-MOVE-125 | pending — no all-jail toast when party over / solo-only wait |
| SC-MOVE-66…76 | removed — push |
| SC-MOVE-78…79 | removed — density/overlap seed; exclusive seed in game/board |
| SC-MOVE-85…86 | modified — exclusive cells; no dual-trap resolve |
| SC-MOVE-100…106 | removed — single-player steps/peeks refresh |
| SC-MOVE-110 | pending — one board action per turn |
| SC-MOVE-111 | pending — turn advances after modal/trap settle |
| SC-MOVE-112 | modified — turn deadline 60s restored |
| SC-MOVE-113 | pending — rescue free before move |
| SC-MOVE-114 | pending — push rejected |
| SC-MOVE-115 | pending — return spends turn |
| SC-MOVE-116 | pending — knowledge on piece |
| SC-MOVE-117 | pending — grille timer tick on become-current |
| SC-MOVE-118 | pending — grille auto-free at 1 then become-current |
| SC-MOVE-119 | modified — all-jail wait remainingTurns; advance turn |
| SC-MOVE-120 | pending — fling open-on-land chain |
| SC-MOVE-121 | pending — no solo leftover infinite |
| SC-MOVE-122 | pending — reopen open task by land |
| SC-MOVE-123 | pending — turn timeout closes open modal |

Related: board peek / removed tiles / grilles / catapults — `game/board`; presence countdown / knowledge strip — `game/presence`; finish / return / catapult→center / knowledge modal — `game/finish`; trapped sync / remainingTurns — `game/pieces`.

## Requirements

### Requirement: One board move or return per turn

While phase is `playing` and it is a non-finished seated player’s turn, that seat MAY perform unlimited **own** rescues of adjacent trapped own pieces, then MUST perform at most **one** board action that relocates a piece: a legal **move** or a legal **return-from-finish**. The server MUST reject a second move or return in the same turn. There is no end-turn message. There is no steps or peeks budget. Push intents MUST be rejected. After the client successfully **sends** an own `move` or `returnFromFinish` this turn (`sendMove` / `sendReturnFromFinish` returned true — optimistic mirror of server `boardActionUsed`; no public schema field and no per-move ack), it MUST locally disable further move/return/rescue send affordances until the synchronized current seat is no longer that player. Shared-modal `peekPlace` / `peekSubmit` remain allowed while the land-opened task modal is open. The client MAY clear the local spent flag early if, after board-pipeline settle, the piece never left its origin and no land modal opened (silent reject recovery).

#### Scenario [SC-MOVE-110]: Second move in same turn rejected

- **GIVEN** it is a seated player’s turn and that seat has already completed one successful move this turn whose open-on-land pipeline has settled without ending the turn yet solely due to an open task modal (client local board-action spent is set from the successful send)
- **WHEN** that player attempts another move before the turn advances
- **THEN** the server rejects the second move
- **AND** piece positions from the first move remain
- **AND** the acting client’s move/return/rescue affordances stay disabled for the rest of that turn (peekPlace/peekSubmit for the open modal still allowed)

#### Scenario [SC-MOVE-114]: Push is rejected

- **GIVEN** it is a seated player’s turn with an own free piece adjacent to another free unfinished piece
- **WHEN** that player submits a push intent
- **THEN** the server rejects the push
- **AND** piece positions are unchanged

### Requirement: Turn advances after the board action resolves

After the seat’s one move or return is accepted, the server MUST run open-on-land / trap pipeline for the landing (and fling chain). When the landing opens a **task** modal, the current turn MUST remain until that modal closes (correct, wrong, turn-timeout, or leave force-close). When the landing is a **grille**, after trap presentation the turn MUST advance to the next eligible seat. When the landing is a **catapult**, the fling chain continues under paced presentation; each hop applies open-on-land; when the chain settles (empty/start/finish/grille-after-trap), turn advance follows the same rules as a direct land of that final content (task waits for modal; grille advances after trap; finish follows `game/finish`). When the landing needs no modal (empty/start/already-empty), the turn MUST advance after presentation settles.

While phase is `playing` and at least one other eligible seat exists (or the sole eligible seat is current under one-move economy), the server MUST run a synchronized **turn deadline of 60 seconds** (`turnBudgetSeconds` / `turnUntil`) for the current seat. The deadline MUST keep ticking while a task modal is open. On deadline expiry with an open modal, the server MUST force-close with incorrect + `reason: 'timeout'`, KEEP the bind/open digit, show clients the «Не успел!» toast path (`game/presence`), then advance the turn. On deadline expiry without an open modal, the server MUST advance the turn without permanently marking the seat `timeExpired` for leftover-solo rules. Permanent leftover `timeExpired` / one-minute solo leftover clocks MUST NOT return.

#### Scenario [SC-MOVE-111]: Task modal close advances turn in multiplayer

- **GIVEN** a tourist room with two eligible seats A then B, it is A’s turn, and A’s move opens a task modal
- **WHEN** A closes the modal (correct or incorrect)
- **THEN** the current turn becomes B

#### Scenario [SC-MOVE-112]: Turn deadline is sixty seconds

- **GIVEN** a tourist room in phase `playing` with two or more eligible seats
- **WHEN** a seat becomes current turn
- **THEN** synchronized turn deadline fields are active with a 60-second budget
- **AND** presence clients MAY show a turn countdown ring (`game/presence`)

#### Scenario [SC-MOVE-123]: Turn timeout closes open task modal

- **GIVEN** it is seat A’s turn with an open task modal and the 60-second deadline elapses
- **WHEN** the server applies turn timeout
- **THEN** the modal closes as incorrect with timeout reason
- **AND** the task bind remains KEEP with open knowledge digit
- **AND** the current turn advances to the next eligible seat
- **AND** clients show the «Не успел!» top-center toast

#### Scenario [SC-MOVE-120]: Catapult land opens destination content

- **GIVEN** a legal move lands on a catapult whose fling destination is an unbound Q&A task cell
- **WHEN** the fling relocate completes
- **THEN** that destination opens as open-on-land (bind + shared task modal)
- **AND** the turn waits for modal close (or timeout) before advancing

### Requirement: Free own rescue until the board action

While it is the seat’s turn and the seat has not yet spent its one move/return, the seat MAY rescue an own trapped unfinished piece when an own free unfinished piece stands at Chebyshev distance 1. Rescue MUST NOT cost knowledge or a board-action slot, MUST clear trapped and the holding grille (empty cell afterward per `game/board`), and MUST NOT permanently relocate the rescuer. Rescue of another seat’s piece MUST be rejected. After the seat has completed its move or return for this turn, further rescues this turn MUST be rejected.

#### Scenario [SC-MOVE-113]: Rescue before move does not end the turn

- **GIVEN** it is a seated player’s turn, one own piece is trapped, and another own free piece is adjacent
- **WHEN** that player successfully rescues the trapped piece
- **THEN** the rescued piece is free
- **AND** the current turn remains that seat
- **AND** that seat may still perform one move or return

### Requirement: Knowledge points on pieces

Each unfinished piece SHALL carry a non-negative **knowledge** integer. The server MUST sync the owning seat’s carried amounts and delivered total via a private room message **`knowledge`** with payload `{ delivered: number, pieces: Record<pieceId, number>, outcome?: 'finish' | 'cut' }` (same privacy family as former `budgets`; other seats MUST NOT receive another seat’s knowledge). Correct task solve MUST add the task **difficulty** (1|2|3) to that piece’s knowledge and re-send. Knowledge MUST NOT increase on wrong answers. When a piece finishes (`game/finish`), its knowledge MUST transfer to the seat’s delivered knowledge total and the piece’s carried knowledge becomes 0, then re-send. On full finish / penultimate cut the server MUST set `outcome` accordingly for the modal copy (`game/finish`). Private steps/peeks `budgets` messages MUST NOT be used for turn economy.

#### Scenario [SC-MOVE-116]: Correct answer adds difficulty knowledge to the piece

- **GIVEN** an own free piece with knowledge 0 opens a task of difficulty 3 and submits the correct order
- **WHEN** the server accepts the answer
- **THEN** that piece’s knowledge is 3
- **AND** the seat’s delivered knowledge is unchanged until that piece finishes

#### Scenario [SC-MOVE-122]: Another seat may solve an open task by landing

- **GIVEN** task cell C is open with bound difficulty 2 after a wrong answer by seat A, A’s piece has since left C so C is unoccupied, and it is seat B’s turn
- **WHEN** seat B’s free piece legally lands on C and submits the correct order
- **THEN** the shared task modal opens with B as the only seat that may edit slots
- **AND** B’s piece knowledge increases by 2
- **AND** cell C becomes an empty playable cell

### Requirement: Grille remaining turns tick on become-current

When a free unfinished piece becomes trapped by a grille, its public schema `remainingTurns` MUST start at **3**. Each time that piece’s seat **becomes** current turn while the piece is still trapped with remaining turns ≥ 1, the server MUST decrement remaining turns by 1. When the seat becomes current and remaining turns were **1**, the server MUST clear the grille (empty cell), free the piece, set remaining turns to 0, and MUST allow that piece to move during that same turn. Auto-free and rescue MUST clear the badge (`remainingTurns = 0`). Other seats’ become-current MUST NOT tick another seat’s trapped counters.

#### Scenario [SC-MOVE-117]: Become-current decrements grille counter

- **GIVEN** seat A has a trapped piece with remaining turns 3 and it is not A’s turn
- **WHEN** the turn advances so A becomes current
- **THEN** that piece’s remaining turns become 2
- **AND** the piece remains trapped

#### Scenario [SC-MOVE-118]: Become-current from 1 frees and allows move

- **GIVEN** seat A has a trapped piece with remaining turns 1
- **WHEN** A becomes current turn
- **THEN** the grille is cleared and the piece is free on the empty cell
- **AND** A may legally move that piece during this turn

### Requirement: No solo leftover infinite economy

When `maxSeats ≥ 2` and exactly one eligible seat remains, the server MUST NOT grant infinite steps/peeks, MUST NOT start a one-minute solo deadline, and MUST NOT time-expire that seat for solo timer rules. That seat continues under the one-move economy until it finishes or is cut by penultimate completion (`game/finish`). Single-player (`maxSeats === 1`) uses the same one-move economy without penultimate cut.

#### Scenario [SC-MOVE-121]: Last eligible seat is not infinite solo

- **GIVEN** a tourist room with maxSeats 2 in phase `playing` where one seat has finished and one eligible seat remains
- **WHEN** that remaining seat becomes current turn
- **THEN** that seat does not receive infinite budgets
- **AND** there is no one-minute solo deadline
- **AND** the seat may take one move or return per turn as in multiplayer

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

### Requirement: Authoritative one-step tourist move

The server SHALL accept a move message only when the room start phase is `playing`, and only from a non-finished seated client who is not time-expired, whose turn it is, that identifies one of that client’s own unfinished pieces and a target cell, and who has steps ≥ 1 in finite steps mode (or infinite steps in solo). A legal move MUST place that piece exactly one cell away in row and/or column (orthogonal or diagonal: Chebyshev distance 1), onto a playable tourist layout cell (start, still-present task cell, **removed** task cell that was formerly a task tile, or center cell), and MUST NOT land on a cell occupied by any unfinished piece in the room (including the mover’s other unfinished pieces). Finished pieces MUST NOT occupy cells and MUST NOT be move targets as pieces. Layout holes that were **never** playable on the map snapshot and cells outside the playable layout MUST be rejected as landing targets. Removed task cells MUST be accepted as landing targets when otherwise legal. Moves from a client that is not seated, is finished, is time-expired, not the current turn, not in phase `playing`, that target another player’s piece, that target an already finished own piece, or that lack a step in finite mode MUST be rejected without changing piece positions or the current turn. On acceptance onto a non-center playable cell the server MUST update the piece’s synchronized row and column, decrement steps by 1 only in finite mode, and **MUST NOT** advance the turn. On acceptance onto a center cell the server MUST finish the piece per `game/finish` (piece leaves board occupancy), decrement steps by 1 only in finite mode, and **MUST NOT** advance the turn solely because of that move. A piece already standing on a removed task cell MAY leave that cell onto a legal neighbor. Player-initiated end-turn and auto end-turn / timeout rules of this capability advance the turn without requiring a move.

#### Scenario [SC-MOVE-04]: Legal orthogonal step updates position without advancing turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1 (or infinite), and one of that player’s unfinished pieces has a free orthogonal neighbor playable non-center cell
- **WHEN** that player submits a move for that piece to that neighbor cell
- **THEN** the server updates that piece’s synchronized row and column to the target
- **AND** the turn does not advance solely because of that move
- **AND** every client observes the new piece position

#### Scenario [SC-MOVE-05]: Diagonal step is allowed

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1 (or infinite), and one of that player’s unfinished pieces has a free diagonal neighbor playable cell
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
- **WHEN** that player submits a move onto a layout hole that was never a playable map cell or otherwise non-playable cell adjacent in grid coordinates
- **THEN** the server rejects the move
- **AND** piece positions and the current turn remain unchanged

#### Scenario [SC-MOVE-49]: Landing on a removed task hole is rejected

- **GIVEN** the room phase is `playing`, it is a seated player’s turn with steps ≥ 1 (or infinite), and a neighbor cell is a removed task hole that was formerly a task tile
- **WHEN** that player submits a move onto that removed cell
- **THEN** the server accepts the move
- **AND** the piece’s synchronized position updates to that cell
- **AND** the turn does not advance solely because of that move

#### Scenario [SC-MOVE-08]: Start and center cells are walkable

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with steps ≥ 1 (or infinite), and an unfinished piece has a free neighbor that is a start cell or a center cell of the tourist layout
- **WHEN** that player submits a move onto that cell
- **THEN** the server accepts the move
- **AND** if the target is a start cell, the piece’s synchronized position updates to that cell
- **AND** if the target is a center cell, the piece is finished per `game/finish` and leaves board occupancy
- **AND** the turn does not advance solely because of that move

#### Scenario [SC-MOVE-09]: Move out of turn is rejected

- **GIVEN** the room phase is `playing` and it is not that seated player’s turn
- **WHEN** that player submits a move
- **THEN** the server rejects the move without changing piece positions

#### Scenario [SC-MOVE-10]: Spectator cannot move

- **GIVEN** a spectator client in a `playing` tourist room
- **WHEN** that client submits a move
- **THEN** the server rejects the move

#### Scenario [SC-MOVE-18]: Move before playing is rejected

- **GIVEN** the room phase is `waiting` or `countdown`
- **WHEN** a seated player submits a move
- **THEN** the server rejects the move

#### Scenario [SC-MOVE-23]: Finished seat cannot move

- **GIVEN** a seated player with a finish place
- **WHEN** that player submits a move
- **THEN** the server rejects the move

#### Scenario [SC-MOVE-44]: Center finish spends a step without rotating turn

- **GIVEN** the room phase is `playing`, it is a multiplayer turn with finite steps ≥ 1, and an unfinished piece has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** the piece is finished per `game/finish`
- **AND** steps decrease by 1 in finite mode
- **AND** the turn does not advance solely because of that move

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

### Requirement: Finish-cell click submits that cell only

Finish cells on the Game board MUST behave like other playable tiles for local move selection chrome and move submission (they are **not** one visual 2×2 block). While the client is submitting a move (not return-mode) and a finish cell F is a legal one-step destination for the selected piece, activating **that** cell MUST submit a move to F. Activating a different finish cell that is not legal MUST NOT submit a move to F via nearest-center remapping. Red destination outlines MUST appear only on finish cells that are legal for the selected piece. White selection outline MUST follow the selected piece’s cell only. Room message `move` payload `{ pieceId, row, col }` and server validation of that destination cell remain unchanged (existing per-cell `7` playable centers); this requirement is client chrome only.

#### Scenario [SC-MOVE-65]: Finish click targets the activated cell

- **GIVEN** it is the user’s turn, an own unfinished piece is selected, and exactly one of several finish cells is a legal one-step destination for that piece
- **WHEN** the user activates that legal finish cell
- **THEN** the client submits a move to that same cell
- **AND** only that legal finish cell shows the red destination outline among finish cells
- **AND WHEN** the user activates a different finish cell that is not a legal destination
- **THEN** the client MUST NOT submit a move to the legal finish cell via nearest-center remapping

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

When phase is `playing` and the server accepts a legal move/return/fling whose target cell still holds an exclusive unspent grille, the server SHALL reveal and clear the hidden status of that grille, mark that piece trapped, set public `Piece.remainingTurns` to **3**, and MUST NOT spend steps/peeks. The piece MUST remain on that cell and MUST continue to occupy it for other pieces. The grille MUST become visible to all clients per `game/board`. After grille presentation settles (and unless all-jail fires), the turn MUST advance per the turn-advance requirement.

#### Scenario [SC-MOVE-51]: Land on grille traps and spends the step

- **GIVEN** it is a seated player’s turn and an unfinished free piece adjacent to an exclusive grille cell
- **WHEN** that player submits a legal move onto that cell
- **THEN** that piece is marked trapped with `remainingTurns` 3
- **AND** the piece remains on that cell
- **AND** no steps budget is spent

### Requirement: Trapped pieces cannot move or peek

While a piece is trapped, the server MUST reject move intents that target that piece and MUST reject client `peek` intents. Other free unfinished pieces of the same seat MAY still move under the one-board-action turn rules when it is that seat’s turn and the board action is not yet spent. Open-on-land task opens remain land-driven only.

#### Scenario [SC-MOVE-52]: Move rejected for trapped piece

- **GIVEN** it is a seated player’s turn and one of that player’s pieces is trapped
- **WHEN** that player submits a move identifying the trapped piece
- **THEN** the server rejects the move
- **AND** piece positions do not change from that attempt

#### Scenario [SC-MOVE-53]: Peek rejected for trapped piece

- **GIVEN** it is a seated player’s turn and a trapped own piece stands on a cell
- **WHEN** that player attempts a client `peek` intent for that trapped piece
- **THEN** the server rejects the peek
- **AND** no task modal opens from that intent

### Requirement: Return a finished piece onto the center ring for one step

While it is a seated client’s own turn, that seat has finish place **0**, has not yet spent its one board action this turn, and at least one own piece is finished, the client MAY return one finished piece onto a legal ring cell: any playable cell at Chebyshev distance 1 from the central 2×2 block (including diagonal corners), including a **removed** task cell on that ring, that is not occupied by an unfinished piece. Layout holes that were never playable MUST NOT be return targets. A successful return MUST count as the seat’s one board action (MUST NOT cost steps), mark that piece unfinished at the chosen cell, apply open-on-land on the landing cell, and advance the turn after land/trap settle per the turn-advance requirement.

#### Scenario [SC-MOVE-57]: Return spends one step and places on ring

- **GIVEN** it is a seated player’s turn with finish place 0, one finished own piece, a free playable ring cell, and the seat has not spent its board action
- **WHEN** that player submits a return of that piece onto that cell with no task/trap content
- **THEN** the return spends the seat’s one board action (no steps budget)
- **AND** the piece is unfinished at that cell in synced state
- **AND** after presentation settles the turn advances to the next eligible seat

#### Scenario [SC-MOVE-58]: Ring includes diagonal cells around the center block

- **GIVEN** it is a seated player’s turn eligible to return and a corner-adjacent playable cell just outside the central 2×2 is free (including a removed task cell)
- **WHEN** that player chooses that cell as the return target
- **THEN** the server accepts the return onto that cell

#### Scenario [SC-MOVE-59]: No return affordance without steps

- **GIVEN** it is a seated player’s turn and that seat has already spent its one move/return board action this turn and at least one finished own piece remains
- **WHEN** the client renders return controls
- **THEN** return is not offered as an actionable control
- **AND** a return message is rejected if sent

### Requirement: All four own pieces trapped resets to starts

When a seated player’s unfinished pieces are all trapped (none free unfinished remaining; count equals that seat’s unfinished pieces, not always four), the server MUST **NOT** teleport those pieces to start cells, MUST **NOT** clear their holding grilles solely because all are trapped, and MUST **NOT** show the retired start-reset all-jail warning modal. Pieces MUST remain trapped with their public `remainingTurns` and continue the grille tick/auto-free rules on become-current.

Whenever that seat cannot perform a board action because all unfinished pieces are trapped — including (a) after the trapping land’s grille presentation settles and (b) after become-current grille ticks if the seat is still fully trapped — **and** at least one **other** eligible seat remains (`countEligible() ≥ 2`), the server MUST **advance** the current turn to the next eligible seat and MUST broadcast a room-wide skip signal so **every** connected client shows a top-center Notify toast with product Russian sense «Все туристы в решётке — ход пропущен». The server MUST **NOT** keep the turn for further board actions while all pieces stay trapped when another eligible seat exists.

When fewer than two eligible seats remain (solo waiting on ticks, or the party has ended / only finished seats remain), the server MUST **NOT** fire the all-jail skip toast. End-of-game outcome uses the knowledge modal (`game/finish`) only — no all-jail toast. If become-current ticks free at least one unfinished piece, the seat keeps the turn and MUST NOT toast.

#### Scenario [SC-MOVE-60]: Fourth trap triggers immediate start reset

- **GIVEN** a seated player has three pieces already trapped and one free unfinished piece that lands on an exclusive grille as a legal move
- **WHEN** that land traps the last free unfinished piece
- **THEN** all of that seat’s unfinished pieces remain trapped on their grille cells (no start teleport)
- **AND** their `remainingTurns` stay positive
- **AND** other seats’ grilles are unchanged

#### Scenario [SC-MOVE-61]: Same turn continues after all-jail reset

- **GIVEN** the same setup as SC-MOVE-60 with two eligible opponents remaining
- **WHEN** the all-jail land presentation settles
- **THEN** that seat does **not** remain the current turn
- **AND** the current turn advances to the next eligible seat
- **AND** that seat’s pieces are still trapped awaiting remainingTurns ticks

#### Scenario [SC-MOVE-63]: All-jail warning modal is only for the affected seat

- **GIVEN** seat A becomes fully trapped while seat B and a spectator are connected
- **WHEN** clients update
- **THEN** no client shows the retired start-reset all-jail warning modal
- **AND** seat B and the spectator likewise do not see that modal

#### Scenario [SC-MOVE-119]: All-jail waits and advances turn

- **GIVEN** seat A has two eligible opponents remaining and A’s last free unfinished piece lands on a grille so all of A’s unfinished pieces are trapped
- **WHEN** the trap presentation settles
- **THEN** A’s pieces remain trapped with remainingTurns
- **AND** the current turn is no longer A

#### Scenario [SC-MOVE-124]: Become-current still all-jailed skips with room toast

- **GIVEN** seat A is fully trapped with remainingTurns > 1 on every unfinished piece, and at least one other eligible seat exists
- **WHEN** A becomes current and grille ticks leave A still fully trapped
- **THEN** the current turn advances away from A
- **AND** every connected client shows the all-jail skip toast
- **AND** A’s pieces remain trapped (no teleport)

#### Scenario [SC-MOVE-125]: No all-jail toast when the party is over

- **GIVEN** trapping or become-current would leave fewer than two eligible seats (party ended / cut / solo-only wait)
- **WHEN** the all-jail condition is evaluated
- **THEN** the all-jail skip toast is not shown
- **AND** end-of-game seats use the knowledge modal path instead of an all-jail toast

### Requirement: Rescue affordance centered above trapped tourist

While it is the user’s own turn with steps ≥ 1 and a legal own rescue exists for a trapped unfinished piece, the client MUST show the rescue affordance **centered above** that trapped tourist on the board (same top-center family as push / peek / strip return). The affordance MUST use the **illustrated board-action rescue icon** and MUST NOT paint a solid colored (including green) circular chrome behind the icon. The illustrated glyph MUST render at **36×36 CSS pixels** for both the illustrated glyph and the hit target (glyph fills the hit). The rescue hit MUST sit entirely **above** that trapped tourist figure (same clearance as peek / push).

#### Scenario [SC-MOVE-77]: Rescue icon centered above trapped

- **GIVEN** it is the user’s turn with steps ≥ 1 and a legal own rescue exists
- **WHEN** the board renders affordances
- **THEN** a rescue affordance with the illustrated rescue icon is shown centered above that trapped tourist
- **AND** the affordance uses the 36×36 glyph-fills-hit family
- **AND** the affordance hit does not cover that tourist figure
- **AND** the affordance has no solid colored circular chrome behind the icon

### Requirement: Solo uses the same grille rules

When exactly one eligible seat remains (`maxSeats ≥ 2` leftover or `maxSeats === 1`), grille land/trap, rescue, return, remainingTurns tick, and all-jail reset MUST follow the same one-move economy rules as multiplayer. Solo leftover infinite budgets and one-minute timer MUST NOT apply.

#### Scenario [SC-MOVE-64]: Solo rescue costs one finite step

- **GIVEN** a leftover eligible seat (`maxSeats ≥ 2`, exactly one eligible) with one own trapped piece and one own free piece at Chebyshev-1, before the board action is spent
- **WHEN** that seat submits a rescue
- **THEN** the rescue succeeds without spending a board-action slot or steps budget
- **AND** the trapped piece becomes free on an empty playable cell

### Requirement: Landing resolves remaining traps on the cell

When phase is `playing` and a piece **lands** on a cell via an accepted move or return-from-finish (push removed), the server SHALL resolve that cell’s exclusive content under open-on-land: at most one of Q&A task, grille, or catapult (never grille+catapult together). Landing after a catapult fling MUST use the same open-on-land rules on the destination cell. The server MUST NOT apply an entire multi-hop catapult chain in a single synchronous tick — paced presentation budgets remain. Because cells are exclusive, there is no remaining second trap on the same cell after grille holding.

#### Scenario [SC-MOVE-87]: Stacked traps shuffle order each resolve

- **GIVEN** exclusive trap seed so no cell holds both an unspent grille and an unspent catapult
- **WHEN** a free unfinished piece lands on a trap cell
- **THEN** only that cell’s single trap type resolves (grille **or** catapult)
- **AND** the server does not shuffle two trap types on one cell

#### Scenario [SC-MOVE-86]: Rescue then remaining catapult flings

- **GIVEN** a piece is trapped on an exclusive grille cell (no catapult on that cell)
- **WHEN** that piece is successfully rescued and becomes free on that same cell
- **THEN** the cell is an empty playable cell
- **AND** no deferred catapult resolves on that former grille cell

#### Scenario [SC-MOVE-90]: Paced resolve applies only the next hop after budget

- **GIVEN** a free unfinished piece lands on a cell with an unspent catapult that will fling onto another cell that still holds an exclusive grille
- **WHEN** the catapult hop is revealed
- **THEN** synced piece coordinates MUST remain on the catapult cell until that hop’s presentation budget elapsed
- **AND** the destination grille MUST NOT appear in `holdingGrilleKeys` until after the fling relocation is applied and that destination land hop begins
- **AND** the server MUST NOT pre-compute and sync the final trapped position in the same tick as the initial land

### Requirement: Catapult flings to a free landable ring cell

When a catapult resolves for a free unfinished piece on its cell, the server SHALL consume that catapult (one-shot) and MUST attempt to relocate the piece as follows: pick uniformly at random among **free legal landing** cells at Chebyshev distance **2** from the catapult cell; if that set is empty, pick uniformly among free legal landing cells at Chebyshev distance **1**. A legal landing cell MUST be playable on the tourist layout (including **removed** task cells that were formerly task tiles), MUST NOT be a never-playable layout hole, MUST NOT be outside the board, and MUST NOT be occupied by any unfinished piece (including trapped). The piece’s own former cell MUST NOT count as occupied against itself once the fling relocates it. Destination selection MUST use occupancy and playability **at fire time** (when the hop’s relocate is applied after the presentation budget). If both rings are empty, the catapult MUST still be consumed, the piece MUST remain on the catapult cell, and clients MUST present the broken vanish per `game/board`. A successful fling MUST NOT spend an extra step beyond the step already spent for the triggering move/push/return in finite mode (rescue-triggered fling spends no additional step beyond the rescue). After a successful fling relocation, destination side-effects MUST follow ordinary landing rules (including further traps and center finish), paced as above.

#### Scenario [SC-MOVE-80]: Move onto catapult flings when ring-2 is free

- **GIVEN** it is a seated player’s turn with steps ≥ 1 (or infinite) and a legal move lands a free unfinished piece on a cell with an unspent catapult and at least one free legal landing cell at Chebyshev-2 (including a removed task cell)
- **WHEN** that move is accepted and the catapult hop completes (reveal + presentation budget + relocate)
- **THEN** the piece occupies one uniformly chosen free Chebyshev-2 legal landing cell
- **AND** the catapult on the origin cell is spent
- **AND** in finite mode steps decreased by 1 for the move only

#### Scenario [SC-MOVE-81]: Fling falls back to ring-1 when ring-2 empty

- **GIVEN** a catapult resolves and no free legal landing cell exists at Chebyshev-2, but at least one exists at Chebyshev-1
- **WHEN** the destination is chosen at fire time
- **THEN** the piece is relocated to a uniformly chosen free Chebyshev-1 legal landing cell

#### Scenario [SC-MOVE-82]: No destination leaves piece and spends broken catapult

- **GIVEN** a catapult resolves and no free legal landing cell exists at Chebyshev-2 or Chebyshev-1
- **WHEN** the catapult is consumed
- **THEN** the piece remains on the catapult cell
- **AND** the catapult is spent
- **AND** clients present broken vanish per `game/board`

#### Scenario [SC-MOVE-83]: Push onto catapult triggers the same resolve

- **GIVEN** a successful push relocates a free unfinished target onto a cell with an unspent catapult
- **WHEN** that landing is resolved
- **THEN** the catapult resolves for the target as on an ordinary move landing (paced)

#### Scenario [SC-MOVE-84]: Return onto catapult triggers the same resolve

- **GIVEN** a successful return-from-finish places a piece onto a cell with an unspent catapult
- **WHEN** that landing is resolved
- **THEN** the catapult resolves for that piece as on an ordinary move landing (paced)

#### Scenario [SC-MOVE-85]: After fling, destination land effects apply

- **GIVEN** a fling relocates a piece onto a cell with further land side-effects
- **WHEN** the fling hop completes
- **THEN** those destination land effects apply per ordinary landing rules

#### Scenario [SC-MOVE-88]: Catapult is one-shot

- **GIVEN** a catapult has already been consumed on a cell
- **WHEN** another piece later lands on that cell
- **THEN** no second catapult fling is triggered from that spent catapult

#### Scenario [SC-MOVE-89]: Fling onto center finishes the piece

- **GIVEN** a catapult fling destination is a free center cell
- **WHEN** the fling relocates the piece there
- **THEN** the piece is finished per `game/finish`

### Requirement: Turn does not advance during trap presentation pipeline

While a trap presentation pipeline is active for the room (paced catapult/grille hops after a land), the server MUST NOT call `advanceTurn` for multiplayer auto-end-turn or for a turn-deadline expiry that fired during that pipeline. If auto-end-turn would otherwise apply, or the turn deadline fires while the pipeline is active, the server MUST record that a turn advance is pending and MUST perform that advance only after the pipeline becomes idle — **except** when idle eligibility repair (`ensureCurrentEligibleOrAdvance` / equivalent) has already moved `currentTurnSessionId` off the seat that held the turn at the start of idle (e.g. finish place assigned mid-pipeline): then the pending flag MUST be cleared **without** a second `advanceTurn` (pending absorb; at most one logical turn leave). When the pipeline becomes idle **without** a pending deadline/auto-end flag already set, the server MUST **re-evaluate** auto-end-turn (and solo steps-exhausted) against the seat’s available actions **after** the last hop’s effects — so a grille (or other trap) that removed the only legal peek or move can still advance the turn. The server MUST NOT require a client presentation-ack message to release the turn. Step/peek economy MUST remain unchanged. Manual `endTurn` while the pipeline is active MUST NOT skip pending presentation (reject or defer consistently with board non-interactive rules).

#### Scenario [SC-MOVE-91]: Auto-end waits for trap pipeline idle

- **GIVEN** multiplayer playing and a seated player’s last available action is a move that starts a catapult presentation pipeline
- **WHEN** after accepting that move the seat would otherwise auto-end-turn
- **THEN** `currentTurnSessionId` MUST remain that seat until the trap pipeline is idle
- **AND** only then MAY the server advance to the next eligible seat

#### Scenario [SC-MOVE-92]: Deadline during pipeline advances only after idle

- **GIVEN** a turn deadline fires while a trap presentation pipeline is still active for that turn’s seat
- **WHEN** the deadline handler runs
- **THEN** the server MUST NOT advance the turn immediately
- **AND** after the pipeline becomes idle the server MUST leave the turn off that seat via a single logical advance (pending `advanceTurn`, or pending absorb if eligibility repair already moved `currentTurnSessionId`)
- **AND** clients MUST still be able to complete the in-flight hop presentations before the turn chrome switches

#### Scenario [SC-MOVE-93]: Idle re-eval after grille removes the only peek

- **GIVEN** multiplayer playing, it is a seat’s turn with steps 0, peeks ≥ 1, and exactly one own unfinished piece on a still-present task cell (so a legal peek exists), and the other own pieces are not on live task cells
- **WHEN** that piece lands on the task cell and a grille traps it as part of the trap pipeline
- **THEN** after the pipeline becomes idle the server MUST re-evaluate available actions
- **AND** because the trapped piece can no longer peek and no other legal peek/move/rescue/return/push remains, the server MUST auto-advance the turn to the next eligible seat
- **AND** a prior `pendingTurnAdvance` from deadline MUST still force advance on idle without being skipped by this re-eval path when eligibility repair did not already change the current seat

### Requirement: Finish place assigned in trap pipeline clears current turn

When the trap presentation pipeline becomes idle and the seat that currently holds the turn has a synchronized finish place (`game/finish`) while at least one other eligible (non-finished, non-time-expired) seated player remains, the server SHALL advance the synchronized current turn to the next eligible seat in join order (same skip rules as finished-seat rotation). This MUST apply when the finish place was assigned during that pipeline — including a catapult fling onto a center cell that finishes the seat’s last piece — whether or not a pending auto-end or deadline advance flag was already set for that pipeline. The idle path MUST release the finished seat **at most once**: if eligibility advance already moved the current turn off that seat, a pending auto-end/deadline flag MUST be cleared **without** a second `advanceTurn` (pending absorb). Idle auto-end re-evaluation alone MUST NOT leave a finished seat as current turn when another eligible seat exists. While the pipeline is still active, the turn MAY remain on that seat until idle (unchanged paced rules). Solo (exactly one eligible seat) is unchanged: no rotation to another player.

#### Scenario [SC-MOVE-96]: Catapult full-finish releases the turn after pipeline idle

- **GIVEN** a tourist room in phase `playing` with at least two eligible seated players, it is seat A’s turn, A has exactly three finished pieces and one unfinished free piece with `finishPlace` still `0`, and that piece’s land starts a catapult presentation whose fling destination is a free center cell
- **AND** after accepting the land move A still has remaining available actions (so no pending auto-end is required solely from that move)
- **WHEN** the trap pipeline becomes idle after the paced catapult relocate finishes A’s fourth piece and assigns A a finish place
- **THEN** A’s synchronized finish place is greater than `0`
- **AND** the synchronized current turn is no longer A
- **AND** the current turn is the next eligible seat in join order (A is skipped)
