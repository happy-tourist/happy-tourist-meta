## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-01 | modified (pieces allowed; client board + pieces render) |
| SC-BOARD-05 | covered-by-reuse (non-interactive; pieces remain non-clickable) |

## MODIFIED Requirements

### Requirement: Tourist board layout on the game screen

The system SHALL present on the Game screen a static board whose playable cells follow the agreed tourist layout on a 10-by-10 grid with empty corner holes: green start cells (`1`), brown task cells (`*`), and one solid yellow center block covering the central 2-by-2 area (`7`). When synced tourist seats exist, the board SHALL render those player pieces on their assigned start cells. Piece placement authority and seating rules are defined by capability `game/pieces`.

#### Scenario [SC-BOARD-01]: Board geometry and tile kinds

- **GIVEN** the user is on the Game screen with an active game session
- **WHEN** the board is shown
- **THEN** start cells appear only in the agreed start positions and use a green fill
- **AND** task cells appear only in the agreed task positions and use a brown fill
- **AND** the center is rendered as one continuous yellow block spanning the central 2-by-2 cells
- **AND** non-playable corner holes show no tile
- **AND** if synced seats exist, tourist pieces appear on the corresponding start cells
- **AND** if no synced seats exist, no player pieces are rendered on the board

### Requirement: Board is non-interactive

The tourist board SHALL NOT accept tile selection, piece selection, move hints, or move submission. Viewing the board or pieces MUST NOT require clicking tiles or pieces to progress.

#### Scenario [SC-BOARD-05]: No tile interaction

- **GIVEN** the user is on the Game screen with the tourist board visible
- **WHEN** the user attempts to activate a board tile or a tourist piece
- **THEN** the client does not select pieces, show move targets, or send a move message as a result of that activation
