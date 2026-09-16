## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-01 | covered (client UX — icon-only exit) |
| SC-LEAVE-05 | covered (client UX — includes time-expired) |
| SC-LEAVE-06 | covered-by-reuse (client UX — finished) |
| SC-LEAVE-07 | covered (client UX) |

Related: time-expired lock — `game/move`; finished leave — `game/finish`.

## MODIFIED Requirements

### Requirement: Exit control label

On the game screen the primary control that returns the user to the lobby SHALL be presented as an **icon-only** exit control using the Material Icons glyph `logout` (no visible text label on the control). The control MUST expose an accessible name equivalent to leaving the game (product Russian accessible name «Выход из игры», localized per client locale files). The control MUST NOT use a visible text label such as «Выход из игры» that consumes horizontal header space on narrow viewports.

#### Scenario [SC-LEAVE-01]: Exit control wording

- **GIVEN** the user is on the game screen of a tourist room
- **WHEN** the primary leave-to-lobby control is shown
- **THEN** the control shows the `logout` icon without a visible text label
- **AND** its accessible name is «Выход из игры» (localized per client locale files)

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
