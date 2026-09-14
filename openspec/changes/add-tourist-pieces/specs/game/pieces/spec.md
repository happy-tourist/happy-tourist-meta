## Purpose

Рассадка в room `tourist`: до четырёх seated-игроков; у каждого уникальный вид туриста и **четыре одинаковые** фигурки — по одной на сторонах N/E/S/W на свободных стартовых клетках; личная полоса из четырёх слотов 1:1 с полевыми (статусы later).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-01 | covered (server mocha) |
| SC-PIECE-02 | covered (server mocha) |
| SC-PIECE-03 | covered (server mocha) |
| SC-PIECE-04 | covered (server mocha) |
| SC-PIECE-05 | covered (server mocha) |
| SC-PIECE-06 | covered (server mocha) |
| SC-PIECE-07 | covered (server mocha) |
| SC-PIECE-08 | covered (server mocha) |
| SC-PIECE-09 | covered (client GamePage strip ×4) |
| SC-PIECE-10 | covered (client GamePage spectator no strip) |

## ADDED Requirements

### Requirement: Four pieces per seated player

Until the tourist room has started, when an authenticated client joins and fewer than four seated players exist, the server SHALL assign that client a unique tourist kind from `1`…`4` not used by any current seat, and SHALL place exactly four pieces for that client — one on each board side `N`, `E`, `S`, and `W`. For each side the start cell MUST be chosen uniformly at random from the start cells of that side that are not already occupied by any piece in the room. All four pieces of the same player MUST use that player’s tourist kind. The assignment MUST be synchronized to all clients in the room.

#### Scenario [SC-PIECE-01]: First join receives four pieces on all sides

- **GIVEN** a tourist room that has not started and has no seated players
- **WHEN** an authenticated client joins the room
- **THEN** the client is assigned exactly one tourist kind from `1`…`4`
- **AND** the client has exactly four pieces, one on each side `N`, `E`, `S`, and `W`
- **AND** each piece’s cell is one of the four start cells of its side
- **AND** every client in the room observes those pieces in synced state

#### Scenario [SC-PIECE-02]: Tourist kinds stay unique among players

- **GIVEN** a tourist room that has not started and already has one or more seated players
- **WHEN** another authenticated client joins and receives a seat
- **THEN** the new tourist kind is not equal to any currently seated player’s kind
- **AND** all four of the new player’s pieces use that same new kind

#### Scenario [SC-PIECE-03]: Start cells lie on the assigned side and stay free

- **GIVEN** a tourist room that has not started
- **WHEN** a client is assigned pieces on the four sides
- **THEN** each piece’s row and column match a start cell of that piece’s side on the agreed tourist layout
- **AND** no two pieces in the room share the same row and column

#### Scenario [SC-PIECE-04]: Second player uses remaining cells on each side

- **GIVEN** a tourist room that has not started and already has one seated player with four pieces
- **WHEN** a second authenticated client joins and receives a seat
- **THEN** the second player also has one piece on each side `N`, `E`, `S`, and `W`
- **AND** none of the second player’s cells equal any cell already occupied by the first player

### Requirement: Spectators and hard stop after four seated players

The room MUST allow clients beyond four connections. After four seated players have been assigned, the room SHALL be considered started. While the room is started, joining clients MUST NOT receive a seat or pieces.

#### Scenario [SC-PIECE-05]: Fourth seated player starts the room

- **GIVEN** a tourist room that has not started and has three seated players
- **WHEN** a fourth authenticated client joins and receives a seat with four pieces
- **THEN** the room is marked started
- **AND** subsequent joining clients receive no seat and no pieces

#### Scenario [SC-PIECE-06]: Fifth connection is a spectator

- **GIVEN** a tourist room that already has four seated players (and is therefore started)
- **WHEN** another authenticated client joins
- **THEN** that client receives no tourist kind and no pieces
- **AND** existing seats and pieces remain unchanged

### Requirement: Leave before and after start

If a seated player leaves before the room has started, the server SHALL remove that player’s seat and all four pieces and return the tourist kind and occupied cells to the available pools. If a seated player leaves after the room has started, the server SHALL remove that player’s pieces from synced state and MUST NOT assign seats to new joiners.

#### Scenario [SC-PIECE-07]: Leave before start frees kind and cells

- **GIVEN** a tourist room that has not started and has at least one seated player
- **WHEN** that seated player leaves the room
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner MAY receive the freed tourist kind and MAY occupy formerly taken start cells

#### Scenario [SC-PIECE-08]: Leave after start does not reopen seating

- **GIVEN** a started tourist room with four seated players
- **WHEN** one seated player leaves
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner still receives no seat

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. A seated client SHALL see a personal strip below the board with exactly four slots corresponding one-to-one to that client’s four board pieces (same kind image per slot; status chrome deferred). The strip MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal strip.

#### Scenario [SC-PIECE-09]: Seated client sees own four-slot strip

- **GIVEN** the user is on the Game screen as a seated player with four pieces of one kind
- **WHEN** the game UI is shown
- **THEN** a strip below the board displays exactly four images of that user’s tourist kind
- **AND** the strip does not display other players’ tourist kinds
- **AND** the four strip slots correspond to the user’s four board pieces (one per side)

#### Scenario [SC-PIECE-10]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the game UI is shown
- **THEN** seated players’ pieces are visible on the board at their cells
- **AND** no personal tourist strip is shown for that user
