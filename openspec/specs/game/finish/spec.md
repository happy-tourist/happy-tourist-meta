# game/finish Specification

## Purpose

Финиш партии в room `tourist`: турист, вошедший в центр, сходит с поля; игрок, проводивший всех четырёх, получает место, остаётся за столом без ходов (say/strip/presence) и освобождает seat только при выходе. Связано с `game/move`, `game/pieces`, `game/leave`, `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-01 | covered (server mocha + client UX — move onto center travel then disappear) |
| SC-FINISH-02 | covered (server mocha) |
| SC-FINISH-03 | covered (server mocha + client UX modal) |
| SC-FINISH-04 | covered (server mocha + client UX modal) |
| SC-FINISH-05 | covered (server: no auto-dispose; client stays in room) |
| SC-FINISH-06 | covered-by-reuse (server mocha + client) |
| SC-FINISH-07 | covered (server mocha — no seat after leave in playing) |
| SC-FINISH-08 | covered-by-reuse (server mocha) |
| SC-FINISH-09 | covered (client UX strip — conditional dim) |
| SC-FINISH-10 | covered (client UX strip) |
| SC-FINISH-11 | covered (server mocha) |
| SC-FINISH-12 | covered (server mocha — return clears finished) |
| SC-FINISH-13 | covered (client UX — green return icon; no confirm modal) |
| SC-FINISH-14 | covered (server mocha — strip selectable after return) |
| SC-FINISH-15 | covered (client UX — return travel from nearest center) |
| SC-FINISH-16 | covered (client UX — finished strip slot click does not start return) |
| SC-FINISH-17 | covered (client UX — push onto center travel+disappear) |
| SC-FINISH-18 | covered (client UX — move onto center seeds lastKnown; travel+disappear parity) |

Related: seating gate — `game/pieces` (new seats only in `waiting`); return geometry and steps — `game/move`; push finish — `game/move` SC-MOVE-76.

## Requirements

### Requirement: Entering any center cell finishes a piece

When the room start phase is `playing` and the server accepts a legal **move or push** whose landing target is any of the four center cells of the tourist layout, the server SHALL mark that piece finished. A finished piece MUST NOT occupy any board cell for subsequent move validation (the center cell becomes free for other pieces immediately). Every client that displays the board MUST remove that piece from the board after travel onto the landing cell and a short disappear animation on that cell (no travel toward the strip) — for finish caused by a **move** and by a **push**, with the same presentation. The piece MUST NOT vanish from its pre-finish board cell without that travel. Finished piece state MUST be synchronized to all clients in the room.

#### Scenario [SC-FINISH-01]: Move onto a center cell finishes the piece

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** that piece is marked finished in synced state
- **AND** that center cell is not occupied by that piece for later moves
- **AND** every client observes the piece travel onto the center cell then leave the board after a short disappear animation

#### Scenario [SC-FINISH-02]: Another piece may reuse the same center cell

- **GIVEN** a piece has just finished on a specific center cell and no other unfinished piece occupies that cell
- **WHEN** another unfinished piece legally moves onto that same center cell on a later turn
- **THEN** the server accepts the move and finishes that second piece
- **AND** the first finished piece remains finished off the board

#### Scenario [SC-FINISH-17]: Push onto a center cell finishes with the same presentation

- **GIVEN** a legal push whose far-side cell is a free center cell
- **WHEN** the push is accepted and the target finishes
- **THEN** every client that displays the board observes the target travel onto that center cell then leave after the same short disappear animation as a move finish
- **AND** the target does not vanish from its pre-push cell without that travel

#### Scenario [SC-FINISH-18]: Own move onto center travels from the pre-move cell

- **GIVEN** it is a seated player’s turn and an own unfinished piece has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell and the server accepts
- **THEN** clients that display the board show that piece traveling from its pre-move cell onto the center cell
- **AND** then leaving after the same short disappear animation used for push→center
- **AND** the piece does not vanish from its pre-move cell without that travel

### Requirement: Completing all four pieces assigns finish place

When a seated player’s fourth piece becomes finished, the server SHALL assign that seat a finish place equal to one plus the number of seats that already hold a finish place in this room (monotonic order of full finishes). The place MUST be synchronized. The client of that seat MUST show a modal congratulating the player on finishing in that place (product Russian sense: first / second / …). Closing the modal MUST leave the player in the room as a finished seat. The room MUST NOT disconnect or dispose solely because one or all seats became finished.

#### Scenario [SC-FINISH-03]: First full finisher gets place one

- **GIVEN** a tourist room in phase `playing` where no seat yet has a finish place, and a seated player has exactly three finished pieces and one unfinished piece adjacent to a free center cell
- **WHEN** that player’s move finishes the fourth piece
- **THEN** that seat’s synchronized finish place is `1`
- **AND** that player’s client shows a place congratulation modal
- **AND** the seat remains in the room after the modal is closed

#### Scenario [SC-FINISH-04]: Later full finisher gets the next place

- **GIVEN** a tourist room where one seat already has finish place `1` and another seated player finishes their fourth piece
- **WHEN** that fourth piece becomes finished
- **THEN** that second seat’s finish place is `2`
- **AND** that player’s client shows a congratulation modal for place `2`

#### Scenario [SC-FINISH-05]: Room stays open when all seats are finished

- **GIVEN** every seated player in the room has a finish place
- **WHEN** no further consented leave or grace timeout has removed seats
- **THEN** the room remains available
- **AND** seated clients are not forcibly disconnected solely because all seats are finished

### Requirement: Finished seat keeps seat capacity and limited agency

A seat that has a finish place SHALL remain a seated player: it MUST keep its tourist kind, personal strip, presence marker, and eligibility to send whitelist say intents per `game/say`. That seat MUST NOT successfully submit board moves. The seat MUST continue to count toward `maxSeats` occupancy until permanently removed by consented leave or reconnect grace timeout. While phase is `countdown` or `playing`, a joining client MUST NOT receive a new seat even when `seats.size < maxSeats` after counting finished seats (`game/pieces`). Spectators remain connections without seats and MUST NOT keep the room alive: when seated count reaches zero the room is disposed even if spectators remain (unchanged dispose rule).

#### Scenario [SC-FINISH-06]: Finished player keeps strip and may say

- **GIVEN** a seated player who has finish place `1` and is connected
- **WHEN** that player views the Game screen and submits a whitelist say intent
- **THEN** the personal four-slot strip remains visible for that player
- **AND** the say is accepted and broadcast per `game/say`
- **AND** a board move from that seat is rejected without changing piece positions

#### Scenario [SC-FINISH-07]: Finished seat blocks mid-game seating until leave

- **GIVEN** a tourist room in phase `playing` with `maxSeats` equal to 2 and both seats finished (each has a finish place)
- **WHEN** another authenticated client joins the room
- **THEN** that client receives no seat and no pieces (spectator)
- **AND** after one finished seat permanently leaves, a subsequent joiner still receives no seat and no pieces while phase remains `playing`

#### Scenario [SC-FINISH-08]: Dispose only when no seats remain

- **GIVEN** a tourist room whose only remaining seats are finished seats, plus zero or more spectators
- **WHEN** every seated player permanently leaves until seated count is zero
- **THEN** the room is disposed
- **AND** spectators alone MUST NOT prevent disposal

### Requirement: Finished strip slots and place chrome

For a seated client, each personal strip slot whose matching piece is finished MUST show a finish indicator at the top-right of that slot and MUST NOT be usable to select or submit a **move**. When return-from-finish is **not** currently available for that side (not the user’s interactive turn, steps < 1, finish place ≠ 0, or no legal free non-hole ring cell), that finished slot MUST be visually dimmed to signal that no action is available. When return **is** available for that side, the slot MUST NOT be dimmed solely because the piece is finished. Sides for unfinished pieces keep existing selection behavior while it is that seat’s turn and the seat is not finished. After the seat has a finish place, all four strip slots MUST remain visible with finish indicators (typically dimmed because return is unavailable).

#### Scenario [SC-FINISH-09]: Finished strip slot is inactive with indicator

- **GIVEN** a seated player has finished exactly one of four pieces and return is not available for that side
- **WHEN** that player views the personal strip
- **THEN** the finished piece’s strip slot shows a finish indicator at the top-right
- **AND** that slot is visually dimmed
- **AND** activating that finished slot does not select a piece, open a return modal, or submit a move

#### Scenario [SC-FINISH-10]: All strip slots stay after full finish

- **GIVEN** a seated player has finish place > 0 and all four pieces finished
- **WHEN** that player views the personal strip
- **THEN** all four strip slots remain visible with finish indicators

### Requirement: Reconnect grace for finished seats

An unexpected disconnect of a finished seat MUST hold the seat for the same reconnect grace as an active seat (`game/pieces` / `game/presence`). Turn handling for finished seats follows `game/move` (finished seats are skipped and MUST NOT hold the current turn while any non-finished seated player remains).

#### Scenario [SC-FINISH-11]: Finished disconnect keeps seat for grace

- **GIVEN** a finished seated player disconnects unexpectedly
- **WHEN** the reconnect grace has not yet expired
- **THEN** the seat remains occupied with offline grace state
- **AND** the tourist kind and finish place remain assigned to that seat until reconnect or timeout

### Requirement: Return from finish puts the piece back in play

When the server accepts a legal return-from-finish for a seat with finish place **0**, the targeted piece MUST become unfinished and MUST occupy the chosen ring cell. That piece MUST again count toward board occupancy and MUST be eligible for later moves under `game/move`. The strip slot for that side MUST no longer show a finished-only lock for selection once the piece is unfinished (subject to turn rules).

#### Scenario [SC-FINISH-12]: Returned piece is unfinished on the board

- **GIVEN** a seated player with finish place 0 has a finished piece and successfully returns it onto a legal ring cell
- **WHEN** synced state updates
- **THEN** that piece’s finished flag is false
- **AND** the piece occupies the chosen cell
- **AND** a later legal move of that piece MAY be accepted on a subsequent action of that turn or a later turn

#### Scenario [SC-FINISH-14]: Returned strip slot is usable again on turn

- **GIVEN** the same seat as SC-FINISH-12 still has the current turn after return
- **WHEN** that player activates the strip slot for the returned side
- **THEN** the slot is treated as an unfinished piece slot for selection
- **AND** it does not remain locked as a finished-only slot

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST show a green return affordance centered above that tourist in the personal strip (same visual family as board rescue/push affordances). Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above) and MUST NOT show the return affordance. Activating the finished strip slot body (the tourist image area) MUST NOT open a confirmation dialog and MUST NOT enter return-mode by itself — only the return affordance starts return. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a green return affordance is shown centered above that tourist (same visual family as board rescue/push)
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation

### Requirement: Return-from-finish travel animation

When the server accepts a legal return-from-finish, every client that displays the board MUST animate that piece traveling from the **nearest** of the four center finish cells to the chosen ring cell (Chebyshev distance to the ring cell; ties broken by lower row, then lower column), using the same travel timing family as ordinary piece moves. The animation MUST be visible to all clients in the room. Input locking for the submitting client during travel follows the same rules as move travel (`game/move`).

#### Scenario [SC-FINISH-15]: Everyone sees return leave the finish block

- **GIVEN** a seated player successfully returns a finished piece onto a legal ring cell
- **WHEN** clients update the board
- **THEN** every client shows that piece animating from the nearest center cell to that ring cell
- **AND** after the animation the piece occupies the ring cell as an unfinished piece
