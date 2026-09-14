# game/pieces Specification

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
| SC-PIECE-07 | covered (server mocha — consented leave before start) |
| SC-PIECE-08 | covered (server mocha — consented leave after start) |
| SC-PIECE-09 | covered (client GamePage strip ×4) |
| SC-PIECE-10 | covered (client GamePage spectator no strip) |
| SC-PIECE-11 | covered (server mocha — unexpected hold before start) |
| SC-PIECE-12 | covered (server mocha — unexpected hold after start) |
| SC-PIECE-13 | covered (server mocha — reconnect within grace) |
| SC-PIECE-14 | covered (server mocha — grace timeout removes seat) |
| SC-PIECE-15 | covered (server mocha — zero seated disposes room) |
| SC-PIECE-16 | covered (server mocha — sync offline + deadline) |
| SC-PIECE-17 | covered (client — localStorage token restores seat after browsing session end) |
| SC-PIECE-18 | covered (client — missing/invalid token = fresh joinById) |

## Requirements

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

A **consented** leave by a seated player (intentional exit from the match UI back to the lobby) SHALL permanently remove that player’s seat and all four pieces. If the room has not started, the tourist kind and occupied cells MUST return to the available pools. If the room has started, the server MUST NOT assign seats to new joiners after that removal. An **unexpected** disconnect is governed by the reconnect grace requirement and MUST NOT be treated as an immediate consented leave.

#### Scenario [SC-PIECE-07]: Leave before start frees kind and cells

- **GIVEN** a tourist room that has not started and has at least one seated player
- **WHEN** that seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner MAY receive the freed tourist kind and MAY occupy formerly taken start cells

#### Scenario [SC-PIECE-08]: Leave after start does not reopen seating

- **GIVEN** a started tourist room with four seated players
- **WHEN** one seated player performs a consented leave (intentional exit to the lobby)
- **THEN** that player’s four pieces are removed from synced state
- **AND** a subsequent joiner still receives no seat

### Requirement: Unexpected disconnect grace and reconnect

When a seated player disconnects unexpectedly (for example page reload or network drop), the server MUST keep that seat and all four pieces in synced state for a grace period of exactly **30 seconds** and MUST allow the same client session to reconnect into that seat within the grace. During the grace the seat MUST be marked offline with a synchronized reconnect deadline visible to all clients in the room. The same grace policy applies whether or not the room has started.

#### Scenario [SC-PIECE-11]: Unexpected disconnect before start holds the seat

- **GIVEN** a tourist room that has not started and has at least one seated player who is marked connected
- **WHEN** that seated player disconnects unexpectedly
- **THEN** that player’s seat and four pieces remain in synced state
- **AND** the seat is marked offline with a reconnect deadline 30 seconds from the disconnect
- **AND** other clients in the room observe the offline mark and deadline

#### Scenario [SC-PIECE-12]: Unexpected disconnect after start holds the seat

- **GIVEN** a started tourist room with four seated players
- **WHEN** one seated player disconnects unexpectedly
- **THEN** that player’s seat and four pieces remain in synced state
- **AND** the seat is marked offline with a reconnect deadline 30 seconds from the disconnect
- **AND** the room remains started and does not assign a seat to a new joiner solely because of that disconnect

#### Scenario [SC-PIECE-13]: Reconnect within grace restores the same seat

- **GIVEN** a seated player whose seat is held offline within the 30-second grace
- **WHEN** that client successfully reconnects before the deadline
- **THEN** the client occupies the same seat with the same tourist kind and pieces
- **AND** the seat is marked connected again
- **AND** other clients observe the seat as online

#### Scenario [SC-PIECE-14]: Grace timeout removes the seat

- **GIVEN** a seated player whose seat is held offline within the grace
- **WHEN** the 30-second grace expires without a successful reconnect
- **THEN** that player’s seat and four pieces are removed from synced state
- **AND** if the room has not started, kind and cells return to the available pools
- **AND** if the room has started, seating remains closed to new joiners

### Requirement: Dispose when no seated players remain

After any permanent seat removal (consented leave or grace timeout), if the room has **zero** seated players left, the server SHALL close the room even if spectator clients are still connected. If at least one seated player remains, the room MUST continue and MUST NOT reopen seating solely because another seated player left after start.

#### Scenario [SC-PIECE-15]: Last seated removal closes the room with spectators present

- **GIVEN** a tourist room with exactly one seated player and at least one spectator
- **WHEN** that seated player permanently leaves (consented leave or grace timeout)
- **THEN** the room is closed
- **AND** remaining spectator clients are disconnected from that room

#### Scenario [SC-PIECE-16]: Seat connectivity is synchronized

- **GIVEN** a tourist room with at least one seated player
- **WHEN** that seat’s connected or offline-with-deadline status changes
- **THEN** every client in the room observes the updated connectivity fields for that seat in synced state

### Requirement: Client persists tourist reconnection credential

After a seated player successfully enters a tourist room, the client MUST persist that room’s Colyseus reconnection token and room id in **`localStorage`** (not session-only storage) so that returning to the Game screen after a full browser or tab restart within the server grace can restore the same seat. The client MUST clear that credential on a consented leave. A second browser tab sharing the same storage MAY use the token to reconnect and take over the seat. The lobby listing MUST NOT use this tourist credential store.

#### Scenario [SC-PIECE-17]: Token survives browser restart within grace

- **GIVEN** a seated player whose client has stored a tourist reconnection token for the current room in localStorage
- **WHEN** the browsing session ends (tab or browser closed) and the user reopens the Game for that room before the 30-second grace expires
- **THEN** the client reconnects using the stored token
- **AND** the player occupies the same seat with the same tourist kind and pieces
- **AND** the seat is marked connected again

#### Scenario [SC-PIECE-18]: Missing or invalid token is a fresh join

- **GIVEN** the user opens the Game for a tourist room without a valid stored reconnection token for that room (or reconnect with the stored token fails)
- **WHEN** the client joins that room
- **THEN** the join is treated as a fresh joinById (not a seat reclaim by user id)
- **AND** if the room has not started and a free seat slot exists, the joiner MAY receive a new seat with pieces
- **AND** if the room has started, the joiner receives no seat (spectator only)
- **AND** any offline grace seat held for another session remains until reconnect or timeout independently

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
