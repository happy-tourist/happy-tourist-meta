# Delta: game/move

Связано с main `openspec/specs/game/move/spec.md` и `game/board` (снятые тайлы) / `game/presence` (соло HUD). Этот change: снятые task-клетки снова landable для move/push/return/fling; соло — 60 с, ∞ steps + peeks, без steps-exhaustion. **Titles** `### Requirement` / `#### Scenario` совпадают с main (OpenSpec validate); **bodies / THEN** — продуктовый канон (design D5).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-29 | modified — server mocha (solo 60s) |
| SC-MOVE-30 | modified-by-reuse — server mocha (clear deadline; one-minute budget) |
| SC-MOVE-31 | modified — client timeout modal (one-minute copy context) |
| SC-MOVE-40 | modified — server mocha (solo ∞ steps+peeks; no step carry grant) |
| SC-MOVE-41 | covered-by-reuse (solo no end-turn) |
| SC-MOVE-45 | modified — server mocha (timer expiry; no steps-exhausted branch) |
| SC-MOVE-48 | modified — server mocha (no time-lock from “exhaustion”; ∞ steps) |
| SC-MOVE-49 | modified — server mocha + unit (land on removed accepted) |
| SC-MOVE-50 | modified — server mocha (become-current solo → ∞; no +1 step) |
| SC-MOVE-57 | modified — server mocha (return onto removed ring cell OK) |
| SC-MOVE-64 | modified — server mocha (solo rescue; steps not spent) |
| SC-MOVE-71 | modified — server mocha + unit (push onto removed OK; still reject occupied / no steps in multi) |
| SC-MOVE-80 | modified-by-reuse — fling may land on removed (legal landing includes removed) |
| SC-MOVE-97 | modified — server mocha (center finish under ∞ does not time-lock) |
| SC-MOVE-98 | modified — server mocha (no exhaustion gate) |
| SC-MOVE-99 | added — server mocha (solo move does not spend steps) |

## MODIFIED Requirements

### Requirement: Private step and peek budgets per seat

While phase is `playing`, the server SHALL maintain for each non-finished seated player a private **steps** budget and a private **peeks** budget that are visible only to that seat’s client (not to other seated players or spectators). The private room message `budgets` continues to carry `{ steps, peeks, infinite, peekedThisTurn }`. In multiplayer, `infinite` is **false** and both counters are finite numbers. In solo, `infinite` MUST be **true** and SHALL mean **both** peeks and steps are unlimited (client shows ∞ for both; server MUST NOT decrement steps while `infinite`). At materialize into `playing` both budgets MUST start at **0**. Each time a seat **becomes** current turn:

- while **two or more** eligible seats remain, the server MUST add exactly **1** step and exactly **1** peek to that seat’s budgets before play continues;
- while **exactly one** eligible seat remains (solo), the server MUST set that seat’s **steps** and **peeks** to **infinite** mode (`infinite: true`) and MUST **not** apply a finite +1 step or +1 peek grant.

Budgets MUST persist across that seat’s turns (remainder carries over in multiplayer finite mode). Becoming solo **without** a change of current turn (another seat permanently left or finished while this seat was already current) MUST switch that seat to infinite steps and peeks without a finite +1 grant. While the same seat remains current under solo rules, successful moves MUST NOT trigger further turn grants and MUST NOT decrement steps. Reconnect within grace MUST restore the same budgets to the reclaiming client. Spectators MUST NOT receive step or peek budgets.

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
- **AND** B’s steps are infinite
- **AND** B is not given a finite +1 step grant
- **AND** B may legally move without spending steps

### Requirement: Moves consume steps and do not advance the turn

A legal one-step move in phase `playing` MUST decrement the mover’s steps budget by exactly **1** when the seat’s steps budget is **finite**. When steps are **infinite** (solo), a legal move MUST NOT decrement steps. The server MUST reject a move that would require a step while the seat’s steps budget is **0** in finite mode without changing piece positions or the current turn. After an accepted move the current turn MUST **remain** on that seat; the turn MUST NOT advance solely because a move succeeded. Orthogonal/diagonal adjacency, occupancy, playable cells (**including** removed task cells as landing targets per `game/board`), center finish, and finished/time-expired gates remain as previously specified except where this change modifies turn advancement.

