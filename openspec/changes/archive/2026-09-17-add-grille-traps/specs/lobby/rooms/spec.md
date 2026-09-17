## Purpose

Delta этого change: выбор плотности решёток при создании `tourist` комнаты. Базовый listing/create maxSeats — main `lobby/rooms`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-13 | done (client UX + server parse) |
| SC-LOBBY-14 | done (server mocha create options — many → 35%) |
| SC-LOBBY-15 | done (client UX default; % under the hood) |

Related: seed решёток — `game/board` / `game/move`.

## ADDED Requirements

### Requirement: Create game chooses grille density

When an authenticated user opens create-game from the lobby, the system SHALL present a choice of grille density with three presets: few, medium, and many. The presets MUST map to **12%**, **22%**, and **35%** of task cells on the tourist layout at playing seed time (rounded to nearest integer, clamped to the task-cell count). Confirming create MUST pass the selected density into the new `tourist` room create options. Cancelling MUST NOT create a room. Product labels remain мало / средне / много without showing the percent numbers.

#### Scenario [SC-LOBBY-13]: Density presets are few medium many

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the selectable grille density options are few, medium, and many
- **AND** the product sense of the labels is мало / средне / много

#### Scenario [SC-LOBBY-14]: Confirm create uses selected density

- **GIVEN** the create modal is open with grille density many selected
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with grille density many (35% of task cells at seed)
- **AND** the user enters that game session

#### Scenario [SC-LOBBY-15]: Default grille density is medium

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** medium (22%) is selected by default for grille density
- **AND** max seats default remains 2 per existing create rules
