## Purpose

Join/spectator gate and bot occupancy for **effective AI** rooms. Dispose-when-human-gone override — `game/leave` (not here). Bot seat shape / `isBot` — `game/ai`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-60 | `test/MyRoom.test.ts` — "SC-PIECE-60: second human cannot join AI room" |
| SC-PIECE-61 | `test/MyRoom.test.ts` — "SC-PIECE-61: creator reconnect restores human seat with bots" |

## ADDED Requirements

### Requirement: AI opponent rooms reject additional human join and spectators

While a tourist room is in **effective AI mode** (`opponentMode: 'ai'` and `maxSeats` ≥ 2), the server MUST NOT assign a seat to any client other than the creating human (including reconnect of that human within grace). Additional authenticated clients MUST NOT remain as spectators in that room: join attempts by other clients MUST be rejected (throw / hard deny — not the silent spectator path used for full people tables). Bot seats still count toward `maxSeats` occupancy for start and turn rules and MUST sync `isBot: true`. Coerced solo (`maxSeats` 1) keeps today’s people seating/spectator rules.

#### Scenario [SC-PIECE-60]: Second human cannot join AI room as seat or spectator

- **GIVEN** an effective-AI tourist room already seating the creator human and bot seats up to maxSeats
- **WHEN** a different authenticated client attempts to join that room by id
- **THEN** the join is rejected without granting a seat
- **AND** the joiner does not remain as a spectator in the room

#### Scenario [SC-PIECE-61]: Creator reconnect restores human seat

- **GIVEN** an effective-AI tourist room where the creator human seat is offline within reconnect grace
- **WHEN** that same client reconnects within grace
- **THEN** the human seat is restored as today
- **AND** bot seats remain occupied with isBot true