#### Scenario [SC-MOVE-35]: Successful move spends one step and keeps the turn

- **GIVEN** the room phase is `playing`, it is a seated non-finished non-time-expired player’s turn with finite steps ≥ 1, and a legal neighbor cell exists for one unfinished piece
- **WHEN** that player submits a legal move to that cell
- **THEN** the server updates the piece position
- **AND** that seat’s steps budget decreases by 1
- **AND** the current turn remains that same seat

#### Scenario [SC-MOVE-36]: Move with zero steps is rejected

- **GIVEN** it is a seated player’s turn with finite steps 0
- **WHEN** that player submits a move message
- **THEN** the server rejects the move
- **AND** piece positions, peeks budget, and the current turn remain unchanged

#### Scenario [SC-MOVE-99]: Solo move does not spend steps

- **GIVEN** it is a solo eligible seat’s turn under infinite steps and peeks
- **WHEN** that player submits a legal move
- **THEN** the server updates the piece position
- **AND** steps remain infinite (not decremented)
- **AND** the current turn remains that same seat

### Requirement: Player end-turn and auto end-turn when no actions remain

While phase is `playing`, at least two eligible seats remain, and it is a non-finished non-time-expired seat’s turn, that client MUST be able to submit an end-turn action. On acceptance the server MUST advance the turn to the next eligible seat in join order and grant that seat +1 step and +1 peek per the budget requirement. The server MUST **not** auto-advance while the current seat still has peeks ≥ 1 **and** at least one own unfinished piece stands on a still-present task cell (a legal peek remains available), even if steps are 0. The server MUST also treat a legal own rescue, a legal own return (when finish place is 0), or a legal own push as available actions. When the current seat has **no available actions** — cannot legally move (steps 0 or no legal destinations; removed task cells **are** legal destinations when otherwise playable) **and** does not have (peeks ≥ 1 and an own unfinished piece on a still-present task cell) **and** has no legal rescue, return, or push — the server MUST auto-advance the turn the same way without waiting for the deadline. Solo mode MUST NOT offer or require end-turn advancement among seats.

#### Scenario [SC-MOVE-37]: End-turn advances and grants the next seat

- **GIVEN** a tourist room in phase `playing` with two eligible seated players A then B, it is A’s turn, and A still has steps or peeks remaining
- **WHEN** A submits end-turn
- **THEN** the current turn becomes B
- **AND** B’s private budgets increase by 1 step and 1 peek
- **AND** no piece positions change solely because of end-turn

#### Scenario [SC-MOVE-38]: Auto end-turn when no actions remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0 and peeks 0 (or peeks remain but no own unfinished piece stands on a still-present task cell), so no legal move and no available peek remain
- **WHEN** the server evaluates available actions
- **THEN** the turn advances to the next eligible seat without waiting for the deadline

#### Scenario [SC-MOVE-47]: Auto end-turn does not fire while peeks and a live task tile remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0, peeks ≥ 1, and an own unfinished piece on a still-present task cell
- **WHEN** the server evaluates available actions
- **THEN** the turn does not auto-advance solely because steps are 0

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

### Requirement: Solo infinite peeks and finite steps

When phase is `playing` and exactly one seated player remains eligible to move, the server SHALL set that seat’s **peeks** budget to an **infinite** mode and SHALL set that seat’s **steps** budget to an **infinite** mode, MUST NOT accept or require end-turn to keep playing, and MUST NOT apply further turn grants while that same seat remains current under solo (successful moves do not re-grant and do not spend steps). When another seat’s permanent leave or finish causes this seat to **become** current turn as the sole eligible seat, infinite steps and peeks MUST apply without a finite +1 step grant. When the seat was **already** current and merely becomes the sole eligible seat, steps and peeks MUST become infinite (any prior finite remainder is superseded; no finite +1). The one-minute solo deadline remains as the sole solo soft-lock besides finishing all pieces (`game/finish`). The server MUST **not** mark the seat time-expired because steps reached 0 or because no legal peek remains. Finishing all pieces follows `game/finish`. The solo client MUST be informed via a modal that they are alone, that moves and peeks are unlimited, and that they have one minute. Other clients MUST NOT see that seat’s budget values.

