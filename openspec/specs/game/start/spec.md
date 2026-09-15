# game/start Specification

## Purpose

Фазы старта партии в room `tourist`: ожидание игроков, подтверждение готовности при недоборе, общий countdown 5…1 и переход в playing, после которого разрешены ходы. Связано с `game/pieces` (ёмкость seats), `game/say` (preset готовности), `game/move` (gate ходов), `lobby/rooms` (create maxSeats).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-START-01 | covered (server mocha) |
| SC-START-02 | covered (server mocha — with SC-START-03) |
| SC-START-03 | covered (server mocha) |
| SC-START-04 | covered (server mocha) |
| SC-START-05 | covered (server mocha) |
| SC-START-06 | covered (server mocha) |
| SC-START-07 | covered (server mocha) |
| SC-START-08 | covered (client UX) |
| SC-START-09 | covered (client UX) |
| SC-START-10 | covered (client UX) |
| SC-START-11 | covered (server mocha) |
| SC-START-12 | covered (server mocha) |

## Requirements

### Requirement: Synced start phase

The tourist room SHALL expose a synchronized start phase among `waiting`, `countdown`, and `playing`. A newly created room MUST begin in `waiting`. While the phase is `waiting` or `countdown`, seated clients MUST NOT successfully complete a board move (see `game/move`). While the phase is `playing`, move rules of `game/move` apply. The phase transition into `playing` MUST occur only after a completed countdown of this capability.

#### Scenario [SC-START-01]: New room is waiting

- **GIVEN** an authenticated client creates a tourist room with a valid maxSeats
- **WHEN** the room becomes available
- **THEN** every client in the room observes start phase `waiting`
- **AND** board moves submitted in that phase are rejected without changing piece positions

#### Scenario [SC-START-02]: Playing unlocks moves

- **GIVEN** a tourist room whose countdown has just completed
- **WHEN** the synchronized phase becomes `playing`
- **THEN** every client in the room observes phase `playing`
- **AND** a legal move from the current-turn seated player MAY be accepted per `game/move`

### Requirement: Auto countdown when seats are full

When the number of seated players reaches the room’s maxSeats while the phase is `waiting`, the server SHALL immediately enter phase `countdown` and run a five-second start countdown (display values 5, 4, 3, 2, 1) that is authoritative for all clients. When the countdown reaches zero, the phase MUST become `playing`.

#### Scenario [SC-START-03]: Full table starts countdown

- **GIVEN** a tourist room in phase `waiting` with maxSeats equal to 3 and two seated players
- **WHEN** a third authenticated client joins and receives a seat
- **THEN** the phase becomes `countdown`
- **AND** every client in the room observes the shared countdown progressing through 5…1
- **AND** after the countdown completes, the phase is `playing`

### Requirement: Ready-to-start when underfilled

While the phase is `waiting` and the seated count is at least 2 and strictly less than maxSeats, each seated connected player who has not yet marked ready MUST be able to submit a one-shot ready intent. A successful ready MUST mark that seat ready in synced state, MUST remove further ready affordance for that seat, and MUST broadcast the readiness say preset per `game/say`. When every currently seated player is marked ready (and seated count ≥ 2), the server SHALL enter `countdown` as for a full table. A room with exactly one seated player MUST NOT offer or accept ready. Ready marks MUST NOT be cleared when another seated player leaves. Spectators MUST NOT submit ready.

#### Scenario [SC-START-04]: Single seated player has no ready

- **GIVEN** a tourist room in phase `waiting` with maxSeats greater than 1 and exactly one seated player
- **WHEN** that player views the game screen
- **THEN** the ready-to-start control is not available
- **AND** a ready intent from that seat is rejected without changing phase

#### Scenario [SC-START-05]: All seated ready starts countdown

- **GIVEN** a tourist room in phase `waiting` with maxSeats 4 and three seated connected players, none ready
- **WHEN** each of those three players successfully submits ready
- **THEN** after the last of them becomes ready the phase becomes `countdown`
- **AND** every client observes countdown 5…1 then phase `playing`

#### Scenario [SC-START-06]: Ready is one-shot and shows say bubble

- **GIVEN** a tourist room in phase `waiting` with at least two seated players and fewer than maxSeats seats filled
- **WHEN** a seated connected player who is not yet ready submits ready
- **THEN** that seat is marked ready
- **AND** every client observes a say event with the readiness preset for that seat
- **AND** that player no longer has a ready-to-start control
- **AND** a second ready intent from that seat is rejected without a second broadcast

#### Scenario [SC-START-07]: Leave does not clear others’ ready

- **GIVEN** a tourist room in phase `waiting` with three seated players where two are already ready
- **WHEN** one ready seated player performs a consented leave
- **THEN** the remaining ready seats stay marked ready
- **AND** the phase remains `waiting` until the remaining seated players all become ready or seats fill to maxSeats

### Requirement: Countdown overlay for all clients

While the phase is `countdown`, every client in the room (seated and spectators) SHALL show a full-screen waiting overlay with the product text that the game will start soon and the current countdown second (5…1). The overlay MUST dismiss when the phase becomes `playing`. Disconnect or leave during countdown MUST follow existing reconnect/leave rules (`game/pieces`); the countdown MUST NOT be cancelled solely because a seated player drops or leaves; remaining seated players continue into `playing` when the countdown completes.

#### Scenario [SC-START-08]: Everyone sees countdown overlay

- **GIVEN** a tourist room that has entered phase `countdown` with at least one spectator and one seated player
- **WHEN** those clients view the game screen
- **THEN** each sees the full-screen start overlay with the shared countdown value
- **AND** when the phase becomes `playing`, the overlay is no longer shown

#### Scenario [SC-START-09]: Ready control beside say affordance

- **GIVEN** a seated connected player eligible for ready (phase `waiting`, seated count ≥ 2, under maxSeats, not yet ready)
- **WHEN** that player views their own presence controls
- **THEN** a ready-to-start control is available alongside the say affordance
- **AND** spectators do not see that control for themselves

#### Scenario [SC-START-10]: Board interaction locked before playing

- **GIVEN** a seated player whose turn it would be under turn-order rules while phase is `waiting` or `countdown`
- **WHEN** that player activates the board or strip as if to move
- **THEN** the client does not submit a successful move
- **AND** move chrome for committing a move is not available as a playable action until phase `playing`

### Requirement: Countdown survives leave and reconnect grace

If a seated player disconnects unexpectedly during `countdown`, the seat MUST remain held for the usual 30-second reconnect grace with the offline progress indicator (`game/presence` / `game/pieces`), including when that seat holds the current turn. If a seated player consented-leaves during `countdown`, the seat is removed as usual and the countdown continues; when it completes, remaining seated players enter `playing`.

#### Scenario [SC-START-11]: Drop during countdown keeps grace and countdown

- **GIVEN** a tourist room in phase `countdown` with at least two seated players
- **WHEN** one seated player disconnects unexpectedly
- **THEN** that seat remains held offline with a 30-second reconnect deadline
- **AND** the phase remains `countdown` and still reaches `playing` after the countdown completes

#### Scenario [SC-START-12]: Consented leave during countdown continues

- **GIVEN** a tourist room in phase `countdown` with at least two seated players
- **WHEN** one seated player performs a consented leave
- **THEN** that player’s seat and pieces are removed
- **AND** the phase remains `countdown` until completion
- **AND** after completion the phase is `playing` for the remaining seated players
