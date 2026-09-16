## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-15 | covered (client UX) |
| SC-PRESENCE-16 | covered (client UX — solo ∞ peeks only) |
| SC-PRESENCE-17 | covered (client UX) |
| SC-PRESENCE-18 | covered (client UX) |
| SC-PRESENCE-19 | covered (client UX — peeks-unlimited modal) |
| SC-PRESENCE-20 | covered (client UX — +N anim ~2s) |
| SC-PRESENCE-21 | covered (client UX — timer vs steps-loss copy) |

Related: budgets / end-turn / solo — `game/move`; peek modal — `game/board`.

## ADDED Requirements

### Requirement: Own step and peek counters beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s private **steps** and **peeks** counters next to their own presence marker. When peeks are infinite (solo mode), the peeks counter MUST display an infinity indication; the steps counter MUST show the finite numeric value. Other seated players’ and spectators’ clients MUST NOT show another seat’s step or peek counters. Spectators MUST NOT see step/peek counters for any seat. When the user’s finite budgets increase, the client SHOULD play a local “+N falls into the counter” animation for steps and peeks grants (turn grant and successful peek rewards) lasting approximately **two seconds**.

#### Scenario [SC-PRESENCE-15]: Seated user sees only own counters

- **GIVEN** two seated players in phase `playing` with different private budgets
- **WHEN** each views Game presence
- **THEN** each sees steps and peeks only on their own marker
- **AND** neither sees the other’s budget numbers on the opponent marker

#### Scenario [SC-PRESENCE-16]: Solo shows infinity only on peeks

- **GIVEN** the user is the sole non-finished seated player under infinite peeks and finite steps
- **WHEN** the user views their presence marker
- **THEN** the peeks counter shows infinity
- **AND** the steps counter shows the numeric steps value
- **AND** other clients still do not see those budget values

#### Scenario [SC-PRESENCE-20]: Budget grant animation lasts about two seconds

- **GIVEN** the user’s finite steps or peeks budget increases while they view their own presence marker
- **WHEN** the local +N fall animation plays
- **THEN** the animation is visibly slower than a sub-second flash and completes in about two seconds
- **AND** other clients do not see that animation

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show a control whose visible label is exactly **«Завершить ход»** next to that user’s own marker. Activating it MUST submit end-turn per `game/move`. The control MUST NOT appear for spectators, for seats that are not current turn, during solo play, or for finished / time-expired seats.

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views their presence marker
- **THEN** a control labeled «Завершить ход» is shown next to that marker
- **AND** activating it submits end-turn

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is under solo infinite peeks
- **WHEN** the user views their presence marker
- **THEN** the «Завершить ход» control is not shown

### Requirement: Solo peeks-unlimited modal

When the user’s seat enters solo infinite-peeks mode (`game/move`), that user’s client MUST show a modal whose product Russian sense informs them that they are alone, that peeks are unlimited, and that steps remain limited. Closing the modal MUST leave the user in the room. Other clients MUST NOT show that modal.

#### Scenario [SC-PRESENCE-19]: Only the solo player sees the peeks-unlimited modal

- **GIVEN** a seated player becomes the sole non-finished seat and at least one spectator or finished seated client is present
- **WHEN** solo infinite peeks apply
- **THEN** that player’s client shows the peeks-unlimited modal
- **AND** other clients do not show that modal

### Requirement: Distinct solo end copy for timer vs steps exhaustion

When the user’s seat becomes time-expired in solo (`game/move`), the client MUST show an end modal whose product Russian copy depends on the cause: timer expiry versus steps exhaustion with no live task tile under any unfinished piece. Both causes share the same locked interaction (no moves/peeks); the visible text MUST differ so the player understands whether time ran out or steps ran out.

#### Scenario [SC-PRESENCE-21]: Steps-loss and timer-loss show different copy

- **GIVEN** the user is the sole non-finished seated player
- **WHEN** the seat becomes time-expired because steps reached 0 with no unfinished piece on a still-present task cell
- **THEN** the client shows the steps-exhausted end copy
- **AND** when instead the five-minute deadline elapses with steps still available, the client shows the timer-expired end copy
- **AND** neither copy is shown to other clients as that user’s private end modal