#### Scenario [SC-MOVE-40]: Becoming solo while already current carries steps

- **GIVEN** a tourist room in phase `playing` where it is non-finished seated player A’s turn with a finite steps budget greater than 0, and every other seated player either receives a finish place or permanently leaves, leaving A as the only eligible seat **without** changing current turn away from A
- **WHEN** that condition becomes true
- **THEN** A’s peeks are infinite and A’s steps are infinite
- **AND** that client is shown the solo unlimited-budgets modal
- **AND** the synchronized one-minute solo deadline applies as specified

#### Scenario [SC-MOVE-41]: Solo needs no end-turn

- **GIVEN** a non-finished seated player under solo infinite peeks and infinite steps
- **WHEN** that player views Game controls for ending a turn
- **THEN** the end-turn control is not offered
- **AND** successful moves do not pass the turn to another seat
- **AND** successful moves do not grant an additional +1 step solely because the seat remains solo current

#### Scenario [SC-MOVE-48]: Solo steps exhaustion without a live task tile time-locks

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seated player under solo infinite peeks and infinite steps, and no own free unfinished piece on a still-present task cell
- **WHEN** the server evaluates solo end conditions
- **THEN** that seat does **not** become time-expired solely because of missing peeks or step count
- **AND** the synchronized one-minute solo deadline keeps running until timer expiry or finish

#### Scenario [SC-MOVE-97]: Last-step ordinary center finish with start and jail remainder time-locks

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seated player under solo infinite steps and peeks, one own free unfinished piece adjacent to a free center cell, at least one other own unfinished free piece on a start cell, and one own unfinished trapped piece (grille), with finish place still 0
- **WHEN** that player successfully moves the free piece onto the center cell (ordinary move, not a trap-pipeline fling), finishing that piece without assigning a finish place
- **THEN** that seat’s steps remain infinite
- **AND** that seat does **not** become time-expired
- **AND** the synchronized solo turn deadline keeps ticking
- **AND** further moves and peeks from that seat remain allowed

#### Scenario [SC-MOVE-98]: Trapped piece on live task does not block steps exhaustion

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seated player under solo infinite steps and peeks, and every own unfinished piece is either trapped or not on a still-present task cell
- **WHEN** the server evaluates solo end conditions
- **THEN** that seat does **not** become time-expired solely for that reason
- **AND** the synchronized solo turn deadline is not cleared by that evaluation

### Requirement: Solo endgame five-minute budget

When phase is `playing` and exactly one seated player remains eligible to move (all other seats either hold a finish place or have permanently left), the server SHALL give that seat a synchronized turn deadline of exactly **one minute** (60 seconds) from the moment that condition becomes true, replacing any prior multiplayer turn deadline, and SHALL apply solo infinite peeks and infinite steps as specified above. While that solo budget runs, the seat remains current turn after its own successful moves without resetting the remaining solo budget to a new one minute on each move and without +1/+1 turn grants; if the player finishes all four pieces before the deadline, normal finish rules apply (`game/finish`) and the turn deadline MUST clear. Spectators and other clients MUST observe the same synchronized solo deadline.

#### Scenario [SC-MOVE-29]: Solo remaining player gets five minutes

- **GIVEN** a tourist room in phase `playing` where every other seated player has either a finish place or has permanently left, leaving exactly one non-finished seated player
- **WHEN** that condition becomes true
- **THEN** that seat’s synchronized turn deadline is exactly one minute (60 seconds) from that moment
- **AND** every client observes that one-minute deadline

