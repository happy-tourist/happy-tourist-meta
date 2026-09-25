## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-01 | modified (map snapshot layout) |
| SC-BOARD-07 | removed (superseded by task deck / difficulty) |
| SC-BOARD-08 | removed (superseded by Q&A peek) |
| SC-BOARD-09 | removed (superseded by Q&A peek) |
| SC-BOARD-10 | modified (fresh vs flipped peeks) |
| SC-BOARD-40 | pending (server mocha — layout from map) |
| SC-BOARD-41 | pending (server mocha — deck bind) |
| SC-BOARD-42 | pending (server mocha + client — flipped digit public) |
| SC-BOARD-43 | pending (server mocha — correct ordered slots) |
| SC-BOARD-44 | pending (server mocha — wrong keeps bind) |
| SC-BOARD-45 | pending (server mocha — flipped free peek at 0) |
| SC-BOARD-46 | pending (client UX — shared modal) |
| SC-BOARD-47 | pending (server mocha — leave mid-peek keeps bind) |
| SC-BOARD-48 | pending (server mocha — cycle deck) |

Related: create snapshot — `lobby/rooms`; piece count — `game/pieces`; peeks budget — `game/move`; stub removal — `content/packs`.

## REMOVED Requirements

### Requirement: Authoritative task-tile step rewards

**Reason:** Rewards no longer come from a fixed 28/14/6 bag on a hardcoded 48-cell layout. Steps come from the bound task’s difficulty.
**Migration:** Use the task-deck and difficulty requirements in this delta. Scenario SC-BOARD-07 no longer applies.

### Requirement: Peek under a task tile with stub answer controls

**Reason:** Stub «Правильно»/«Неправильно» without questions is replaced by shared content Q&A with server-validated ordered slots.
**Migration:** Use the shared peek and answer-validation requirements below. Scenarios SC-BOARD-08 and SC-BOARD-09 no longer apply as stub flows.

## MODIFIED Requirements

### Requirement: Tourist board layout on the game screen

The system SHALL present on the Game screen a board whose playable cells follow the **room’s map snapshot** on a 10-by-10 grid. Each cell MUST be one of: empty/hole, start, task (brown playable field), or finish, matching the snapshot taken at create from an in-catalog map. When synced tourist pieces exist, the board SHALL render every piece at its current synchronized cell (each seated player contributes exactly `touristsPerPlayer` pieces from that map snapshot — see `game/pieces`). Piece placement authority, seating, and movement rules are defined by capabilities `game/pieces` and `game/move`. The legacy fixed tourist ASCII layout MUST NOT be used when a map snapshot is present.

#### Scenario [SC-BOARD-01]: Board geometry and tile kinds

- **GIVEN** the user is authenticated and on the Game screen for a tourist room created from map M
- **WHEN** the board is shown
- **THEN** the 10×10 cells match M’s snapshot grid (start / task / finish / hole) with the product fills for those kinds
- **AND** if synced pieces exist, tourist pieces appear on their synchronized cells
- **AND** if no synced pieces exist, no player pieces are rendered on the board

## ADDED Requirements

### Requirement: Room ignores hardcoded tourist layout

Creating a room from an in-catalog map whose grid differs from the legacy fixed tourist layout MUST use that map’s snapshot for movement and rendering.

#### Scenario [SC-BOARD-40]: Room ignores hardcoded tourist layout

- **GIVEN** an in-catalog map M whose grid differs from the legacy fixed tourist layout
- **WHEN** a room is created from M and clients view Game in playing
- **THEN** movement and rendering use M’s snapshot
- **AND** the legacy fixed layout MUST NOT override M

### Requirement: Task deck binds on first peek

At playing start the server SHALL build a shuffled deck from all tasks in the room’s selected published task sets (same pack snapshot). Unrevealed still-present task cells (`*`) MUST NOT have a bound task until a successful first peek on that cell. The first peek on an unbound task cell MUST take the next task from the deck and **bind** it to that cell for the rest of the match until the tile is removed. When the deck is exhausted, the server MUST reshuffle a new round from the same task pool and continue dealing. Bound tasks MUST NOT be reassigned to another cell while the tile remains.

#### Scenario [SC-BOARD-41]: First peek binds next deck task

