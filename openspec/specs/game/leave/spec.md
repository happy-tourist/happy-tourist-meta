# game/leave Specification

## Purpose

Клиентский выход из room `tourist` обратно в лобби: brand logo слева в общей шапке приложения как primary leave-control на Game с accessible name «Выход из игры» и модальное подтверждение перед irreversible consented leave для seated-игрока в `playing` без finish place и без time-expired. Серверный контракт leave не меняется (`game/pieces`); finished — `game/finish`; time-expired — `game/move`; match status в шапке — `game/presence`; brand chrome вне Game — `ui/branding`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-01 | covered (client UX — brand logo as leave control in shared header left) |
| SC-LEAVE-02 | covered (client UX — active playing still confirms) |
| SC-LEAVE-03 | covered (client UX) |
| SC-LEAVE-04 | covered (client UX) |
| SC-LEAVE-05 | covered (client UX — includes time-expired) |
| SC-LEAVE-06 | covered-by-reuse (client UX — finished) |
| SC-LEAVE-07 | covered (client UX) |
| SC-LEAVE-08 | covered (client UX — leave semantics only on Game; no Material logout) |

Related: time-expired lock — `game/move`; finished leave — `game/finish`; match status — `game/presence`; brand chrome — `ui/branding`.

## Requirements

### Requirement: Exit control label

On the game screen the primary control that returns the user to the lobby SHALL be the shared-header **Happy Tourist brand logo** (image mark, no visible text label such as «Выход из игры» or «В лобби»), placed at the **left** side of the shared application header (theme toggle remains on the right). The control MUST expose an accessible name equivalent to leaving the game (product Russian accessible name «Выход из игры», localized per client locale files). The control MUST NOT use the Material Icons glyph `logout`. Confirm / immediate-leave rules for activating the control are unchanged by this requirement. Brand-logo presence on non-Game screens is specified in `ui/branding`.

#### Scenario [SC-LEAVE-01]: Exit control wording

- **GIVEN** the user is on the game screen of a tourist room
- **WHEN** the primary leave-to-lobby control is shown
- **THEN** the control is the brand logo without a visible text label
- **AND** it is positioned at the left of the shared application header
- **AND** it does not use the Material `logout` icon
- **AND** its accessible name is «Выход из игры» (localized per client locale files)

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

### Requirement: Exit control only while on Game

The shared application header MUST apply leave-to-lobby semantics (confirmation when required, then consented leave and navigation to the lobby) to the brand logo **only** while the user is on the Game screen of a tourist room. The Material Icons `logout` leave control MUST NOT appear in the shared header. On Lobby, auth, and other non-Game screens brand-logo click behavior is specified in `ui/branding` (navigate toward lobby, lobby no-op, or auth→lobby with possible guard bounce), not as a consented leave. The brand logo remains an interactive control on those screens for layout stability (`ui/branding` SC-BRAND-09). Theme toggle availability MUST remain unchanged (`ui/theme`).

#### Scenario [SC-LEAVE-08]: No exit control on Lobby

- **GIVEN** the user is on the Lobby screen
- **WHEN** the shared application header is shown
- **THEN** the Material `logout` leave control is not shown
- **AND** activating the brand logo does not perform a consented leave of a tourist room
- **AND** the theme toggle remains available
