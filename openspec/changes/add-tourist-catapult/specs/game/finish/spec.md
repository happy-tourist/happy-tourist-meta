## Purpose

Delta этого change: fling катапульты на center завершает фишку. Базовый finish / return — main `game/finish`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-19 | covered (server mocha + client UX — catapult fling onto center finishes after vanish) |

Related: fling rules — `game/move`; board anim — `game/board`.

## ADDED Requirements

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules, the server SHALL finish that piece with the same authority and presentation expectations as an ordinary move or push onto that center cell (`game/finish` enter-center rules). Clients MUST run finish travel and disappear **after** the triggering catapult overlay has fully vanished (`game/board` successful presentation), not in parallel with that overlay.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** that catapult overlay has fully vanished and the deferred fling presentation runs
- **THEN** that piece becomes finished (server authority may already mark finished)
- **AND** clients present finish travel and disappear consistent with move/push onto center, starting only after the catapult vanish
