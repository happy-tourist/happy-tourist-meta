## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-06 | covered-by-reuse (server mocha + client) |
| SC-FINISH-07 | covered (server mocha — no seat after leave in playing) |
| SC-FINISH-08 | covered-by-reuse (server mocha) |

Related: seating gate — `game/pieces` (new seats only in `waiting`).

## MODIFIED Requirements

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
