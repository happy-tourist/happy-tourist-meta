## Purpose

Delta этого change: визуал и анимация решёток на task-клетках. Геометрия поля, peek, дыры — main `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-16 | done (server mocha seed count) |
| SC-BOARD-17 | done (client UX — hidden until land) |
| SC-BOARD-18 | done (client UX — drop anim all clients) |
| SC-BOARD-19 | done (client UX — rise+vanish on clear) |
| SC-BOARD-20 | done (server mocha — spent tile still peekable) |

Related: trap/rescue rules — `game/move`; piece trapped — `game/pieces`.

## ADDED Requirements

### Requirement: Grilles seed on task cells by density

When the room start phase becomes `playing`, the server SHALL place a hidden grille on a random subset of still-present brown task cells (`*`). The subset size MUST equal the create-time density percent of the layout’s task-cell count (few **25%**, medium **45%**, many **65%**), rounded to the nearest integer and clamped to `[0, taskCount]`. Start cells and center cells MUST NOT receive grilles. Grille placement MUST NOT be predictable from coordinates alone.

#### Scenario [SC-BOARD-16]: Medium density seeds about forty-five percent of tasks

- **GIVEN** a tourist room created with medium grille density transitioning into phase `playing` with the standard tourist layout (48 task cells)
- **WHEN** grilles are seeded
- **THEN** exactly `round(48 * 0.45)` = **22** hidden grilles exist on distinct task cells
- **AND** no grille is on a start or center cell

### Requirement: Hidden grilles are invisible until triggered

Until a piece lands on a cell that still holds an unspent grille, clients MUST NOT render that grille. Spectators and other seats MUST NOT learn grille locations from synced state before reveal.

#### Scenario [SC-BOARD-17]: Board shows no grille before land

- **GIVEN** phase is `playing` and hidden grilles exist on some task cells
- **WHEN** clients render the board before any of those cells is landed on
- **THEN** no grille artwork is shown on those cells

### Requirement: Revealed grille drop and clear animations are public

When a piece lands on an unspent grille, every client that displays the board MUST show the grille lowering onto that cell (product sense: drops from above downward). When that grille is later cleared (rescue or all-jail holding clear), every such client MUST show the grille rising and disappearing. Cleared grilles MUST NOT remain visible afterward.

#### Scenario [SC-BOARD-18]: Everyone sees the drop

- **GIVEN** seated players and a spectator view the board
- **WHEN** a piece lands on a cell with an unspent grille
- **THEN** every client shows the grille drop animation on that cell

#### Scenario [SC-BOARD-19]: Everyone sees rise and vanish on clear

- **GIVEN** a revealed grille is holding a trapped piece and is then cleared by a successful rescue
- **WHEN** clients update
- **THEN** every client shows the grille rise and vanish
- **AND** the cell no longer shows grille artwork

### Requirement: Spent grille leaves the task tile peekable

Clearing a grille MUST remove that trap from the cell and MUST NOT remove the brown task tile or its hidden peek reward (if still present). After clear, a free unfinished piece standing on that cell MAY open a peek per existing peek rules.

#### Scenario [SC-BOARD-20]: After rescue the task may still be peeked

- **GIVEN** a piece was trapped on a still-present task cell whose grille was then cleared by rescue, and it is that seat’s turn with peeks remaining
- **WHEN** that free unfinished piece remains on that task cell
- **THEN** that seat MAY open a peek on that cell
- **AND** the cell is not treated as a removed-task hole solely because the grille was cleared
