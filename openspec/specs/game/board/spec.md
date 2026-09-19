# game/board Specification

## Purpose

Игровое поле «Счастливый турист» на экране игры: форма доски, типы тайлов и адаптивная вёрстка; фигурки туристов — по sync seats (см. `game/pieces`); выбор/подсказки/ход — по `game/move` для текущего хода.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-01 | covered-by-reuse (client GamePage geometry; pieces at synced cells) |
| SC-BOARD-02 | covered (client GamePage --gap/--radius 2px) |
| SC-BOARD-03 | covered (client full-width board without side presence gutters) |
| SC-BOARD-04 | covered (fixed tile fills, transparent holes, task 2.3) |
| SC-BOARD-05 | covered (client — non-movers still non-interactive) |
| SC-BOARD-06 | covered (client — current-turn mover may select/hint/submit) |
| SC-BOARD-07 | covered (server mocha) |
| SC-BOARD-08 | covered (server mocha — no one-peek-per-turn gate) |
| SC-BOARD-09 | covered (server mocha) |
| SC-BOARD-10 | covered (server mocha) |
| SC-BOARD-11 | covered (client UX + server — hole not landable) |
| SC-BOARD-12 | covered (client UX — piece may stand on hole) |
| SC-BOARD-13 | covered (client UX — eye centered above peekable tourist) |
| SC-BOARD-14 | covered (client UX — eye; no one-peek-per-turn) |
| SC-BOARD-15 | covered (server mocha — land on hole rejected; mirrors SC-MOVE-49) |
| SC-BOARD-16 | covered (server mocha — medium density seed count) |
| SC-BOARD-17 | covered (client UX — hidden until land) |
| SC-BOARD-18 | covered (client UX — drop anim ~1000 ms all clients) |
| SC-BOARD-19 | covered (client UX — rise+vanish ~1000 ms on clear) |
| SC-BOARD-20 | covered (server mocha — spent tile still peekable) |
| SC-BOARD-21 | covered (client UX — rise on leave clear all clients) |

Related: presence row layout — `game/presence` (opponents top / self bottom); turn budgets / multi peek — `game/move`; trap/rescue — `game/move`; trapped / leave clear — `game/pieces`.

## Requirements

### Requirement: Tourist board layout on the game screen

The system SHALL present on the Game screen a board whose playable cells follow the agreed tourist layout on a 10-by-10 grid with empty corner holes: green start cells (`1`), brown task cells (`*`), and one solid yellow center block covering the central 2-by-2 area (`7`). When synced tourist pieces exist, the board SHALL render every piece at its current synchronized cell (each seated player contributes four pieces, one per side identity). Piece placement authority, seating, and movement rules are defined by capabilities `game/pieces` and `game/move`.

#### Scenario [SC-BOARD-01]: Board geometry and tile kinds

- **GIVEN** the user is on the Game screen with an active game session
- **WHEN** the board is shown
- **THEN** start cells appear only in the agreed start positions and use a green fill
- **AND** task cells appear only in the agreed task positions and use a brown fill
- **AND** the center is rendered as one continuous yellow block spanning the central 2-by-2 cells
- **AND** non-playable corner holes show no tile
- **AND** if synced pieces exist, tourist pieces appear on their synchronized cells
- **AND** if no synced pieces exist, no player pieces are rendered on the board

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

### Requirement: Board sits on page chrome background

The area behind the board holes and around the board SHALL use the same light or dark page background as the rest of the Game screen chrome. Switching the application theme between light and dark MUST keep the tourist tile colors (green starts, brown tasks, yellow center) unchanged.

#### Scenario [SC-BOARD-04]: Theme does not recolor tourist tiles

- **GIVEN** the user is on the Game screen with the tourist board visible
- **WHEN** the chrome theme is switched between light and dark
- **THEN** start, task, and center tile colors remain the same
- **AND** the board holes continue to show the current page background

### Requirement: Board is non-interactive

The tourist board SHALL accept piece selection, move-destination hints, and move submission only for the seated client whose turn it is, as defined by capability `game/move`. Spectators and seated clients who are not the current turn MUST NOT select pieces, show move-target hints, or submit moves from board or strip activation. Geometry, tile chrome sizing, and theme-independent tourist tile colors remain as previously specified for this capability.

#### Scenario [SC-BOARD-05]: No tile interaction

- **GIVEN** the user is on the Game screen with the tourist board visible as a spectator or as a seated player whose turn it is not
- **WHEN** the user attempts to activate a board tile or a tourist piece
- **THEN** the client does not select pieces, show move targets, or send a move message as a result of that activation

#### Scenario [SC-BOARD-06]: Current-turn seated player may interact for moves

- **GIVEN** the user is on the Game screen as the seated player whose turn it is
- **WHEN** the user activates their piece or a legal destination per `game/move`
- **THEN** the client MAY select that user’s pieces, show local move hints, and submit a move

