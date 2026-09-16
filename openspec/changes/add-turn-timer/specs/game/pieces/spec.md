## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-01 | covered (server mocha — kind without pieces in waiting) |
| SC-PIECE-02 | covered-by-reuse (server mocha — unique kinds) |
| SC-PIECE-03 | covered (server mocha — spawn only on playing materialize) |
| SC-PIECE-04 | covered (server mocha — second seat cells after materialize) |
| SC-PIECE-05 | covered (server mocha — full table seat without pieces until playing) |
| SC-PIECE-06 | covered-by-reuse (server mocha — spectator) |
| SC-PIECE-07 | covered (server mocha — leave in waiting reopens) |
| SC-PIECE-08 | covered (server mocha — leave in playing does NOT reopen) |
| SC-PIECE-09 | covered-by-reuse (client UX) |
| SC-PIECE-10 | covered-by-reuse (client UX) |
| SC-PIECE-14 | covered (server mocha — grace timeout in playing does not reopen) |
| SC-PIECE-17 | covered (client hasOwnPieces gate) |
| SC-PIECE-18 | covered (server mocha — materialize on playing) |
| SC-PIECE-19 | covered (server mocha — playing join is spectator) |
| SC-PIECE-20 | covered-by-reuse (server mocha — dispose) |
| SC-PIECE-21 | covered (server mocha — finished occupancy; no reopen after leave in playing) |

Related: phase transition — `game/start`; turn timers — `game/move`; finish occupancy — `game/finish`.

## ADDED Requirements

### Requirement: Materialize pieces when playing begins

When the start phase transitions from `countdown` to `playing`, the server SHALL place exactly four pieces for every seated player that does not yet have pieces — one on each board side `N`, `E`, `S`, and `W` — choosing start cells uniformly at random among free start cells of each side, using that seat’s already assigned tourist kind. Every client MUST observe those pieces in synced state as phase becomes `playing`.

#### Scenario [SC-PIECE-18]: Waiting seats receive pieces at playing

- **GIVEN** a tourist room that completed countdown with two or more seated players who had tourist kinds but no pieces during `waiting`/`countdown`
- **WHEN** the synchronized phase becomes `playing`
- **THEN** each of those seats has exactly four pieces, one per side
- **AND** no two pieces in the room share the same cell
- **AND** every client observes those pieces

## MODIFIED Requirements

### Requirement: Four pieces per seated player

While the start phase is `waiting` and fewer seated players exist than maxSeats, when an authenticated client joins, the server SHALL assign that client a unique tourist kind from `1`…`4` not used by any current seat and MUST NOT place pieces yet; pieces for those seats are created when playing begins per the materialize requirement. All four pieces of the same player MUST use that player’s tourist kind. The assignment MUST be synchronized to all clients in the room. New seats MUST NOT be assigned while the phase is `countdown` or `playing` (see spectators requirement).

#### Scenario [SC-PIECE-01]: First join receives kind without pieces until playing

- **GIVEN** a tourist room in phase `waiting` that has no seated players and has free seat capacity
- **WHEN** an authenticated client joins the room
- **THEN** the client is assigned exactly one tourist kind from `1`…`4`
- **AND** the client has a seat with no pieces yet in synced state
- **AND** when the phase later becomes `playing`, that seat has exactly four pieces, one on each side `N`, `E`, `S`, and `W`
- **AND** every client in the room observes those pieces after materialize

#### Scenario [SC-PIECE-02]: Tourist kinds stay unique among players

- **GIVEN** a tourist room in phase `waiting` that already has one or more seated players and still has free seat capacity
- **WHEN** another authenticated client joins and receives a seat
- **THEN** the new tourist kind is not equal to any currently seated player’s kind
- **AND** when pieces exist for that seat, all four of the new player’s pieces use that same new kind

#### Scenario [SC-PIECE-03]: Start cells lie on the assigned side and stay free

- **GIVEN** a tourist room that is materializing pieces at playing start for seated players
- **WHEN** seats are assigned pieces on the four sides
- **THEN** each piece’s row and column match a start cell of that piece’s side on the agreed tourist layout
- **AND** no two pieces in the room share the same row and column

#### Scenario [SC-PIECE-04]: Second player uses remaining cells on each side

- **GIVEN** a tourist room that entered `playing` with two seated players who received seats during `waiting`
- **WHEN** pieces are materialized for both seats
- **THEN** each player has one piece on each side `N`, `E`, `S`, and `W`
- **AND** none of the second player’s cells equal any cell occupied by the first player

### Requirement: Spectators and hard stop after four seated players

