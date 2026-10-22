# Delta: game/presence

Связано с main `openspec/specs/game/presence/spec.md` (SC-PRESENCE-21) и delta `game/move` этого change (немедленный solo `timeExpired` при steps=0 без legal peek). Client модалка уже зеркалит `timeExpired`; уточняем infer copy: trapped на `*` не считать «живой задачей» для steps-exhausted vs timer.

## Traceability

| ID | Coverage |
|----|----------|
| SC-PRESENCE-21 | modified-by-this-change (steps vs timer infer; trapped ≠ live peek target) |
| SC-PRESENCE-35 | covered (client vitest — steps-exhausted copy when timeExpired after steps=0 and only trapped/start remainder) |

## MODIFIED Requirements

### Requirement: Distinct solo end copy for timer vs steps exhaustion

When the user’s seat becomes time-expired in solo (`game/move`), the client MUST show an end modal whose product Russian copy depends on the cause: timer expiry versus steps exhaustion with no **legal peek** target (no own free unfinished piece on a still-present task cell). Both causes share the same locked interaction (no moves/peeks); the visible text MUST differ so the player understands whether time ran out or steps ran out. When inferring steps-exhausted vs timer locally, an unfinished piece that is **trapped** MUST NOT count as standing on a live task tile for the steps-exhausted branch — even if its cell is still a present task cell.

#### Scenario [SC-PRESENCE-21]: Steps-loss and timer-loss show different copy

- **GIVEN** the user is the sole non-finished seated player
- **WHEN** the seat becomes time-expired because steps reached 0 with no free unfinished piece on a still-present task cell
- **THEN** the client shows the steps-exhausted end copy
- **AND** when instead the five-minute deadline elapses with steps still available, the client shows the timer-expired end copy
- **AND** neither copy is shown to other clients as that user’s private end modal

#### Scenario [SC-PRESENCE-35]: Steps-exhausted copy when only jail or start remainder

- **GIVEN** the user’s seat becomes time-expired while steps are 0 and every own unfinished piece is either trapped or not on a still-present task cell
- **WHEN** that client opens the solo end modal
- **THEN** the client shows the steps-exhausted end copy (not the five-minute timer copy)
- **AND** a trapped unfinished piece on a still-present task cell MUST NOT flip the copy to the timer-expired text
