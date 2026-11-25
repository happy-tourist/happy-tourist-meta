## Purpose

Effective-AI full table (human + bots = `maxSeats`) enters the same countdown → playing path as a human full table, without ready marks from bots. Bot seating — `game/ai`; pieces materialize — `game/pieces`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-START-20 | `test/MyRoom.test.ts` — "SC-START-20: AI room countdown then playing with pieces" |

## ADDED Requirements

### Requirement: AI mode full table starts countdown after bots seat

When a tourist room is in **effective AI mode** (`opponentMode: 'ai'` and `maxSeats` ≥ 2), after the human creator seat and all bot seats are assigned so that seated count equals `maxSeats`, the server MUST enter phase `countdown` and run the same five-second start countdown as a human full table, then transition to `playing` with deferred pieces materialized (`game/pieces`). Ready-to-start MUST NOT be required for that AI full table (bots are not required to mark `ready`).

#### Scenario [SC-START-20]: AI room reaches countdown when bots fill seats

- **GIVEN** an authenticated client creates a tourist room with opponentMode `ai` and maxSeats 3
- **WHEN** the human seat and two bot seats are assigned
- **THEN** the phase becomes `countdown`
- **AND** after the countdown completes the phase is `playing`
- **AND** each of the three seats has pieces per `game/pieces`