The room MUST allow clients beyond maxSeats connections. The room’s maxSeats MUST be one of 2, 3, or 4 as set at create. While the phase is `waiting` and the number of seated players is strictly less than maxSeats, joining authenticated clients MUST receive a seat and a tourist kind (pieces wait until playing begins). When seated count equals maxSeats, or when the phase is `countdown` or `playing`, joining clients MUST NOT receive a seat, kind, or pieces (spectators / guests). Filling the last free seat while phase is `waiting` triggers start countdown per `game/start` (this requirement does not itself define countdown). The legacy meaning of `started` as a permanent seating lock after exactly four seats MUST NOT apply; seating lock for newcomers is the non-`waiting` phase.

#### Scenario [SC-PIECE-05]: Fourth seated player starts the room

- **GIVEN** a tourist room in phase `waiting` whose maxSeats equals 4 and that has three seated players
- **WHEN** a fourth authenticated client joins and receives a seat
- **THEN** seated count equals maxSeats
- **AND** that seat has a tourist kind and no pieces yet
- **AND** subsequent joining clients receive no seat and no pieces
- **AND** start countdown begins per `game/start`

#### Scenario [SC-PIECE-06]: Fifth connection is a spectator

- **GIVEN** a tourist room that already has seated count equal to maxSeats
- **WHEN** another authenticated client joins
- **THEN** that client receives no tourist kind and no pieces
- **AND** existing seats and pieces remain unchanged

#### Scenario [SC-PIECE-19]: Join in countdown or playing is spectator

- **GIVEN** a tourist room in phase `playing` with maxSeats 4 and two seated players
- **WHEN** another authenticated client joins
- **THEN** that client receives no seat, no tourist kind, and no pieces (spectator)
- **AND** existing seats and pieces remain unchanged

### Requirement: Leave before and after start

A **consented** leave by a seated player (intentional exit from the match UI back to the lobby) SHALL permanently remove that player’s seat and all pieces (if any). The tourist kind and occupied cells MUST return to the available pools whenever a seat is permanently removed, regardless of start phase. After that removal, if the phase is still `waiting` and seated count is again below maxSeats, a subsequent joiner MUST be allowed to receive a seat. If the phase is `countdown` or `playing`, a subsequent joiner MUST NOT receive a seat even when seated count is below maxSeats. An **unexpected** disconnect is governed by the reconnect grace requirement and MUST NOT be treated as an immediate consented leave. Ready marks of remaining seats are not cleared by this leave (see `game/start`). If seated count reaches zero, the room is disposed even if spectators remain (unchanged).

#### Scenario [SC-PIECE-07]: Leave before start frees kind and cells

- **GIVEN** a tourist room in phase `waiting` with at least one seated player
- **WHEN** that seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s seat is removed from synced state
- **AND** a subsequent joiner MAY receive the freed tourist kind and a seat while phase remains `waiting`

#### Scenario [SC-PIECE-08]: Leave after full table reopens seating

- **GIVEN** a tourist room in phase `playing` with maxSeats 4 and four seated players
- **WHEN** one seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s seat and pieces are removed from synced state
- **AND** a subsequent joiner receives no seat and no pieces while phase remains `playing`

#### Scenario [SC-PIECE-20]: Empty seated disposes room

- **GIVEN** a tourist room with exactly one seated player and any number of spectators
- **WHEN** that seated player permanently leaves
- **THEN** the room is disposed
- **AND** spectators do not keep the room alive

### Requirement: Unexpected disconnect grace and reconnect

When a seated player disconnects unexpectedly (for example page reload or network drop), the server MUST keep that seat and all pieces (if any) in synced state for a grace period of exactly **30 seconds** and MUST allow the same client session to reconnect into that seat within the grace. During the grace the seat MUST be marked offline with a synchronized reconnect deadline visible to all clients in the room. The same grace policy applies in any start phase. After grace timeout removes a seat, a subsequent joiner MAY receive a seat only while phase is `waiting` and capacity remains.

#### Scenario [SC-PIECE-11]: Unexpected disconnect before start holds the seat

- **GIVEN** a tourist room in phase `waiting` with at least one seated player who is marked connected
- **WHEN** that seated player disconnects unexpectedly
- **THEN** that player’s seat remains in synced state
- **AND** the seat is marked offline with a reconnect deadline 30 seconds from the disconnect
- **AND** other clients in the room observe the offline mark and deadline

#### Scenario [SC-PIECE-12]: Unexpected disconnect after start holds the seat

