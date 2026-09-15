# game/finish Specification

## Purpose

Финиш партии в room `tourist`: турист, вошедший в центр, сходит с поля; игрок, проводивший всех четырёх, получает место, остаётся за столом без ходов (say/strip/presence) и освобождает seat только при выходе. Связано с `game/move`, `game/pieces`, `game/leave`, `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-01 | covered (server mocha) |
| SC-FINISH-02 | covered (server mocha) |
| SC-FINISH-03 | covered (server mocha + client UX modal) |
| SC-FINISH-04 | covered (server mocha + client UX modal) |
| SC-FINISH-05 | covered (server: no auto-dispose; client stays in room) |
| SC-FINISH-06 | covered (server mocha move reject; client strip/say) |
| SC-FINISH-07 | covered (server mocha) |
| SC-FINISH-08 | covered (server mocha) |
| SC-FINISH-09 | covered (client UX strip) |
| SC-FINISH-10 | covered (client UX strip) |
| SC-FINISH-11 | covered (server mocha) |

## Requirements

### Requirement: Entering any center cell finishes a piece

When the room start phase is `playing` and the server accepts a legal move whose target is any of the four center cells of the tourist layout, the server SHALL mark that piece finished. A finished piece MUST NOT occupy any board cell for subsequent move validation (the center cell becomes free for other pieces immediately). Every client that displays the board MUST remove that piece from the board after a short disappear animation on the landing cell (no travel toward the strip). Finished piece state MUST be synchronized to all clients in the room.

#### Scenario [SC-FINISH-01]: Move onto a center cell finishes the piece

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** that piece is marked finished in synced state
- **AND** that center cell is not occupied by that piece for later moves
- **AND** every client observes the piece leave the board after a short disappear animation

#### Scenario [SC-FINISH-02]: Another piece may reuse the same center cell

- **GIVEN** a piece has just finished on a specific center cell and no other unfinished piece occupies that cell
- **WHEN** another unfinished piece legally moves onto that same center cell on a later turn
- **THEN** the server accepts the move and finishes that second piece
- **AND** the first finished piece remains finished off the board

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

A seat that has a finish place SHALL remain a seated player: it MUST keep its tourist kind, personal strip, presence marker, and eligibility to send whitelist say intents per `game/say`. That seat MUST NOT successfully submit board moves. The seat MUST continue to count toward `maxSeats` occupancy until permanently removed by consented leave or reconnect grace timeout. While any finished seat remains, a joining client MUST receive a new seat only if `seats.size < maxSeats` after counting finished seats. Spectators remain connections without seats and MUST NOT keep the room alive: when seated count reaches zero the room is disposed even if spectators remain (unchanged dispose rule).

#### Scenario [SC-FINISH-06]: Finished player keeps strip and may say

- **GIVEN** a seated player who has finish place `1` and is connected
- **WHEN** that player views the Game screen and submits a whitelist say intent
- **THEN** the personal four-slot strip remains visible for that player
- **AND** the say is accepted and broadcast per `game/say`
- **AND** a board move from that seat is rejected without changing piece positions

#### Scenario [SC-FINISH-07]: Finished seat blocks mid-game seating until leave

- **GIVEN** a tourist room with `maxSeats` equal to 2 and both seats finished (each has a finish place)
- **WHEN** another authenticated client joins the room
- **THEN** that client receives no seat and no pieces (spectator)
- **AND** after one finished seat permanently leaves, a subsequent joiner MAY receive a seat with four pieces from scratch while capacity remains

#### Scenario [SC-FINISH-08]: Dispose only when no seats remain

- **GIVEN** a tourist room whose only remaining seats are finished seats, plus zero or more spectators
- **WHEN** every seated player permanently leaves until seated count is zero
- **THEN** the room is disposed
- **AND** spectators alone MUST NOT prevent disposal

### Requirement: Finished strip slots and place chrome

For a seated client, each personal strip slot whose matching piece is finished MUST show a finish indicator at the top-right of that slot and MUST NOT be usable to select or submit a move. Slots for unfinished pieces keep existing selection behavior while it is that seat’s turn and the seat is not finished. After the seat has a finish place, all four strip slots MUST remain visible with finish indicators.

#### Scenario [SC-FINISH-09]: Finished strip slot is inactive with indicator

- **GIVEN** a seated player has finished exactly one of four pieces and still has unfinished pieces
- **WHEN** that player views the personal strip
- **THEN** the finished piece’s strip slot shows a finish indicator at the top-right
- **AND** activating that finished slot does not select a piece or submit a move

#### Scenario [SC-FINISH-10]: All strip slots stay after full finish

- **GIVEN** a seated player has finish place assigned
- **WHEN** that player views the Game screen
- **THEN** the personal strip still shows four slots of that player’s tourist kind
- **AND** each slot shows a finish indicator

### Requirement: Reconnect grace for finished seats

An unexpected disconnect of a finished seat MUST hold the seat for the same reconnect grace as an active seat (`game/pieces` / `game/presence`). Turn handling for finished seats follows `game/move` (finished seats are skipped and MUST NOT hold the current turn while any non-finished seated player remains).

#### Scenario [SC-FINISH-11]: Finished disconnect keeps seat for grace

- **GIVEN** a finished seated player disconnects unexpectedly
- **WHEN** the reconnect grace has not yet expired
- **THEN** the seat remains occupied with offline grace state
- **AND** the tourist kind and finish place remain assigned to that seat until reconnect or timeout
