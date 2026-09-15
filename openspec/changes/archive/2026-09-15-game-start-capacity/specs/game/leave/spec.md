## Purpose

Клиентский выход из room `tourist` обратно в лобби: подпись действия «Выход из игры» и модальное подтверждение перед irreversible consented leave для seated-игрока после старта партии (`playing`). Серверный контракт leave не меняется (`game/pieces`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-01 | covered (client UX) |
| SC-LEAVE-02 | covered (client UX) |
| SC-LEAVE-03 | covered (client UX) |
| SC-LEAVE-04 | covered (client UX) |
| SC-LEAVE-05 | covered (client UX) |

## ADDED Requirements

### Requirement: Exit control label

On the game screen the primary control that returns the user to the lobby SHALL be labeled as leaving the game (product Russian label «Выход из игры»), not as navigating to the lobby by name alone.

#### Scenario [SC-LEAVE-01]: Exit control wording

- **GIVEN** the user is on the game screen of a tourist room
- **WHEN** the primary leave-to-lobby control is shown
- **THEN** its visible label is «Выход из игры» (localized per client locale files)

### Requirement: Confirm leave after play has started

When a seated player whose room start phase is `playing` activates the exit control, the client SHALL present a confirmation dialog before performing a consented leave. The dialog MUST ask whether the user is sure and MUST warn that leaving will reset progress (product Russian copy: «Вы уверены? Если выйдете, прогресс будет сброшен.»). The dialog MUST offer cancel and confirm-exit actions. Cancel MUST dismiss the dialog without leaving the room. Confirm-exit MUST perform the existing consented leave and navigate the user to the lobby.

#### Scenario [SC-LEAVE-02]: Playing seated player sees confirm

- **GIVEN** a seated player in a tourist room whose phase is `playing`
- **WHEN** that player activates the exit control
- **THEN** a confirmation dialog is shown with the progress-reset warning
- **AND** the player remains in the room until they confirm exit

#### Scenario [SC-LEAVE-03]: Cancel keeps the player in the game

- **GIVEN** the leave confirmation dialog is open for a seated player in phase `playing`
- **WHEN** the player chooses cancel
- **THEN** the dialog closes
- **AND** the player stays in the tourist room without a consented leave

#### Scenario [SC-LEAVE-04]: Confirm performs consented leave

- **GIVEN** the leave confirmation dialog is open for a seated player in phase `playing`
- **WHEN** the player confirms exit
- **THEN** the client performs a consented leave of the tourist room
- **AND** the user is taken to the lobby screen

### Requirement: No confirm when leave is not progress-critical

The confirmation dialog MUST NOT be required when the user is not a seated player in phase `playing`. Spectators, and seated players while the phase is `waiting` or `countdown`, MUST leave immediately via the existing consented leave path when they activate the exit control.

#### Scenario [SC-LEAVE-05]: Immediate leave outside playing seat

- **GIVEN** a user on the game screen who is a spectator, or a seated player while phase is `waiting` or `countdown`
- **WHEN** that user activates the exit control
- **THEN** no leave confirmation dialog is shown
- **AND** the client performs a consented leave and navigates to the lobby
