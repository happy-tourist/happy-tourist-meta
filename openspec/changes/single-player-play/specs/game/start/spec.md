# Delta: game/start

Связано с main `openspec/specs/game/start/spec.md`. Этот change: полный стол при `maxSeats = 1` стартует countdown. Ready-ветка (≥2 seated, underfill) без изменений. Экономика одиночной — `game/move` / `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-START-03 | covered-by-reuse (full table ≥2) |
| SC-START-14 | added — server mocha (maxSeats=1 → countdown) |

## MODIFIED Requirements

### Requirement: Auto countdown when seats are full

When the number of seated players reaches the room’s maxSeats while the phase is `waiting`, the server SHALL immediately enter phase `countdown` and run a five-second start countdown (display values 5, 4, 3, 2, 1) that is authoritative for all clients. When the countdown reaches zero, the phase MUST become `playing`. This MUST apply when **maxSeats is 1** and the sole seated player has joined — the table is full with one seat; the server MUST NOT require a second seated player before starting countdown.

#### Scenario [SC-START-03]: Full table starts countdown

- **GIVEN** a tourist room in phase `waiting` with maxSeats equal to 3 and two seated players
- **WHEN** a third authenticated client joins and receives a seat
- **THEN** the phase becomes `countdown`
- **AND** every client in the room observes the shared countdown progressing through 5…1
- **AND** after the countdown completes, the phase is `playing`

#### Scenario [SC-START-14]: Single-seat full table starts countdown

- **GIVEN** an authenticated client creates a tourist room with maxSeats equal to 1 and joins as the sole seated player while phase is `waiting`
- **WHEN** that seat is assigned
- **THEN** the phase becomes `countdown`
- **AND** after the countdown completes, the phase is `playing`
- **AND** the ready-to-start control is not required for that start
