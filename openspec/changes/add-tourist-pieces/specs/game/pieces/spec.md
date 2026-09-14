## Purpose

Рассадка до четырёх фигурок-туристов в room `tourist`: уникальный вид и сторона, случайная стартовая клетка, старт при четвёртом seated-игроке, гости без личной полосы под доской.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-01 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-02 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-03 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-04 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-05 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-06 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-07 | covered (server mocha `test/MyRoom.test.ts`) |
| SC-PIECE-08 | covered (client `GamePage` + `stores/game`; lint/typecheck) |
| SC-PIECE-09 | covered (client `GamePage` + `stores/game`; lint/typecheck) |

## ADDED Requirements

### Requirement: Seat assignment before start

Until the tourist room has started, when an authenticated client joins and fewer than four seated players exist, the server SHALL assign that client a seat with a tourist kind and a board side chosen uniformly at random from the kinds and sides not already assigned, and a start cell chosen uniformly at random from the four start cells on that side. Assigned kinds and sides MUST remain unique among current seats. The assignment MUST be synchronized to all clients in the room.

#### Scenario [SC-PIECE-01]: First join receives a seat

- **GIVEN** a tourist room that has not started and has no seated players
- **WHEN** an authenticated client joins the room
- **THEN** the client is assigned exactly one tourist kind from `1`…`4` and exactly one board side
- **AND** the assigned start cell is one of the four start cells on that side
- **AND** every client in the room observes that seat in synced state

#### Scenario [SC-PIECE-02]: Kinds and sides stay unique

- **GIVEN** a tourist room that has not started and already has one or more seated players
- **WHEN** another authenticated client joins and receives a seat
- **THEN** the new tourist kind is not equal to any currently seated kind
- **AND** the new board side is not equal to any currently seated side

#### Scenario [SC-PIECE-03]: Start cell lies on the assigned side

- **GIVEN** a tourist room that has not started
- **WHEN** a client is assigned a seat on a given board side
- **THEN** the seat’s row and column match one of the four start cells belonging to that side on the agreed tourist layout

### Requirement: Spectators and hard stop after four seats

The room MUST allow clients beyond four connections. After four seats have been assigned, the room SHALL be considered started. While the room is started, joining clients MUST NOT receive a seat. Before start, a joining client that cannot receive a seat because four seats already exist MUST be treated as a spectator (no seat).

#### Scenario [SC-PIECE-04]: Fifth connection is a spectator before explicit start edge

- **GIVEN** a tourist room that already has four seated players (and is therefore started)
- **WHEN** another authenticated client joins
- **THEN** that client receives no tourist kind, side, or start cell
- **AND** existing seats remain unchanged

#### Scenario [SC-PIECE-05]: Fourth seat starts the room

- **GIVEN** a tourist room that has not started and has three seated players
- **WHEN** a fourth authenticated client joins and receives a seat
- **THEN** the room is marked started
- **AND** subsequent joining clients receive no seats

### Requirement: Leave before and after start

If a seated player leaves before the room has started, the server SHALL remove that seat and return its tourist kind and side to the available pools so a later joiner can be seated. If a seated player leaves after the room has started, the server SHALL remove that player’s piece from synced state and MUST NOT assign seats to new joiners.

#### Scenario [SC-PIECE-06]: Leave before start frees pools

- **GIVEN** a tourist room that has not started and has at least one seated player
- **WHEN** that seated player leaves the room
- **THEN** the seat is removed from synced state
- **AND** a subsequent joiner MAY receive the freed tourist kind and/or side

#### Scenario [SC-PIECE-07]: Leave after start does not reopen seating

- **GIVEN** a started tourist room with four seats
- **WHEN** one seated player leaves
- **THEN** that player’s piece is removed from synced state
- **AND** a subsequent joiner still receives no seat

### Requirement: Board pieces and personal tourist strip

All clients in the room SHALL see the current seated pieces on the Game board at their synced start cells, using the tourist images for kinds `1`…`4`. A seated client SHALL see a personal strip below the board showing only their own tourist image. A spectator MUST NOT be shown that personal strip.

#### Scenario [SC-PIECE-08]: Seated client sees own strip

- **GIVEN** the user is on the Game screen as a seated player with an assigned tourist kind
- **WHEN** the game UI is shown
- **THEN** a strip below the board displays the image for that user’s tourist kind
- **AND** the strip does not display other players’ tourist images as the user’s own

#### Scenario [SC-PIECE-09]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the game UI is shown
- **THEN** the seated pieces are visible on the board at their start cells
- **AND** no personal tourist strip is shown for that user
