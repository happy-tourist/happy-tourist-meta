## Purpose

Server dispose for **effective AI** rooms when the sole human permanently leaves (any room phase). Client leave chrome / confirm UX stays as in main `game/leave`; people-table dispose-when-zero-seats stays in `game/pieces` except as overridden here.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-20 | `test/MyRoom.test.ts` — "SC-LEAVE-20: consented leave disposes AI room in playing" |
| SC-LEAVE-21 | `test/MyRoom.test.ts` — "SC-LEAVE-21: human grace expiry disposes AI room" |
| SC-LEAVE-22 | `test/MyRoom.test.ts` — "SC-LEAVE-22: consented leave in countdown disposes AI room" |

Related: bot seats / no grace — `game/ai`; seating capacity — `game/pieces`.

## ADDED Requirements

### Requirement: Human leave disposes AI opponent room immediately

When the sole human seated player in an **effective AI mode** tourist room (`opponentMode: 'ai'` and `maxSeats` ≥ 2) permanently leaves in **any** room phase (`waiting` / `countdown` / `playing`) — consented leave or reconnect grace expiry — the server MUST dispose the room immediately even if bot seats remain. This overrides the usual “dispose only when zero seats” rule for people tables (`game/pieces`): bot seats alone MUST NOT keep an effective-AI room alive. Client leave UX (confirm dialog rules) for the human is unchanged.

#### Scenario [SC-LEAVE-20]: Consented leave ends AI room with bots present

- **GIVEN** an effective-AI tourist room in phase `playing` with one human seat and two bot seats
- **WHEN** the human completes a consented leave
- **THEN** the room is disposed
- **AND** bot seats do not keep the room running

#### Scenario [SC-LEAVE-21]: Human reconnect grace expiry ends AI room

- **GIVEN** an effective-AI tourist room in any phase where the human seat is offline within reconnect grace and bot seats remain
- **WHEN** the human’s reconnect grace expires without reconnect
- **THEN** the room is disposed

#### Scenario [SC-LEAVE-22]: Consented leave in waiting still disposes AI room

- **GIVEN** an effective-AI tourist room still in phase `waiting` or `countdown` with one human seat and bot seats already assigned
- **WHEN** the human completes a consented leave
- **THEN** the room is disposed
