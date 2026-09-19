## Purpose

Delta этого change: отдельный выбор плотности катапульт при создании `tourist` комнаты. Базовый listing/create и плотность решёток — main `lobby/rooms`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-16 | covered (client UX + server parse) |
| SC-LOBBY-17 | covered (server mocha create options — many → 35% catapults) |
| SC-LOBBY-18 | covered (client UX default medium for catapults) |

Related: seed катапульт — `game/board` / `game/move`.

## ADDED Requirements

### Requirement: Create game chooses catapult density

When an authenticated user opens create-game from the lobby, the system SHALL present a choice of catapult density with three presets: few, medium, and many, **independent** of the grille density choice. The presets MUST map to **12%**, **22%**, and **35%** of task cells on the tourist layout at playing seed time (rounded to the nearest integer, clamped to the task-cell count). Confirming create MUST pass the selected catapult density into the new `tourist` room create options alongside grille density. Cancelling MUST NOT create a room. Product labels remain мало / средне / много without showing the percent numbers.

#### Scenario [SC-LOBBY-16]: Catapult density presets are few medium many

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the selectable catapult density options are few, medium, and many
- **AND** the product sense of the labels is мало / средне / много
- **AND** catapult density is chosen separately from grille density

#### Scenario [SC-LOBBY-17]: Confirm create uses selected catapult density

- **GIVEN** the create modal is open with catapult density many selected
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with catapult density many (35% of task cells at seed)
- **AND** the user enters that game session

#### Scenario [SC-LOBBY-18]: Default catapult density is medium

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** medium (22%) is selected by default for catapult density
- **AND** grille density default remains medium per existing create rules
