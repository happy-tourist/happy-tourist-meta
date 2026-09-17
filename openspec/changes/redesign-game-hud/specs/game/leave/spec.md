# game/leave Delta

Related: shared header chrome also carries theme (`ui/theme`) and match status (`game/presence`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-01 | pending (client icon-only exit in shared header left) |
| SC-LEAVE-08 | pending (client exit only on Game, not Login/Lobby) |

## MODIFIED Requirements

### Requirement: Exit control label

On the game screen the primary control that returns the user to the lobby SHALL be presented as an **icon-only** exit control using the Material Icons glyph `logout` (no visible text label on the control), placed at the **left** side of the shared application header (theme toggle remains on the right). The control MUST expose an accessible name equivalent to leaving the game (product Russian accessible name «Выход из игры», localized per client locale files). The control MUST NOT use a visible text label such as «Выход из игры» that consumes horizontal header space on narrow viewports. Confirm / immediate-leave rules for activating the control are unchanged by this requirement.

#### Scenario [SC-LEAVE-01]: Exit control wording

- **GIVEN** the user is on the game screen of a tourist room
- **WHEN** the primary leave-to-lobby control is shown
- **THEN** the control shows the `logout` icon without a visible text label
- **AND** it is positioned at the left of the shared application header
- **AND** its accessible name is «Выход из игры» (localized per client locale files)

## ADDED Requirements

### Requirement: Exit control only while on Game

The shared application header MUST show the leave-to-lobby exit control only while the user is on the Game screen of a tourist room. On Login and Lobby screens the exit control MUST NOT appear. Theme toggle availability on those screens MUST remain unchanged (`ui/theme`).

#### Scenario [SC-LEAVE-08]: No exit control on Lobby

- **GIVEN** the user is on the Lobby screen
- **WHEN** the shared application header is shown
- **THEN** the leave-to-lobby `logout` control is not shown
- **AND** the theme toggle remains available