### Requirement: Authoritative task-tile step rewards

When phase becomes `playing`, the server SHALL assign a hidden step reward of **1**, **2**, or **3** to every present brown task cell (`*`) using the fixed bag proportions **28** cells of 1, **14** of 2, and **6** of 3 (48 task cells on the tourist layout), shuffled so placement is not predictable from coordinates alone. Reward values MUST NOT be synchronized to clients until a seat successfully opens a peek on that cell. Start cells and center cells MUST NOT receive peek rewards.

#### Scenario [SC-BOARD-07]: Playing start seeds the reward bag

- **GIVEN** a tourist room transitioning into phase `playing` with the standard tourist layout
- **WHEN** task rewards are initialized
- **THEN** exactly 48 task cells have hidden rewards
- **AND** the multiset of rewards is 28×1, 14×2, and 6×3

### Requirement: Peek under a task tile with stub answer controls

While it is a seated client’s own turn in phase `playing`, that client MAY open a peek only when: the seat has peeks remaining (or solo infinite peeks), and one of that client’s own unfinished pieces stands on a task cell that still has a hidden reward. Opening a peek MUST show that client a modal whose product Russian sense is that they may earn the revealed reward amount of steps, with exactly two actions labeled **«Правильно»** and **«Неправильно»**. Other clients MUST NOT see that modal or the reward amount. Choosing «Правильно» MUST add the reward to that seat’s private steps budget and MUST remove the task tile per the removal requirement. Choosing «Неправильно» MUST add no steps and MUST NOT remove the task tile; the hidden reward MUST remain on that cell for a later successful peek. In either case the server MUST consume one peek in finite peeks mode. Forced incorrect resolves (multiplayer turn timeout, leave mid-peek, or end-turn with an open peek) MUST behave the same as «Неправильно» for tile and reward retention. Peek attempts that violate turn, budget, piece ownership, or cell rules MUST be rejected without revealing the reward. Multiple peeks in the same turn are allowed while peeks remain (`game/move`).

#### Scenario [SC-BOARD-08]: Correct answer grants the hidden steps

- **GIVEN** it is a seated player’s turn with peeks ≥ 1 and one own unfinished piece on a still-present task cell whose hidden reward is 2
- **WHEN** that player opens the peek and chooses «Правильно»
- **THEN** that seat’s steps budget increases by 2
- **AND** the peeks budget decreases by 1 in finite peeks mode
- **AND** only that client saw the reward amount in the modal
- **AND** the task tile is removed for all clients

#### Scenario [SC-BOARD-09]: Incorrect answer grants nothing and keeps the tile

- **GIVEN** the same setup as SC-BOARD-08
- **WHEN** that player opens the peek and chooses «Неправильно»
- **THEN** that seat’s steps budget does not increase from the peek
- **AND** the peeks budget decreases by 1 in finite peeks mode
- **AND** the task tile remains present for all clients
- **AND** the same hidden reward remains available on that cell

#### Scenario [SC-BOARD-10]: Peek rejected off task or without budget

- **GIVEN** it is a seated player’s turn and either peeks are 0 in finite peeks mode, or no own unfinished piece stands on a still-present task cell
- **WHEN** that player attempts to open a peek
- **THEN** the server rejects the peek
- **AND** no reward is revealed and no tile is removed

### Requirement: Removed task tiles are visual holes and not landable

When a peek resolves as **correct**, the server SHALL mark that task cell as removed in synchronized state visible to every client in the room. Incorrect resolves (button, timeout-forced, leave mid-peek, or end-turn with open peek) MUST NOT mark the cell removed. Every client MUST render a removed task cell as an empty hole (page background, no brown tile chrome). Removed task cells MUST **not** be legal landing targets for any move; red destination hints MUST NOT include them. A piece that stood on the cell when it was removed MUST remain at that row and column (standing on the void is allowed). That piece MAY later leave onto a legal neighbor. Start holes that were never playable stay non-playable. Shop / restoring tiles is out of scope for this change.

#### Scenario [SC-BOARD-11]: Everyone sees the hole and cannot land on it

- **GIVEN** a peek resolves as correct on a task cell while seated players and a spectator view the board
- **WHEN** synced removed-tile state updates
- **THEN** every client shows that cell without brown task chrome
- **AND** a later move onto that cell is rejected as onto a non-landable hole

#### Scenario [SC-BOARD-12]: Piece stays when its tile is removed

- **GIVEN** a tourist piece stands on a task cell that is then removed by a **correct** peek
- **WHEN** clients render the board
- **THEN** that piece remains at the same row and column
- **AND** the cell shows as a visual hole under the piece

#### Scenario [SC-BOARD-15]: No client offers a removed hole as a move target

- **GIVEN** it is the user’s turn with steps ≥ 1 and a selected unfinished piece adjacent to a removed task hole
- **WHEN** red legal destination hints are shown
- **THEN** that removed hole is not outlined as a destination
- **AND** submitting a move onto it is rejected by the server

