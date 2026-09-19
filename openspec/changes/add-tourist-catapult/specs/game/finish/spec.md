## Purpose

Delta этого change: fling катапульты на center завершает фишку. Базовый finish / return — main `game/finish`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-19 | covered (server mocha + client UX — finish after land + catapult vanish; paced hop) |

Related: fling rules / paced resolve — `game/move`; board anim — `game/board`.

## ADDED Requirements

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules (after that hop’s presentation budget), the server SHALL finish that piece with the same authority and presentation expectations as an ordinary move or push onto that center cell (`game/finish` enter-center rules). Clients MUST run finish travel and disappear **after** the triggering catapult’s land arrival and overlay have fully completed (`game/board` successful presentation: land → ~1000 ms overlay → then finish travel), not in parallel with that overlay. Turn advance, if pending after the pipeline, MUST wait until finish presentation for that hop is covered by the trap pipeline idle rules in `game/move`.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** that catapult’s arrival and overlay have fully completed and the deferred fling / finish presentation runs
- **THEN** that piece becomes finished when the paced relocate+finish is applied
- **AND** clients present finish travel and disappear consistent with move/push onto center, starting only after the catapult vanish