- **GIVEN** a tourist room in phase `playing` with seated count equal to maxSeats
- **WHEN** one seated player disconnects unexpectedly
- **THEN** that player’s seat and pieces remain in synced state
- **AND** the seat is marked offline with a reconnect deadline 30 seconds from the disconnect
- **AND** the room does not free that seat for a new joiner solely because of that disconnect (grace holds the seat)

#### Scenario [SC-PIECE-13]: Reconnect within grace restores the same seat

- **GIVEN** a seated player whose seat is held offline within the 30-second grace
- **WHEN** that client successfully reconnects before the deadline
- **THEN** the client occupies the same seat with the same tourist kind and pieces
- **AND** the seat is marked connected again
- **AND** other clients observe the seat as online

#### Scenario [SC-PIECE-14]: Grace timeout removes the seat

- **GIVEN** a seated player whose seat is held offline within the grace while phase is `playing`
- **WHEN** the 30-second grace expires without a successful reconnect
- **THEN** that player’s seat and pieces are removed from synced state
- **AND** kind and cells return to the available pools
- **AND** a subsequent joiner receives no seat while phase remains `playing`

### Requirement: Dispose when no seated players remain

After any permanent seat removal (consented leave or grace timeout), if the room has **zero** seated players left, the server SHALL close the room even if spectator clients are still connected. If at least one seated player remains, seated count is below maxSeats, and phase is `waiting`, a subsequent joiner MAY receive a seat. If phase is `countdown` or `playing`, a subsequent joiner MUST NOT receive a seat solely because capacity reopened.

#### Scenario [SC-PIECE-15]: Last seated removal closes the room with spectators present

- **GIVEN** a tourist room with exactly one seated player and at least one spectator
- **WHEN** that seated player permanently leaves (consented leave or grace timeout)
- **THEN** the room is closed
- **AND** remaining spectator clients are disconnected from that room

#### Scenario [SC-PIECE-16]: Seat connectivity is synchronized

- **GIVEN** a tourist room with at least one seated player
- **WHEN** that seat’s connected or offline-with-deadline status changes
- **THEN** every client in the room observes the updated connectivity fields for that seat in synced state

### Requirement: Finished seats occupy capacity until leave

A seat that has finished all four pieces (`game/finish`) MUST continue to occupy one slot under `maxSeats` until that seat is permanently removed by consented leave or reconnect grace timeout. While phase is `countdown` or `playing`, a joining client MUST NOT receive a seat even when seated count is strictly less than maxSeats after counting finished seats.

#### Scenario [SC-PIECE-21]: Finished seats counted in occupancy

- **GIVEN** a tourist room in phase `playing` with `maxSeats` equal to 3, two finished seats, and one non-finished seat
- **WHEN** another authenticated client joins
- **THEN** that client receives no seat (spectator)
- **AND** after one finished seat permanently leaves, a subsequent joiner still receives no seat while phase remains `playing`

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current unfinished pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. While a seated player has no pieces yet (phase `waiting` or `countdown`), no pieces for that seat MUST appear on the board. Finished pieces MUST NOT be rendered on the board (see `game/finish`). A seated client SHALL see a personal strip below the board once their four pieces exist, with exactly four slots corresponding one-to-one to that client’s four pieces by side (same kind image per slot). Until pieces exist, the seated client MUST NOT be shown moveable strip slots for that seat’s missing pieces. Each strip slot whose piece is finished MUST show a finish indicator at the top-right and MUST NOT participate in move selection. Slots for unfinished pieces keep existing move-selection behavior while applicable (`game/move`). The strip MUST remain visible for finished seats. The strip MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal strip.

#### Scenario [SC-PIECE-09]: Seated client sees own four-slot strip

- **GIVEN** the user is on the Game screen as a seated player with four pieces of one kind
- **WHEN** the game UI is shown
- **THEN** a strip below the board displays exactly four images of that user’s tourist kind
- **AND** the strip does not display other players’ tourist kinds
- **AND** the four strip slots correspond to the user’s four pieces (one per side)
- **AND** any finished piece’s slot shows a finish indicator and is not used for move selection

#### Scenario [SC-PIECE-10]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the game UI is shown
- **THEN** seated players’ unfinished pieces are visible on the board at their cells
- **AND** no personal tourist strip is shown for that user

#### Scenario [SC-PIECE-17]: No board pieces before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with one or more seated players who have tourist kinds
- **WHEN** any client views the Game board
- **THEN** no tourist pieces for those seats are shown on the board
- **AND** when phase becomes `playing` and pieces are materialized, those pieces appear at their synced cells
