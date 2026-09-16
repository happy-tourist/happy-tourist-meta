## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-07 | pending (server mocha) |
| SC-BOARD-08 | pending-update (server mocha — no one-peek-per-turn gate) |
| SC-BOARD-09 | pending (server mocha) |
| SC-BOARD-10 | pending (server mocha) |
| SC-BOARD-11 | pending-update (client UX + server — hole not landable) |
| SC-BOARD-12 | pending (client UX — piece may stand on hole) |
| SC-BOARD-13 | pending (server mocha) |
| SC-BOARD-14 | pending-update (client UX — eye; no one-peek-per-turn) |
| SC-BOARD-15 | pending (server mocha — land on hole rejected; mirrors SC-MOVE-49) |

Related: turn budgets / multi peek — `game/move`; presence — `game/presence`.

## ADDED Requirements

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

While it is the user’s multiplayer or solo turn and a peek is currently allowed for one of their unfinished pieces on a still-present task cell, activating that piece MUST offer an eye affordance to open the peek modal. When local selection is already kept on such a piece after a move (`game/move` keep-focus), the eye MUST appear without requiring another activation click. Clients that are not allowed to peek MUST NOT show that affordance as a way to open a peek. A removed-task hole under the piece MUST NOT offer the eye.

#### Scenario [SC-BOARD-13]: Current player sees eye on a peekable tourist

- **GIVEN** it is the user’s turn with an allowed peek and an own unfinished piece on a still-present task cell
- **WHEN** the user activates that piece
- **THEN** an eye affordance is available to open the peek
- **AND** other clients do not gain that peek affordance for the user’s piece

#### Scenario [SC-BOARD-14]: Eye after move without re-select when still on a present task tile

- **GIVEN** it is the user’s turn with peeks remaining (or solo infinite peeks) and after a successful move the same unfinished piece remains selected on a still-present task cell
- **WHEN** the move animation finishes and interaction returns
- **THEN** the eye affordance is available on that piece without requiring another selection click
- **AND** if the cell under the piece is a removed-task hole or not a task cell, the eye is not shown for that piece
