## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-15 | pending (client UX) |
| SC-PRESENCE-16 | pending (client UX) |
| SC-PRESENCE-17 | pending (client UX) |
| SC-PRESENCE-18 | pending (client UX) |
| SC-PRESENCE-19 | pending (client UX) |

Related: budgets / end-turn / solo ∞ — `game/move`; peek modal — `game/board`.

## ADDED Requirements

### Requirement: Own step and peek counters beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s private **steps** and **peeks** counters next to their own presence marker. When a budget is infinite (solo mode), the counter MUST display an infinity indication rather than a finite number. Other seated players’ and spectators’ clients MUST NOT show another seat’s step or peek counters. Spectators MUST NOT see step/peek counters for any seat. When the user’s finite budgets increase, the client SHOULD play a short local “+N falls into the counter” animation for steps and peeks grants (turn grant and successful peek rewards).

#### Scenario [SC-PRESENCE-15]: Seated user sees only own counters

- **GIVEN** two seated players in phase `playing` with different private budgets
- **WHEN** each views Game presence
- **THEN** each sees steps and peeks only on their own marker
- **AND** neither sees the other’s budget numbers on the opponent marker

#### Scenario [SC-PRESENCE-16]: Solo shows infinity on own counters

- **GIVEN** the user is the sole non-finished seated player under infinite budgets
- **WHEN** the user views their presence marker
- **THEN** steps and peeks counters show infinity
- **AND** other clients still do not see those budget values

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show a control whose visible label is exactly **«Завершить ход»** next to that user’s own marker. Activating it MUST submit end-turn per `game/move`. The control MUST NOT appear for spectators, for seats that are not current turn, during solo infinite-budget play, or for finished / time-expired seats.

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views their presence marker
- **THEN** a control labeled «Завершить ход» is shown next to that marker
- **AND** activating it submits end-turn

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is under solo infinite budgets
- **WHEN** the user views their presence marker
- **THEN** the «Завершить ход» control is not shown

### Requirement: Solo unlimited-resources modal

When the user’s seat enters solo infinite steps/peeks mode (`game/move`), that user’s client MUST show a modal whose product Russian sense informs them that they are alone, that steps and peeks are unlimited, and that tile peeks are not limited per turn. Closing the modal MUST leave the user in the room. Other clients MUST NOT show that modal.

#### Scenario [SC-PRESENCE-19]: Only the solo player sees the unlimited modal

- **GIVEN** a seated player becomes the sole non-finished seat and at least one spectator or finished seated client is present
- **WHEN** solo infinite budgets apply
- **THEN** that player’s client shows the unlimited-resources modal
- **AND** other clients do not show that modal
