## Purpose

Delta этого change: peek eye affordance — сверху по центру над выбранным туристом (единый якорь с rescue/push/return).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-13 | covered (client UX — eye centered above peekable tourist) |
| SC-BOARD-14 | covered-by-reuse (eye after move keep-focus) |

Related: rescue/push centering — delta `game/move`; return strip — delta `game/finish`.

## MODIFIED Requirements

### Requirement: Eye affordance to open a peek

While it is the user’s multiplayer or solo turn and a peek is currently allowed for one of their unfinished pieces on a still-present task cell, activating that piece MUST offer an eye affordance to open the peek modal. The eye affordance MUST be shown **centered above** that tourist on the board (same top-center family as strip return and board rescue/push affordances). When local selection is already kept on such a piece after a move (`game/move` keep-focus), the eye MUST appear without requiring another activation click. Clients that are not allowed to peek MUST NOT show that affordance as a way to open a peek. A removed-task hole under the piece MUST NOT offer the eye.

#### Scenario [SC-BOARD-13]: Current player sees eye on a peekable tourist

- **GIVEN** it is the user’s turn with an allowed peek and an own unfinished piece on a still-present task cell
- **WHEN** the user activates that piece
- **THEN** an eye affordance is available centered above that tourist to open the peek
- **AND** other clients do not gain that peek affordance for the user’s piece

#### Scenario [SC-BOARD-14]: Eye after move without re-select when still on a present task tile

- **GIVEN** it is the user’s turn with peeks remaining (or solo infinite peeks) and after a successful move the same unfinished piece remains selected on a still-present task cell
- **WHEN** the move animation finishes and interaction returns
- **THEN** the eye affordance is available on that piece without requiring another selection click
- **AND** if the cell under the piece is a removed-task hole or not a task cell, the eye is not shown for that piece
