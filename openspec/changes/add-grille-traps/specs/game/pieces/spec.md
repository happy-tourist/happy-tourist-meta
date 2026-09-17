## Purpose

Delta этого change: sync `trapped`, all-jail placement на старты стороны. Рассадка/materialize — main `game/pieces`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-24 | done (server mocha — trapped sync) |
| SC-PIECE-25 | done (server mocha — all-jail one per side) |
| SC-PIECE-26 | done (server mocha — occupancy while trapped) |
| SC-PIECE-27 | done (client UX — trapped piece still on board) |

Related: trap/rescue/all-jail — `game/move`; grille clear — `game/board`.

## ADDED Requirements

### Requirement: Trapped flag is synchronized on pieces

When a piece becomes trapped or freed, the server SHALL synchronize that trapped state to all clients in the room. Finished pieces MUST NOT be trapped. A trapped piece MUST remain unfinished and MUST keep its row and column until freed, returned-from elsewhere, or all-jail reset.

#### Scenario [SC-PIECE-24]: Clients observe trapped state

- **GIVEN** phase is `playing` and a piece just became trapped after landing on a grille
- **WHEN** synced state updates
- **THEN** every client observes that piece as trapped at its landing cell

#### Scenario [SC-PIECE-27]: Trapped tourist remains visible on the board

- **GIVEN** a piece is trapped
- **WHEN** clients render the board
- **THEN** that piece remains visible on its cell under/with the revealed grille presentation
- **AND** the piece is not treated as finished or off-board

### Requirement: Trapped pieces still occupy their cell

For move validation occupancy, a trapped unfinished piece MUST block its cell the same way as a free unfinished piece. Other pieces MUST NOT legally land on that cell while it remains occupied.

#### Scenario [SC-PIECE-26]: Occupancy blocks landing on trapped cell

- **GIVEN** a trapped unfinished piece occupies a cell and another unfinished piece is adjacent with steps available on its seat’s turn
- **WHEN** that other piece attempts to move onto the trapped piece’s cell
- **THEN** the server rejects the move

### Requirement: All-jail places one piece per side on free starts

On all-jail reset for a seat, each of that seat’s four pieces MUST be placed on a start cell of its own side identity (`N`/`E`/`S`/`W`), chosen uniformly at random among start cells of that side that are not occupied by any unfinished piece after clearing that seat’s previous cells. Pieces MUST be free (not trapped) after placement.

#### Scenario [SC-PIECE-25]: Reset uses each side’s free start cells

- **GIVEN** a seated player triggers all-jail reset
- **WHEN** the server places the four pieces
- **THEN** the `N` piece is on a free North start cell, `E` on East, `S` on South, and `W` on West
- **AND** none of the four share a cell with another unfinished piece
- **AND** none of the four remain trapped
