# game/board Specification

## Purpose

Статическое игровое поле «Счастливый турист» на экране игры: форма доски, типы тайлов и адаптивная вёрстка без кликов и правил хода; фигурки туристов — по sync seats (см. `game/pieces`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-01 | covered (client GamePage: all sync pieces) |
| SC-BOARD-02 | covered (CSS --tile/--gap/--radius, task 2.3) |
| SC-BOARD-03 | covered (width 100% + min(60px,…), task 2.3) |
| SC-BOARD-04 | covered (fixed tile fills, transparent holes, task 2.3) |
| SC-BOARD-05 | covered (non-interactive; pieces remain non-clickable) |

## Requirements

### Requirement: Tourist board layout on the game screen

The system SHALL present on the Game screen a static board whose playable cells follow the agreed tourist layout on a 10-by-10 grid with empty corner holes: green start cells (`1`), brown task cells (`*`), and one solid yellow center block covering the central 2-by-2 area (`7`). When synced tourist pieces exist, the board SHALL render every piece on its assigned start cell (each seated player contributes four pieces, one per side). Piece placement authority and seating rules are defined by capability `game/pieces`.

#### Scenario [SC-BOARD-01]: Board geometry and tile kinds

- **GIVEN** the user is on the Game screen with an active game session
- **WHEN** the board is shown
- **THEN** start cells appear only in the agreed start positions and use a green fill
- **AND** task cells appear only in the agreed task positions and use a brown fill
- **AND** the center is rendered as one continuous yellow block spanning the central 2-by-2 cells
- **AND** non-playable corner holes show no tile
- **AND** if synced pieces exist, tourist pieces appear on the corresponding start cells
- **AND** if no synced pieces exist, no player pieces are rendered on the board

### Requirement: Tile chrome sizing

Each board tile SHALL use a maximum side length of 60 CSS pixels, a corner radius of 12 CSS pixels, and a gap of 6 CSS pixels between adjacent tiles. On narrow viewports the board MUST span the content width from the left edge to the right edge of the game content area so that tile size shrinks below 60px as needed while preserving the grid proportions.

#### Scenario [SC-BOARD-02]: Max tile size on wide viewports

- **GIVEN** the Game screen is shown on a viewport wide enough for ten tiles at 60px plus gaps
- **WHEN** the board is laid out
- **THEN** individual tile sides do not exceed 60px
- **AND** gaps between tiles are 6px
- **AND** tile corners use a 12px radius

#### Scenario [SC-BOARD-03]: Edge-to-edge board on narrow viewports

- **GIVEN** the Game screen is shown on a narrow viewport where ten 60px tiles plus gaps would overflow
- **WHEN** the board is laid out
- **THEN** the board occupies the full width of the game content area from left to right
- **AND** tiles scale down uniformly below 60px to fit

### Requirement: Board sits on page chrome background

The area behind the board holes and around the board SHALL use the same light or dark page background as the rest of the Game screen chrome. Switching the application theme between light and dark MUST keep the tourist tile colors (green starts, brown tasks, yellow center) unchanged.

#### Scenario [SC-BOARD-04]: Theme does not recolor tourist tiles

- **GIVEN** the user is on the Game screen with the tourist board visible
- **WHEN** the chrome theme is switched between light and dark
- **THEN** start, task, and center tile colors remain the same
- **AND** the board holes continue to show the current page background

### Requirement: Board is non-interactive

The tourist board SHALL NOT accept tile selection, piece selection, move hints, or move submission. Viewing the board or pieces MUST NOT require clicking tiles or pieces to progress.

#### Scenario [SC-BOARD-05]: No tile interaction

- **GIVEN** the user is on the Game screen with the tourist board visible
- **WHEN** the user attempts to activate a board tile or a tourist piece
- **THEN** the client does not select pieces, show move targets, or send a move message as a result of that activation