#### Scenario [SC-MOVE-30]: Successful solo finish clears the deadline

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under the one-minute solo budget
- **WHEN** that player finishes their last piece and receives a finish place
- **THEN** the turn deadline for that seat is cleared
- **AND** the seat follows finished-seat rules without a time-expired lock

#### Scenario [SC-MOVE-45]: Solo timer expiration time-locks with timer copy

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under the one-minute solo budget with infinite peeks and infinite steps
- **WHEN** the one-minute deadline elapses without finishing all pieces
- **THEN** that seat becomes time-expired and further moves and peeks are rejected
- **AND** the client is shown the timer-expired end copy

### Requirement: Solo budget expiry locks moves

When the one-minute solo budget elapses without that seat finishing all pieces, the server SHALL mark that seat time-expired in synchronized state, MUST clear the active turn deadline, and MUST reject further move messages from that seat without changing piece positions. The room MUST NOT dispose or force-leave solely because of that expiry; the seat remains until consented leave or reconnect grace timeout as for other seats. The time-expired client’s Game UI MUST show a modal whose product Russian sense is that the tourists were not finished in time; closing the modal MUST leave the player in the room. That client MUST clear any local piece selection and move-destination hints and MUST NOT offer board/strip move interaction while time-expired. Other clients MUST NOT show that modal. Say and presence for the seat remain allowed as for a finished seat without a finish place.

#### Scenario [SC-MOVE-31]: Solo timeout locks and shows modal only to that player

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player whose one-minute solo budget has just elapsed, and at least one spectator or finished seated client also in the room
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

### Requirement: Return a finished piece onto the center ring for one step

While it is a seated client’s own turn, that seat has finish place **0**, steps ≥ 1 in finite mode (or infinite steps in solo), and at least one own piece is finished, the client MAY return one finished piece onto a legal ring cell: any playable cell at Chebyshev distance 1 from the central 2×2 block (including diagonal corners relative to the block), including a **removed** task cell on that ring, that is not occupied by an unfinished piece. Layout holes that were never playable MUST NOT be return targets. A successful return MUST decrease steps by 1 only in finite mode, mark that piece unfinished at the chosen cell, and place it on the board for all clients. Return MUST be offered per finished strip slot whenever steps (or infinite) and at least one legal ring cell exist (not only in a jail deadlock).

#### Scenario [SC-MOVE-57]: Return spends one step and places on ring

- **GIVEN** it is a seated player’s turn with finite steps ≥ 1, finish place 0, one finished own piece, and a free playable ring cell around the center block (including a removed task cell)
- **WHEN** that player submits a return of that piece onto that cell
- **THEN** steps decrease by 1 in finite mode
- **AND** the piece is unfinished at that cell in synced state
- **AND** every client shows that piece on the board

#### Scenario [SC-MOVE-58]: Ring includes diagonal cells around the center block

- **GIVEN** it is a seated player’s turn eligible to return and a corner-adjacent playable cell just outside the central 2×2 is free (including a removed task cell)
- **WHEN** that player chooses that cell as the return target
- **THEN** the server accepts the return onto that cell

#### Scenario [SC-MOVE-59]: No return affordance without steps

- **GIVEN** it is a seated player’s multiplayer turn with finite steps 0 and at least one finished own piece
- **WHEN** the client renders return controls
- **THEN** return is not offered as an actionable control
- **AND** a return message is rejected if sent

### Requirement: Push adjacent free tourist through for one step

While it is a seated client’s own turn with steps ≥ 1 in finite mode (or infinite steps in solo), that client MAY push another unfinished **free** (not trapped) piece that stands at Chebyshev distance 1 from one of the current seat’s own unfinished **free** pieces (the pusher), when the cell on the far side of the target — continuing the same Chebyshev delta from pusher through target — is a legal landing cell for that target as if the target performed an ordinary one-step move onto it (playable including removed task cells, not a never-playable layout hole, not occupied by any unfinished piece). The target MAY belong to the same seat or another seat. A successful push MUST decrease the current seat’s steps by 1 only in finite mode, relocate the target piece onto that destination cell, and MUST NOT permanently relocate the pusher (approach-and-back animation is client presentation only). The pusher’s selection on the submitting client MUST remain on that pusher after a successful push. Push MUST NOT advance the turn solely because of the push. Side-effects of the destination MUST follow ordinary move landing rules: landing on a center cell finishes the **target** piece per `game/finish`; landing on an unspent grille traps the **target** per existing grille rules. Pushing a trapped piece MUST be rejected. Destinations that are never-playable layout holes or occupied MUST be rejected without spending a step. Removed task cells MUST be accepted when otherwise legal.