- **GIVEN** phase is playing, an unbound still-present task cell C, and a shuffled deck whose next task is T with difficulty 2
- **WHEN** a seated player successfully opens the first peek on C
- **THEN** T is bound to C
- **AND** every client observes C as flipped with difficulty digit 2

#### Scenario [SC-BOARD-48]: Exhausted deck starts a new round

- **GIVEN** the deck has no remaining unbound deals and at least one unbound task cell remains
- **WHEN** a first peek is opened on that cell
- **THEN** the server deals from a reshuffled round of the same task pool
- **AND** a task is bound to that cell

### Requirement: Flipped task cells show difficulty to everyone

After a task is bound to a still-present task cell, every client in the room MUST render that cell’s **difficulty** digit (1, 2, or 3) on the board without requiring the peek modal to be open. Unbound task cells MUST NOT show a difficulty digit. Removed task holes MUST NOT show a difficulty digit.

#### Scenario [SC-BOARD-42]: All clients see flipped difficulty

- **GIVEN** task cell C is bound with difficulty 3
- **WHEN** seated players and a spectator view the board
- **THEN** each sees digit 3 on C
- **AND** the peek modal need not be open

### Requirement: Shared peek modal with question and answer slots

While it is a seated client’s own turn and a peek is allowed for an own unfinished free piece on a still-present task cell, opening a peek MUST show a modal synchronized so **every** client in the room sees it: the bound task’s question, empty answer slots above, and answer-card chips below from that pack’s live/snapshot answer cards. Only the peeking seat MAY place chips into slots; other clients MUST see placements in realtime and MUST NOT edit slots. Closing without a correct submit (wrong order, leave cell, turn timeout, end-turn with open peek) MUST keep the tile and keep the same bound task. A **correct** submit MUST require the slot order to match the task’s defined slot order exactly; on success the server MUST add **difficulty** steps to that seat’s private steps budget and MUST remove the task tile. Forced incorrect resolves MUST keep bind and tile.

#### Scenario [SC-BOARD-43]: Correct ordered slots grant difficulty steps

- **GIVEN** it is a seated player’s turn, piece on bound cell C with difficulty 2 and two ordered slots A then B
- **WHEN** that player fills slots in order A then B and submits
- **THEN** the server accepts the answer
- **AND** that seat’s steps increase by 2
- **AND** the task tile is removed for all clients

#### Scenario [SC-BOARD-44]: Wrong order keeps the same bound task

- **GIVEN** the same setup as SC-BOARD-43
- **WHEN** that player submits slots in order B then A
- **THEN** the server rejects the answer as incorrect
- **AND** no steps are added from that submit
- **AND** C remains present and still bound to the same task
- **AND** every client still sees difficulty 2 on C

#### Scenario [SC-BOARD-46]: Everyone sees the peek modal and placements

- **GIVEN** a seated player opens a peek modal
- **WHEN** that player places an answer chip into a slot
- **THEN** every other client in the room observes the same question, slots, and that placement
- **AND** those other clients cannot change the slots

#### Scenario [SC-BOARD-47]: Leave mid-peek keeps bind

- **GIVEN** a seated player has an open peek on bound cell C
- **WHEN** that player’s piece leaves C or the peek is force-resolved as incorrect
- **THEN** C remains present with the same bound task
- **AND** a later peek on C shows the same question

### Requirement: Fresh peek spends peeks; flipped peek is free

Opening a peek on an **unbound** still-present task cell MUST require peeks remaining (or solo infinite peeks) and MUST consume one peek in finite mode for that first open. Opening a peek on an **already bound (flipped)** still-present task cell MUST be allowed even when peeks are **0** in finite mode, MUST NOT consume a peek, and still requires own turn and an own unfinished free piece on that cell. Peek on removed holes or trapped pieces remains rejected per existing rules.

#### Scenario [SC-BOARD-45]: Flipped peek allowed at zero peeks

- **GIVEN** it is a seated player’s turn with peeks 0 in finite mode and an own unfinished free piece on flipped still-present cell C
- **WHEN** that player opens a peek on C
- **THEN** the server accepts the open
- **AND** peeks remain 0
- **AND** the same bound task is shown

#### Scenario [SC-BOARD-10]: Fresh peek rejected without budget

- **GIVEN** it is a seated player’s turn with peeks 0 in finite mode and an own unfinished free piece on an **unbound** still-present task cell
- **WHEN** that player attempts to open a peek
- **THEN** the server rejects the peek
- **AND** no task is bound
