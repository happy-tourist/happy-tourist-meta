## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-09 | pending (client UX — strip finish chrome) |
| SC-PIECE-10 | covered-by-reuse (client spectator no strip) |
| SC-PIECE-21 | pending (server mocha) |

Related: finish semantics — `game/finish`.

## MODIFIED Requirements

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current unfinished pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. Finished pieces MUST NOT be rendered on the board (see `game/finish`). A seated client SHALL see a personal strip below the board with exactly four slots corresponding one-to-one to that client’s four pieces by side (same kind image per slot). Each strip slot whose piece is finished MUST show a finish indicator at the top-right and MUST NOT participate in move selection. Slots for unfinished pieces keep existing move-selection behavior while applicable (`game/move`). The strip MUST remain visible for finished seats. The strip MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal strip.

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

## ADDED Requirements

### Requirement: Finished seats occupy capacity until leave

A seat that has finished all four pieces (`game/finish`) MUST continue to occupy one slot under `maxSeats` until that seat is permanently removed by consented leave or reconnect grace timeout. Mid-game seating while phase is `playing` MUST still follow free-slot rules: a joiner receives a seat and four pieces from scratch only when seated count is strictly less than `maxSeats`, counting finished seats as occupied.

#### Scenario [SC-PIECE-21]: Finished seats counted in occupancy

- **GIVEN** a tourist room with `maxSeats` equal to 3, two finished seats, and one non-finished seat
- **WHEN** another authenticated client joins
- **THEN** that client receives no seat (spectator)
- **AND** after one finished seat permanently leaves, a subsequent joiner MAY receive a seat with four pieces on free start cells
