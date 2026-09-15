## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-05 | pending (client UX — includes time-expired) |
| SC-LEAVE-06 | covered-by-reuse (client UX — finished) |
| SC-LEAVE-07 | pending (client UX) |

Related: time-expired lock — `game/move`; finished leave — `game/finish`.

## MODIFIED Requirements

### Requirement: No confirm when leave is not progress-critical

The confirmation dialog MUST NOT be required when the user is not an active move-capable seated player in phase `playing`. Spectators, seated players while the phase is `waiting` or `countdown`, seated players who already have a finish place, and seated players marked time-expired MUST leave immediately via the existing consented leave path when they activate the exit control.

#### Scenario [SC-LEAVE-05]: Immediate leave outside playing seat

- **GIVEN** a user on the game screen who is a spectator, a seated player while the phase is `waiting` or `countdown`, a seated player who already has a finish place, or a seated player marked time-expired
- **WHEN** that user activates the exit control
- **THEN** no leave confirmation dialog is shown
- **AND** the client performs a consented leave and navigates to the lobby

#### Scenario [SC-LEAVE-06]: Finished seat leaves without confirm

- **GIVEN** a seated player who has a finish place on the game screen
- **WHEN** that player activates the exit control
- **THEN** no leave confirmation dialog is shown
- **AND** the client performs a consented leave and navigates to the lobby

#### Scenario [SC-LEAVE-07]: Time-expired seat leaves without confirm

- **GIVEN** a seated player marked time-expired on the game screen
- **WHEN** that player activates the exit control
- **THEN** no leave confirmation dialog is shown
- **AND** the client performs a consented leave and navigates to the lobby
