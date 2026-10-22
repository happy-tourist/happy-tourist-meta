## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-01 | modified (skin fills for kinds) |
| SC-BOARD-02 | modified (gap 2; border-radius 0; silhouette from skin) |
| SC-BOARD-03 | unchanged (narrow edge-to-edge; carried with Tile chrome sizing) |
| SC-BOARD-04 | modified (theme-independent skins, not flat fills) |
| SC-BOARD-42 | modified (readable digit `#212121` on flip face) |
| SC-BOARD-55 | covered (client vitest boardGeometry — one tile per finish, no span) |
| SC-BOARD-56 | covered (client vitest GameBoardWire — unbound vs flipped task skins) |
| SC-BOARD-57 | covered (client vitest GameBoardWire — start/finish skins on board) |

## MODIFIED Requirements

### Requirement: Tourist board layout on the game screen

The system SHALL present on the Game screen a board whose playable cells follow the **room’s map snapshot** on a 10-by-10 grid. Each cell MUST be one of: empty/hole, start, task (playable field), or finish, matching the snapshot taken at create from an in-catalog map. Start, unbound task, flipped task, and finish cells MUST use the shipped product tile skins for those kinds (not flat CSS color fills). When synced tourist pieces exist, the board SHALL render every piece at its current synchronized cell (each seated player contributes exactly `touristsPerPlayer` pieces from that map snapshot — see `game/pieces`). Piece placement authority, seating, and movement rules are defined by capabilities `game/pieces` and `game/move`. The legacy fixed tourist ASCII layout MUST NOT be used when a map snapshot is present.

#### Scenario [SC-BOARD-01]: Board geometry and tile kinds

- **GIVEN** the user is authenticated and on the Game screen for a tourist room created from map M
- **WHEN** the board is shown
- **THEN** the 10×10 cells match M’s snapshot grid (start / task / finish / hole) with the product tile skins for those kinds
- **AND** if synced pieces exist, tourist pieces appear on their synchronized cells
- **AND** if no synced pieces exist, no player pieces are rendered on the board

### Requirement: Tile chrome sizing

Each board tile SHALL use a maximum side length of 60 CSS pixels and a gap of **2** CSS pixels between adjacent tiles. Tile silhouette (including notched corners) MUST come from the shipped tile skin artwork. Playable tiles MUST use CSS `border-radius: 0` (the prior **2px** corner radius MUST NOT be applied over skin art). On narrow viewports the board MUST span the content width from the left edge to the right edge of the game content area so that tile size shrinks below 60px as needed while preserving the grid proportions. Side presence columns MUST NOT steal horizontal space from that board width (presence rows sit above and/or below the board per `game/presence`).

#### Scenario [SC-BOARD-02]: Max tile size on wide viewports

- **GIVEN** the Game screen is shown on a viewport wide enough for ten tiles at 60px plus gaps
- **WHEN** the board is laid out
- **THEN** individual tile sides do not exceed 60px
- **AND** gaps between tiles are 2px
- **AND** each playable tile’s visible silhouette matches its skin (CSS border-radius 0 — not a clipped rounded square over the art)

#### Scenario [SC-BOARD-03]: Edge-to-edge board on narrow viewports

- **GIVEN** the Game screen is shown on a narrow viewport where ten 60px tiles plus gaps would overflow
- **WHEN** the board is laid out
- **THEN** the board occupies the full width of the game content area from left to right
- **AND** tiles scale down uniformly below 60px to fit
- **AND** no left/right presence gutter reduces that board width

### Requirement: Board sits on page chrome background

The area behind the board holes and around the board SHALL use the same light or dark page background as the rest of the Game screen chrome. Switching the application theme between light and dark MUST keep the tourist tile skins (start / task / flipped-task / finish artwork) unchanged — theme MUST NOT retint those skins.

#### Scenario [SC-BOARD-04]: Theme does not recolor tourist tiles

- **GIVEN** the user is on the Game screen with the tourist board visible
- **WHEN** the chrome theme is switched between light and dark
- **THEN** start, task, flipped-task, and finish tile skins remain the same
- **AND** the board holes continue to show the current page background

### Requirement: Flipped task cells show difficulty to everyone

After a task is bound to a still-present task cell, every client in the room MUST render that cell with the **flipped** task skin and that cell’s **difficulty** digit (1, 2, or 3) on the board without requiring the peek modal to be open. The digit MUST use dark ink **`#212121`** (no light text-shadow) so it remains readable on the flipped skin’s beige center. Unbound task cells MUST use the unbound task skin and MUST NOT show a difficulty digit. Removed task holes MUST NOT show a difficulty digit or task skin.

#### Scenario [SC-BOARD-42]: All clients see flipped difficulty

- **GIVEN** task cell C is bound with difficulty 3
- **WHEN** any client views the board
- **THEN** C shows the flipped task skin with a readable digit `3`
- **AND** unbound still-present task cells show the unbound task skin without a digit

## ADDED Requirements

### Requirement: Finish cells are one skin tile each

Every finish cell on the synced map snapshot MUST render as its **own** board tile with the finish skin. The client MUST NOT merge contiguous finish cells into a single multi-cell visual block. Selection outline and move-target outline on finish cells follow ordinary per-cell chrome (`game/move`).

#### Scenario [SC-BOARD-55]: Contiguous finish cells stay separate tiles

- **GIVEN** the map snapshot has a 2×2 block of finish cells
- **WHEN** the Game board is shown
- **THEN** the board shows four separate finish-skinned tiles
- **AND** the board MUST NOT present those four cells as one spanning finish block

### Requirement: Unbound and flipped task skins

A still-present **unbound** task cell MUST use the unbound task skin. After bind (flip), that cell MUST switch to the flipped task skin while it remains present. Correct peek removal still yields a hole (page background) per existing removed-task rules.

#### Scenario [SC-BOARD-56]: Unbound vs flipped task skins

- **GIVEN** a still-present unbound task cell C and a still-present flipped task cell D
- **WHEN** the board is shown
- **THEN** C uses the unbound task skin without a difficulty digit
- **AND** D uses the flipped task skin with its difficulty digit

### Requirement: Start and finish skins on the game board

Start cells MUST use the start skin. Finish cells MUST use the finish skin. Holes remain empty (page background).

#### Scenario [SC-BOARD-57]: Start and finish skins

- **GIVEN** a map snapshot with at least one start cell and one finish cell
- **WHEN** the Game board is shown
- **THEN** each start cell shows the start skin
- **AND** each finish cell shows the finish skin