### Requirement: Eye affordance to open a peek

While it is the user’s multiplayer or solo turn and a peek is currently allowed for one of their unfinished pieces on a still-present task cell, activating that piece MUST offer an eye affordance to open the peek modal. The eye affordance MUST be shown **centered above** that tourist on the board (same top-center family as strip return and board rescue/push affordances). When local selection is already kept on such a piece after a move (`game/move` keep-focus), the eye MUST appear without requiring another activation click. Clients that are not allowed to peek MUST NOT show that affordance as a way to open a peek. A removed-task hole under the piece MUST NOT offer the eye.

#### Scenario [SC-BOARD-13]: Current player sees eye on a peekable tourist

- **GIVEN** it is the user’s turn with an allowed peek and an own unfinished piece on a still-present task cell
- **WHEN** the user activates that piece
- **THEN** an eye affordance is available centered above that tourist to open the peek
- **AND** other clients do not gain that peek affordance for the user’s piece

#### Scenario [SC-BOARD-14]: Eye after move without re-select when still on a present task tile

- **GIVEN** it is the user’s turn with peeks remaining (or solo infinite peeks) and after a successful move the same unfinished piece remains selected on a still-present task cell
- **WHEN** the move animation finishes and interaction returns
- **THEN** the eye affordance is available on that piece without requiring another selection click
- **AND** if the cell under the piece is a removed-task hole or not a task cell, the eye is not shown for that piece

### Requirement: Grilles seed on task cells by density

When the room start phase becomes `playing`, the server SHALL place a hidden grille on a random subset of still-present brown task cells (`*`). The subset size MUST equal the create-time density percent of the layout’s task-cell count (few **12%**, medium **22%**, many **35%**), rounded to the nearest integer and clamped to `[0, taskCount]`. Start cells and center cells MUST NOT receive grilles. Grille placement MUST NOT be predictable from coordinates alone.

#### Scenario [SC-BOARD-16]: Medium density seeds about twenty-two percent of tasks

- **GIVEN** a tourist room created with medium grille density transitioning into phase `playing` with the standard tourist layout (48 task cells)
- **WHEN** grilles are seeded
- **THEN** exactly `round(48 * 0.22)` = **11** hidden grilles exist on distinct task cells
- **AND** no grille is on a start or center cell

### Requirement: Hidden grilles are invisible until triggered

Until a piece lands on a cell that still holds an unspent grille, clients MUST NOT render that grille. Spectators and other seats MUST NOT learn grille locations from synced state before reveal.

#### Scenario [SC-BOARD-17]: Board shows no grille before land

- **GIVEN** phase is `playing` and hidden grilles exist on some task cells
- **WHEN** clients render the board before any of those cells is landed on
- **THEN** no grille artwork is shown on those cells

### Requirement: Revealed grille drop and clear animations are public

When a piece lands on an unspent grille, every client that displays the board MUST show the grille lowering onto that cell (product sense: drops from above downward) for about **1000 ms**. When that grille is later cleared (rescue, all-jail holding clear, or permanent leave of the seat whose piece held that grille), every such client MUST show the grille rising and disappearing for about **1000 ms**. Cleared grilles MUST NOT remain visible afterward. Personal tourist chrome that mirrors a trapped piece (`game/pieces`) MUST use the same about **1000 ms** timing for its grille drop/rise presentation.

#### Scenario [SC-BOARD-18]: Everyone sees the drop

- **GIVEN** phase is `playing` and a piece lands on a cell with an unspent grille
- **WHEN** the move settles on that cell
- **THEN** every client shows the grille drop animation on that cell lasting about 1000 ms

#### Scenario [SC-BOARD-19]: Everyone sees rise and vanish on clear

- **GIVEN** a revealed grille is holding a trapped piece and is then cleared by a successful rescue
- **WHEN** the holding grille is cleared
- **THEN** every client shows the grille rise and vanish lasting about 1000 ms

#### Scenario [SC-BOARD-21]: Everyone sees rise when leave clears holding

- **GIVEN** a revealed holding grille is cleared because that seat permanently left
- **WHEN** clients update the board
- **THEN** every client shows the grille rise and vanish lasting about 1000 ms

### Requirement: Spent grille leaves the task tile peekable

Clearing a grille MUST remove that trap from the cell and MUST NOT remove the brown task tile or its hidden peek reward (if still present). After clear, a free unfinished piece standing on that cell MAY open a peek per existing peek rules.

#### Scenario [SC-BOARD-20]: After rescue the task may still be peeked

- **GIVEN** a piece was trapped on a still-present task cell whose grille was then cleared by rescue, and it is that seat’s turn with peeks remaining
- **WHEN** that free unfinished piece remains on that task cell
- **THEN** that seat MAY open a peek on that cell
- **AND** the cell is not treated as a removed-task hole solely because the grille was cleared
