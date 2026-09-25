## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-30 | covered (client vitest — focus control placement) |
| SC-PRESENCE-31 | covered (client vitest — focuses nearest actionable) |
| SC-PRESENCE-32 | covered (client vitest — hidden when none actionable) |

Related: strip / selection — `game/pieces`; peek/move availability — `game/board` / `game/move`; end-turn — existing presence end-turn requirement.

## ADDED Requirements

### Requirement: Focus actionable tourist control

While the local user is a seated current-turn player in phase `playing` and at least one of their unfinished free pieces can currently take an action (legal move with steps remaining, fresh peek, flipped peek, rescue, push, or return-from-finish when applicable), the Game presence chrome MUST show a **circular** icon-only control placed on the own presence avatar between the say affordance and the end-turn affordance (product placement: along the avatar circumference between those two controls). Activating it MUST select / focus the **nearest** (or otherwise first) own unfinished free piece that has at least one such available action, without requiring a confirm dialog. When no own piece has an available action, the control MUST be hidden or disabled. Spectators and non-current seats MUST NOT see an enabled control for themselves.

#### Scenario [SC-PRESENCE-30]: Focus control sits between say and end-turn

- **GIVEN** it is the local seated user’s multiplayer turn with at least one actionable own tourist
- **WHEN** the own presence chrome is shown
- **THEN** a circular focus control is present between the say and end-turn affordances on the own avatar

#### Scenario [SC-PRESENCE-31]: Focus selects nearest actionable tourist

- **GIVEN** it is the local user’s turn with two own unfinished free pieces where only piece B can peek or move and piece A cannot
- **WHEN** the user activates the focus control
- **THEN** piece B becomes selected / focused for board actions
- **AND** piece A is not selected

#### Scenario [SC-PRESENCE-32]: Focus hidden when nothing actionable

- **GIVEN** it is the local user’s turn and no own unfinished free piece has a legal move, peek, rescue, push, or return
- **WHEN** the own presence chrome is shown
- **THEN** the focus control is not available as an enabled action
