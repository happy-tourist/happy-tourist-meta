# Delta: game/board

Связано с main `openspec/specs/game/board/spec.md` и `game/move` (landable removed). Этот change: снятый task-тайл остаётся визуальной дырой, но снова **проходим** (hints + сервер). **Titles** `### Requirement` / `#### Scenario` совпадают с main (OpenSpec validate); **bodies / THEN** — продуктовый канон (design D5).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-11 | modified — client UX + server (hole visible; land accepted) |
| SC-BOARD-12 | covered-by-reuse (piece may stand on hole) |
| SC-BOARD-15 | modified — client UX + server (red hints include removed; mirrors SC-MOVE-49) |

## MODIFIED Requirements

### Requirement: Removed task tiles are visual holes and not landable

When a peek resolves as **correct**, the server SHALL mark that task cell as removed in synchronized state visible to every client in the room. Incorrect resolves (button, timeout-forced, leave mid-peek, or end-turn with open peek) MUST NOT mark the cell removed. Every client MUST render a removed task cell as an empty hole (page background, no brown tile chrome). Removed task cells MUST be **legal landing targets** for move, push, return-from-finish, and catapult fling when otherwise playable and unoccupied (`game/move`); red destination hints MUST include them when they are legal for the selected piece. A piece that stood on the cell when it was removed MUST remain at that row and column (standing on the void is allowed). That piece MAY later leave onto a legal neighbor or another piece MAY land onto the void. Layout holes that were **never** playable on the map snapshot stay non-playable. Peek MUST NOT open on a removed hole. Shop / restoring tiles is out of scope.

#### Scenario [SC-BOARD-11]: Everyone sees the hole and cannot land on it

- **GIVEN** a peek resolves as correct on a task cell while seated players and a spectator view the board
- **WHEN** synced removed-tile state updates
- **THEN** every client shows that cell without brown task chrome
- **AND** a later legal move onto that cell is accepted (removed task holes are landable)

#### Scenario [SC-BOARD-12]: Piece stays when its tile is removed

- **GIVEN** a tourist piece stands on a task cell that is then removed by a **correct** peek
- **WHEN** clients render the board
- **THEN** that piece remains at the same row and column
- **AND** the cell shows as a visual hole under the piece

#### Scenario [SC-BOARD-15]: No client offers a removed hole as a move target

- **GIVEN** it is the user’s turn with steps ≥ 1 (or infinite) and a selected unfinished piece adjacent to a free removed task hole
- **WHEN** red legal destination hints are shown
- **THEN** that removed hole is outlined as a destination
- **AND** submitting a move onto it is accepted by the server
