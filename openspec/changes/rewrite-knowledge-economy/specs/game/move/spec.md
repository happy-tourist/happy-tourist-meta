# Delta: game/move

Перепись экономики хода: один move|return, knowledge, open-on-land, grille timer, таймер хода 60s, all-jail без teleport, без push/steps/peeks/solo leftover/end-turn.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
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

## ADDED Requirements

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

## MODIFIED Requirements

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

### Requirement: Solo uses the same grille rules

When exactly one eligible seat remains (`maxSeats ≥ 2` leftover or `maxSeats === 1`), grille land/trap, rescue, return, remainingTurns tick, and all-jail reset MUST follow the same one-move economy rules as multiplayer. Solo leftover infinite budgets and one-minute timer MUST NOT apply.

#### Scenario [SC-MOVE-64]: Solo rescue costs one finite step

- **GIVEN** a leftover eligible seat (`maxSeats ≥ 2`, exactly one eligible) with one own trapped piece and one own free piece at Chebyshev-1, before the board action is spent
- **WHEN** that seat submits a rescue
- **THEN** the rescue succeeds without spending a board-action slot or steps budget
- **AND** the trapped piece becomes free on an empty playable cell

## REMOVED Requirements

### Requirement: Private step and peek budgets per seat

**Reason**: Replaced by one-move turns and knowledge points.
**Migration**: No `budgets` steps/peeks; use piece/seat knowledge sync.

### Requirement: Single-player room budgets without turn timer

**Reason**: Single-player uses the shared one-move economy; old +1/+1 refresh removed.
**Migration**: `maxSeats === 1` follows one move/return per turn without steps/peeks.

### Requirement: Single-player refresh when no actions remain

**Reason**: No steps/peeks refresh loop.
**Migration**: Player takes move/return; rescue free before move; all-jail if all trapped.

### Requirement: Authoritative turn deadline while playing

**Reason**: Turn timer removed entirely.
**Migration**: No `turnUntil` / time-expired from turn clock.

### Requirement: Turn timeout advances among eligible seats

**Reason**: No turn deadline to expire.
**Migration**: Turn advances only after board-action resolve / modal close.

### Requirement: Turn timeout forces open peek as incorrect

**Reason**: No turn-timeout force-close path.
**Migration**: Task modal closes via submit/wrong/leave only; see `game/board` peek-close.

### Requirement: Solo infinite peeks and finite steps

**Reason**: Solo leftover infinite economy removed.
**Migration**: Last eligible seat continues one-move play until finish or penultimate cut.

### Requirement: Solo endgame five-minute budget

**Reason**: One-minute solo leftover deadline removed (title historic; duration was 60s).
**Migration**: No solo timer; penultimate cut / full finish end the run.

### Requirement: Solo budget expiry locks moves

**Reason**: No solo timer expiry / time-expired lock from leftover mode.
**Migration**: Cut seat uses `finishPlace = 1` + knowledge modal (`game/finish`).

### Requirement: Player end-turn and auto end-turn when no actions remain

**Reason**: Turn advances after the single board action resolves; no end-turn control.
**Migration**: Remove end-turn message and auto-end based on peeks/steps.

### Requirement: Multiple peeks per turn while peeks remain

**Reason**: Peeks budget and client `peek` open removed; open-on-land only.
**Migration**: Land to open; one board action then resolve.

### Requirement: Flipped peek consumes peeks budget

**Reason**: Peeks budget removed.
**Migration**: Reopen only by a later legal land on the open cell.

### Requirement: Auto end-turn gates flipped peek by peeks budget

**Reason**: End-turn and peeks gates removed.
**Migration**: Turn advances after land resolve / modal close.

### Requirement: Auto end-turn waits for available rescue or return

**Reason**: End-turn removed; rescue is free before the one board action.
**Migration**: Player rescues then moves/returns; turn advances after resolve.

### Requirement: Moves consume steps and do not advance the turn

**Reason**: No steps; successful move starts resolve then advances.
**Migration**: One move per turn then advance after open-on-land settle.

### Requirement: Push adjacent free tourist through for one step

**Reason**: Push removed from the product.
**Migration**: Delete push message, affordances, and tests.

### Requirement: Push affordances on targets of the selected pusher

**Reason**: Push removed from the product.
**Migration**: Remove push chrome; keep rescue affordance when applicable.

### Requirement: Rescue own adjacent trapped piece for one step

**Reason**: Superseded by free rescue requirement (no steps).
**Migration**: Use ADDED free rescue until board action.

### Requirement: Playing seeds hidden catapults by density

**Reason**: Fixed exclusive 12%/15% partition lives in `game/board`; density/overlap seed removed.
**Migration**: Server ignores create density; exclusive cells only.
