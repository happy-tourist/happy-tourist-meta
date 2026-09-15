## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-02 | pending (client UX — active playing still confirms) |
| SC-LEAVE-05 | pending (client UX — includes finished) |
| SC-LEAVE-06 | pending (client UX) |

Related: finished seat — `game/finish`.

## MODIFIED Requirements

### Requirement: Confirm leave after play has started

When a seated player whose room start phase is `playing` and who does **not** yet have a finish place activates the exit control, the client SHALL present a confirmation dialog before performing a consented leave. The dialog MUST ask whether the user is sure and MUST warn that leaving will reset progress (product Russian copy: «Вы уверены? Если выйдете, прогресс будет сброшен.»). The dialog MUST offer cancel and confirm-exit actions. Cancel MUST dismiss the dialog without leaving the room. Confirm-exit MUST perform the existing consented leave and navigate the user to the lobby.

#### Scenario [SC-LEAVE-02]: Playing seated player sees confirm

- **GIVEN** a seated player in a tourist room whose phase is `playing` and who has no finish place
- **WHEN** that player activates the exit control
- **THEN** a confirmation dialog is shown with the progress-reset warning
- **AND** the player remains in the room until they confirm exit

#### Scenario [SC-LEAVE-03]: Cancel keeps the player in the game

- **GIVEN** the leave confirmation dialog is open for a seated player in phase `playing` without a finish place
- **WHEN** the player chooses cancel
- **THEN** the dialog closes
- **AND** the player stays in the tourist room without a consented leave

#### Scenario [SC-LEAVE-04]: Confirm performs consented leave

- **GIVEN** the leave confirmation dialog is open for a seated player in phase `playing` without a finish place
- **WHEN** the player confirms exit
- **THEN** the client performs a consented leave of the tourist room
- **AND** the user is taken to the lobby screen

### Requirement: No confirm when leave is not progress-critical

The confirmation dialog MUST NOT be required when the user is not an active (non-finished) seated player in phase `playing`. Spectators, seated players while the phase is `waiting` or `countdown`, and seated players who already have a finish place MUST leave immediately via the existing consented leave path when they activate the exit control.

#### Scenario [SC-LEAVE-05]: Immediate leave outside playing seat

- **GIVEN** a user on the game screen who is a spectator, a seated player while the phase is `waiting` or `countdown`, or a seated player who already has a finish place
- **WHEN** that user activates the exit control
- **THEN** no leave confirmation dialog is shown
- **AND** the client performs a consented leave and navigates to the lobby

#### Scenario [SC-LEAVE-06]: Finished seat leaves without confirm

- **GIVEN** a seated player who has a finish place on the game screen
- **WHEN** that player activates the exit control
- **THEN** no leave confirmation dialog is shown
- **AND** the client performs a consented leave and navigates to the lobby
