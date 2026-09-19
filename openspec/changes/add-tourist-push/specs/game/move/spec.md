## Purpose

Delta этого change: действие push (толкнуть), стоимость в steps, side-effects посадки, UX affordances (вкл. centering), push→finish travel/fade, auto-end. Базовые budgets/turn/move — main `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-66 | covered (server mocha — push costs 1 step, relocates target) |
| SC-MOVE-67 | covered (server mocha + unit — collinear through geometry) |
| SC-MOVE-68 | covered (server mocha + unit — cannot push trapped) |
| SC-MOVE-69 | covered (server mocha — push onto center finishes target) |
| SC-MOVE-70 | covered (server mocha — push onto grille traps target) |
| SC-MOVE-71 | covered (server mocha + unit — reject hole / occupied / no steps) |
| SC-MOVE-72 | covered (server mocha — push does not advance turn) |
| SC-MOVE-73 | covered (server mocha — auto-end waits for legal push) |
| SC-MOVE-74 | covered (client UX — push icons centered above targets) |
| SC-MOVE-75 | covered (client UX — approach/back + target travel; keep selection) |
| SC-MOVE-76 | covered (client UX — push onto center: travel then disappear like move) |
| SC-MOVE-77 | covered (client UX — rescue affordance centered above trapped) |

Related: finish side-effect — `game/finish`; peek centering — delta `game/board`; return strip UX — delta `game/finish`.

## ADDED Requirements

### Requirement: Push adjacent free tourist through for one step

While it is a seated client’s own turn with steps ≥ 1, that client MAY push another unfinished **free** (not trapped) piece that stands at Chebyshev distance 1 from one of the current seat’s own unfinished **free** pieces (the pusher), when the cell on the far side of the target — continuing the same Chebyshev delta from pusher through target — is a legal landing cell for that target as if the target performed an ordinary one-step move onto it (playable, not a removed-task hole, not occupied by any unfinished piece). The target MAY belong to the same seat or another seat. A successful push MUST decrease the current seat’s steps by 1, relocate the target piece onto that destination cell, and MUST NOT permanently relocate the pusher (approach-and-back animation is client presentation only). The pusher’s selection on the submitting client MUST remain on that pusher after a successful push. Push MUST NOT advance the turn solely because of the push. Side-effects of the destination MUST follow ordinary move landing rules: landing on a center cell finishes the **target** piece per `game/finish`; landing on an unspent grille traps the **target** per existing grille rules. Pushing a trapped piece MUST be rejected. Destinations that are removed holes, non-playable, or occupied MUST be rejected without spending a step.

#### Scenario [SC-MOVE-66]: Push spends one step and moves the target

- **GIVEN** it is a seated player’s turn with steps ≥ 1, one own free unfinished piece adjacent to another free unfinished piece (own or other seat), and the far-side cell through that target is a free playable non-center non-hole cell
- **WHEN** that player submits a push of that target through that destination
- **THEN** steps decrease by 1
- **AND** the target piece occupies the destination cell
- **AND** the pusher remains on its original cell

#### Scenario [SC-MOVE-67]: Push direction is collinear through the target

- **GIVEN** the pusher is at cell A, the target at cell B with Chebyshev distance 1, and destination C equals B plus the same delta as B minus A
- **WHEN** that player submits a push identifying that pusher, target, and C
- **THEN** the server accepts only when C is that far-side cell and is a legal landing for the target
- **AND** a destination that is not that far-side cell is rejected

#### Scenario [SC-MOVE-68]: Cannot push a trapped tourist

- **GIVEN** it is a seated player’s turn with steps ≥ 1 and an own free piece adjacent to a trapped piece whose far-side cell would otherwise be landable
- **WHEN** that player submits a push targeting the trapped piece
- **THEN** the server rejects the push
- **AND** steps and piece positions do not change from that attempt

#### Scenario [SC-MOVE-69]: Push onto center finishes the target

- **GIVEN** it is a seated player’s turn with steps ≥ 1, an own free pusher adjacent to a free unfinished target, and the far-side cell is a free center cell
- **WHEN** that player submits a push onto that center cell
- **THEN** the target piece is marked finished per `game/finish`
- **AND** steps decrease by 1
- **AND** the turn does not advance solely because of that push

#### Scenario [SC-MOVE-70]: Push onto grille traps the target

- **GIVEN** it is a seated player’s turn with steps ≥ 1, an own free pusher adjacent to a free unfinished target, and the far-side cell still holds an unspent grille
- **WHEN** that player submits a push onto that cell
- **THEN** steps decrease by 1
- **AND** the target piece is marked trapped on that cell
- **AND** the grille becomes visible per `game/board`

#### Scenario [SC-MOVE-71]: Push rejected for hole, occupied, or no steps

- **GIVEN** it is a seated player’s turn and either steps are 0, or the far-side cell is a removed-task hole or occupied
- **WHEN** that player submits a push toward that far-side cell
- **THEN** the server rejects the push
- **AND** steps and piece positions do not change from that attempt

#### Scenario [SC-MOVE-72]: Push does not advance the turn

- **GIVEN** it is a seated player’s turn with steps ≥ 2 and a legal push exists
- **WHEN** that player successfully pushes one target
- **THEN** that seat remains the current turn
- **AND** with remaining steps the seat MAY push again or take another legal action

### Requirement: Push affordances on targets of the selected pusher

While it is the user’s own turn with steps ≥ 1 and a free unfinished own piece is locally selected, the client MUST show a green push affordance **centered above** each unfinished free piece (own or other) that that selected piece can legally push under the push requirement (same top-center family as strip return). The peek eye affordance MUST remain on the selected own piece when peek is available (centered above that piece per `game/board`). Push affordances MUST NOT appear for trapped targets, when no piece is selected, when it is not the user’s turn, or when steps are 0. Activating a push affordance MUST submit that push. Destination cells for push MUST NOT use red move-target rings; only the affordance initiates push. After a successful push the submitting client MUST keep selection on the pusher. The client MUST present an approach-and-back animation of the pusher toward the target and ordinary piece travel of the target to the destination (visible to the submitting client for approach; travel visible to clients that display the board). When the destination is a center cell and the target finishes, every client that displays the board MUST present the same travel-to-landing then disappear animation as for an ordinary move onto center (`game/finish` / SC-FINISH-01) — the target MUST NOT vanish without traveling to the center cell.

#### Scenario [SC-MOVE-74]: Push icons appear over pushable neighbors of the selection

- **GIVEN** it is the user’s turn with steps ≥ 1, one own free piece is selected, and two different free neighbors are each legally pushable through their far-side cells
- **WHEN** the board renders affordances
- **THEN** a push affordance is shown centered above each of those two targets
- **AND** no push affordance is shown over the selected pusher for those actions
- **AND** if a peek is also available for the selected piece, the eye remains centered above the selected piece

#### Scenario [SC-MOVE-75]: Push click animates and keeps selection

- **GIVEN** the same turn with a selected pusher and one push affordance visible
- **WHEN** the user activates that affordance and the server accepts the push
- **THEN** the pusher shows an approach-and-back presentation
- **AND** the target travels to the destination
- **AND** local selection remains on the pusher

#### Scenario [SC-MOVE-76]: Push onto center travels then disappears

- **GIVEN** it is a seated player’s turn with a legal push whose far-side cell is a free center cell
- **WHEN** that player successfully pushes the target onto that center cell
- **THEN** clients that display the board show the target traveling onto the center cell
- **AND** then removing that piece with the same short disappear animation as an ordinary center finish
- **AND** the target does not disappear from its pre-push cell without that travel

### Requirement: Rescue affordance centered above trapped tourist

While it is the user’s own turn with steps ≥ 1 and a legal own rescue exists for a trapped unfinished piece, the client MUST show the green rescue affordance **centered above** that trapped tourist on the board (same top-center family as push / peek / strip return).

#### Scenario [SC-MOVE-77]: Rescue icon centered above trapped

- **GIVEN** it is the user’s turn with steps ≥ 1 and a legal own rescue exists
- **WHEN** the board renders affordances
- **THEN** a rescue affordance is shown centered above that trapped tourist

## MODIFIED Requirements

### Requirement: Player end-turn and auto end-turn when no actions remain

While phase is `playing`, at least two eligible seats remain, and it is a non-finished non-time-expired seat’s turn, that client MUST be able to submit an end-turn action. On acceptance the server MUST advance the turn to the next eligible seat in join order and grant that seat +1 step and +1 peek per the budget requirement. The server MUST **not** auto-advance while the current seat still has peeks ≥ 1 **and** at least one own unfinished piece stands on a still-present task cell (a legal peek remains available), even if steps are 0. The server MUST also treat a legal own rescue, a legal own return (when finish place is 0), or a legal own push as available actions. When the current seat has **no available actions** — cannot legally move (steps 0 or no legal destinations including no landing on removed holes) **and** does not have (peeks ≥ 1 and an own unfinished piece on a still-present task cell) **and** has no legal rescue, return, or push — the server MUST auto-advance the turn the same way without waiting for the deadline. Solo mode MUST NOT offer or require end-turn advancement among seats.

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

### Requirement: Auto end-turn waits for available rescue or return

While evaluating whether the current seat has no available actions for auto end-turn, the server MUST treat a legal own rescue, a legal own return (when finish place is 0), and a legal own push as available actions. Auto end-turn MUST NOT fire solely because no board move remains if rescue, return, or push is still legal under steps and geometry rules.

#### Scenario [SC-MOVE-62]: Auto-end deferred while rescue is legal

- **GIVEN** it is a seated player’s turn with steps ≥ 1, no legal board move for any free piece, and a legal own rescue exists
- **WHEN** the server evaluates auto end-turn
- **THEN** the turn does not auto-end solely for lack of board moves
- **AND** the seat may still submit that rescue

#### Scenario [SC-MOVE-73]: Auto-end deferred while push is legal

- **GIVEN** it is a seated player’s turn with steps ≥ 1, no legal board move for any free piece, no legal rescue or return, and a legal own push exists
- **WHEN** the server evaluates auto end-turn
- **THEN** the turn does not auto-end solely for lack of board moves
- **AND** the seat may still submit that push
