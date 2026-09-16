## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-02 | pending (client GamePage --gap/--radius 2px) |
| SC-BOARD-03 | pending (client full-width board without side presence gutters) |

Related: presence row layout — `game/presence` (opponents top / self bottom).

## MODIFIED Requirements

### Requirement: Tile chrome sizing

Each board tile SHALL use a maximum side length of 60 CSS pixels, a corner radius of **2** CSS pixels, and a gap of **2** CSS pixels between adjacent tiles. On narrow viewports the board MUST span the content width from the left edge to the right edge of the game content area so that tile size shrinks below 60px as needed while preserving the grid proportions. Side presence columns MUST NOT steal horizontal space from that board width (presence rows sit above and/or below the board per `game/presence`).

#### Scenario [SC-BOARD-02]: Max tile size on wide viewports

- **GIVEN** the Game screen is shown on a viewport wide enough for ten tiles at 60px plus gaps
- **WHEN** the board is laid out
- **THEN** individual tile sides do not exceed 60px
- **AND** gaps between tiles are 2px
- **AND** tile corners use a 2px radius

#### Scenario [SC-BOARD-03]: Edge-to-edge board on narrow viewports

- **GIVEN** the Game screen is shown on a narrow viewport where ten 60px tiles plus gaps would overflow
- **WHEN** the board is laid out
- **THEN** the board occupies the full width of the game content area from left to right
- **AND** tiles scale down uniformly below 60px to fit
- **AND** no left/right presence gutter reduces that board width