#### Scenario [SC-MOVE-66]: Push spends one step and moves the target

- **GIVEN** it is a seated player’s turn with finite steps ≥ 1, one own free unfinished piece adjacent to another free unfinished piece (own or other seat), and the far-side cell through that target is a free playable non-center cell (including a removed task cell)
- **WHEN** that player submits a push of that target through that destination
- **THEN** steps decrease by 1 in finite mode
- **AND** the target piece occupies the destination cell
- **AND** the pusher remains on its original cell

#### Scenario [SC-MOVE-67]: Push direction is collinear through the target

- **GIVEN** the pusher is at cell A, the target at cell B with Chebyshev distance 1, and destination C equals B plus the same delta as B minus A
- **WHEN** that player submits a push identifying that pusher, target, and C
- **THEN** the server accepts only when C is that far-side cell and is a legal landing for the target
- **AND** a destination that is not that far-side cell is rejected

#### Scenario [SC-MOVE-68]: Cannot push a trapped tourist

- **GIVEN** it is a seated player’s turn with steps ≥ 1 (or infinite) and an own free piece adjacent to a trapped piece whose far-side cell would otherwise be landable
- **WHEN** that player submits a push targeting the trapped piece
- **THEN** the server rejects the push
- **AND** steps and piece positions do not change from that attempt

#### Scenario [SC-MOVE-69]: Push onto center finishes the target

- **GIVEN** it is a seated player’s turn with finite steps ≥ 1, an own free pusher adjacent to a free unfinished target, and the far-side cell is a free center cell
- **WHEN** that player submits a push onto that center cell
- **THEN** the target piece is marked finished per `game/finish`
- **AND** steps decrease by 1 in finite mode
- **AND** the turn does not advance solely because of that push

#### Scenario [SC-MOVE-70]: Push onto grille traps the target

- **GIVEN** it is a seated player’s turn with finite steps ≥ 1, an own free pusher adjacent to a free unfinished target, and the far-side cell still holds an unspent grille
- **WHEN** that player submits a push onto that cell
- **THEN** steps decrease by 1 in finite mode
- **AND** the target piece is marked trapped on that cell
- **AND** the grille becomes visible per `game/board`

#### Scenario [SC-MOVE-71]: Push rejected for hole, occupied, or no steps

- **GIVEN** it is a seated player’s multiplayer turn and either finite steps are 0, or the far-side cell is occupied
- **WHEN** that player submits a push toward that far-side cell
- **THEN** the server rejects the push
- **AND** steps and piece positions do not change from that attempt
- **AND** when instead the far-side cell is a free removed task hole and steps ≥ 1 (or infinite), the push is accepted onto that cell

#### Scenario [SC-MOVE-72]: Push does not advance the turn

- **GIVEN** a successful push in multiplayer
- **WHEN** the push is accepted
- **THEN** the current turn remains on the pushing seat

### Requirement: Solo uses the same grille rules

When exactly one eligible seat remains (solo), grille land/trap, rescue, return, and all-jail reset MUST follow the same rules as multiplayer, subject to existing solo infinite budgets and one-minute timer rules (`game/move` main). Solo rescue, return, and push MUST NOT decrement steps while steps are infinite.

#### Scenario [SC-MOVE-64]: Solo rescue costs one finite step

- **GIVEN** a solo eligible seat with infinite steps, one own trapped piece, and one own free piece at Chebyshev-1
- **WHEN** that seat submits a rescue
- **THEN** steps remain infinite
- **AND** the trapped piece becomes free

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
