## Purpose

Delta этого change: land→trap, rescue, return-from-finish, all-jail reset, steps cost, auto-end. Базовые budgets/turn — main `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-51 | done (server mocha — land traps) |
| SC-MOVE-52 | done (server mocha — trapped cannot move) |
| SC-MOVE-53 | done (server mocha — trapped cannot peek) |
| SC-MOVE-54 | done (server mocha — rescue costs 1 step) |
| SC-MOVE-55 | done (server mocha — cannot rescue other seat) |
| SC-MOVE-56 | done (server mocha — rescue needs adjacency + steps) |
| SC-MOVE-57 | done (server mocha — return costs 1 step) |
| SC-MOVE-58 | done (server mocha — return ring cells) |
| SC-MOVE-59 | done (server mocha — return rejected without steps) |
| SC-MOVE-60 | done (server mocha — all-jail auto reset) |
| SC-MOVE-61 | done (server mocha — all-jail keeps turn) |
| SC-MOVE-62 | done (server mocha — auto-end waits for rescue/return) |
| SC-MOVE-63 | done (client UX — all-jail modal self only) |
| SC-MOVE-64 | done (server mocha — solo same rules) |

Related: grille visuals — `game/board`; trapped/sync placement — `game/pieces`; unfinished again — `game/finish`.

## ADDED Requirements

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

While it is a seated client’s own turn with steps ≥ 1, that client MAY rescue exactly one of their own trapped pieces when at least one of their own free unfinished pieces stands at Chebyshev distance 1 from the trapped piece’s cell (orthogonal or diagonal). A successful rescue MUST decrease steps by 1, clear trapped on the rescued piece, and clear the holding grille (rise and vanish ~1500 ms per `game/board`). Rescue of another seat’s piece MUST be rejected. Rescue MUST NOT permanently relocate the rescuer (approach animation is client presentation only).

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
