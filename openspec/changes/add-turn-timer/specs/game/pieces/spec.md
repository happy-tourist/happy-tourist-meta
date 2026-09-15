## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-01 | pending (server mocha — kind without pieces pre-playing) |
| SC-PIECE-02 | covered-by-reuse (server mocha — unique kinds) |
| SC-PIECE-03 | pending (server mocha — spawn on playing) |
| SC-PIECE-04 | pending (server mocha — second seat cells after spawn) |
| SC-PIECE-05 | pending (server mocha — full table seat without pieces until playing) |
| SC-PIECE-06 | covered-by-reuse (server mocha — spectator) |
| SC-PIECE-09 | covered-by-reuse (client UX) |
| SC-PIECE-10 | covered-by-reuse (client UX) |
| SC-PIECE-17 | pending (client UX — empty board before playing) |
| SC-PIECE-18 | pending (server mocha — materialize on playing) |
| SC-PIECE-19 | pending (server mocha — mid-playing join still gets pieces) |

Related: phase transition — `game/start`; turn timers — `game/move`.

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

Until the tourist room has no free seat under its maxSeats, when an authenticated client joins and fewer seated players exist than maxSeats, the server SHALL assign that client a unique tourist kind from `1`…`4` not used by any current seat. If the room start phase is `playing`, the server SHALL also place exactly four pieces for that client — one on each board side `N`, `E`, `S`, and `W` — choosing start cells uniformly at random from the start cells of that side that are not already occupied. If the phase is `waiting` or `countdown`, the server MUST assign the seat and tourist kind but MUST NOT place pieces yet; pieces for those seats are created when playing begins per the materialize requirement. All four pieces of the same player MUST use that player’s tourist kind. The assignment MUST be synchronized to all clients in the room.

#### Scenario [SC-PIECE-01]: First join receives four pieces on all sides

- **GIVEN** a tourist room in phase `waiting` that has no seated players and has free seat capacity
- **WHEN** an authenticated client joins the room
- **THEN** the client is assigned exactly one tourist kind from `1`…`4`
- **AND** the client has a seat with no pieces yet in synced state
- **AND** when the phase later becomes `playing`, that seat has exactly four pieces, one on each side `N`, `E`, `S`, and `W`
- **AND** every client in the room observes those pieces after materialize

#### Scenario [SC-PIECE-02]: Tourist kinds stay unique among players

- **GIVEN** a tourist room that already has one or more seated players and still has free seat capacity
- **WHEN** another authenticated client joins and receives a seat
- **THEN** the new tourist kind is not equal to any currently seated player’s kind
- **AND** when pieces exist for that seat, all four of the new player’s pieces use that same new kind

#### Scenario [SC-PIECE-03]: Start cells lie on the assigned side and stay free

- **GIVEN** a tourist room with free seat capacity that is placing pieces for a seat (join during `playing` or materialize at playing start)
- **WHEN** a client is assigned pieces on the four sides
- **THEN** each piece’s row and column match a start cell of that piece’s side on the agreed tourist layout
- **AND** no two pieces in the room share the same row and column

#### Scenario [SC-PIECE-04]: Second player uses remaining cells on each side

- **GIVEN** a tourist room in phase `playing` that already has one seated player with four pieces and still has free seat capacity
- **WHEN** a second authenticated client joins and receives a seat
- **THEN** the second player also has one piece on each side `N`, `E`, `S`, and `W`
- **AND** none of the second player’s cells equal any cell already occupied by the first player

### Requirement: Spectators and hard stop after four seated players

The room MUST allow clients beyond maxSeats connections. The room’s maxSeats MUST be one of 2, 3, or 4 as set at create. While the number of seated players is strictly less than maxSeats, joining authenticated clients MUST receive a seat and a tourist kind (and pieces immediately only when phase is `playing`; otherwise pieces wait until playing begins). When seated count equals maxSeats, joining clients MUST NOT receive a seat, kind, or pieces (spectators / guests). Filling the last free seat while phase is `waiting` triggers start countdown per `game/start` (this requirement does not itself define countdown). The legacy meaning of `started` as a permanent seating lock after exactly four seats MUST NOT apply.

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

#### Scenario [SC-PIECE-19]: Mid-game join takes a free seat

- **GIVEN** a tourist room in phase `playing` with maxSeats 4 and two seated players
- **WHEN** another authenticated client joins
- **THEN** that client receives a seat and four pieces on free start cells
- **AND** every client observes the new seat and pieces

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
