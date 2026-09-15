## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-START-01 | covered-by-reuse (server mocha) |
| SC-START-02 | pending (server mocha — playing materializes pieces) |
| SC-START-13 | pending (server mocha) |

Related: deferred pieces — `game/pieces`; turn timers begin in playing — `game/move`.

## ADDED Requirements

### Requirement: Playing transition places deferred pieces

When the five-second start countdown completes and the synchronized phase becomes `playing`, the server MUST materialize pieces for every seated player that still has none, per `game/pieces`, before clients may successfully complete board moves. Seats that joined already during `playing` keep their existing pieces.

#### Scenario [SC-START-13]: Countdown completion spawns pieces then unlocks play

- **GIVEN** a tourist room in phase `countdown` with seated players who have tourist kinds but no pieces
- **WHEN** the countdown completes and phase becomes `playing`
- **THEN** each of those seats has four pieces in synced state
- **AND** a legal move from the current-turn seated player MAY be accepted per `game/move`

## MODIFIED Requirements

### Requirement: Synced start phase

The tourist room SHALL expose a synchronized start phase among `waiting`, `countdown`, and `playing`. A newly created room MUST begin in `waiting`. While the phase is `waiting` or `countdown`, seated clients MUST NOT successfully complete a board move (see `game/move`), and seats MUST NOT yet have board pieces (see `game/pieces`). While the phase is `playing`, move rules of `game/move` apply and pieces exist for seated players per `game/pieces`. The phase transition into `playing` MUST occur only after a completed countdown of this capability and MUST include materializing any deferred pieces.

#### Scenario [SC-START-01]: New room is waiting

- **GIVEN** an authenticated client creates a tourist room with a valid maxSeats
- **WHEN** the room becomes available
- **THEN** every client in the room observes start phase `waiting`
- **AND** board moves submitted in that phase are rejected without changing piece positions

#### Scenario [SC-START-02]: Playing unlocks moves

- **GIVEN** a tourist room whose countdown has just completed
- **WHEN** the synchronized phase becomes `playing`
- **THEN** every client in the room observes phase `playing`
- **AND** seated players who waited without pieces now have four pieces each
- **AND** a legal move from the current-turn seated player MAY be accepted per `game/move`
