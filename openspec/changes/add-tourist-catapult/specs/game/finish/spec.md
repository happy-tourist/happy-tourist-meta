## Purpose

Delta этого change: fling катапульты на center завершает фишку. Базовый finish / return — main `game/finish`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-19 | covered (server mocha + client UX — catapult fling onto center finishes) |

Related: fling rules — `game/move`; board anim — `game/board`.

## ADDED Requirements

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules, the server SHALL finish that piece with the same authority and presentation expectations as an ordinary move or push onto that center cell (`game/finish` enter-center rules), including travel/fade presentation for clients.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** the fling relocates that piece onto the center
- **THEN** that piece becomes finished
- **AND** clients present finish travel and disappear consistent with move/push onto center
