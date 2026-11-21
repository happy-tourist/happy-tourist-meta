# Delta: game/board

Взаимоисключающие типы клеток под тайлом; open-on-land вместо peek-иконки; knowledge digit; счётчик решётки; пустая клетка после ловушки/угаданного.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-10 | removed — peeks budget gate |
| SC-BOARD-13 | removed — peek affordance icon |
| SC-BOARD-16 | removed — medium density seed; see SC-BOARD-80 |
| SC-BOARD-20 | removed — spent grille leaves task peekable |
| SC-BOARD-41 | modified — bind on first open-on-land |
| SC-BOARD-43 | modified — correct grants knowledge not steps |
| SC-BOARD-45 | removed — flipped peek spends peeks |
| SC-BOARD-48 | modified — deck reshuffle on open-on-land |
| SC-BOARD-70 | modified — no turn-timeout close reason |
| SC-BOARD-80 | pending — exclusive cell types at seed |
| SC-BOARD-81 | pending — fixed catapult 15% exclusive |
| SC-BOARD-82 | pending — open-on-land binds/opens task |
| SC-BOARD-83 | pending — wrong keeps open knowledge digit |
| SC-BOARD-84 | pending — no peek icon affordance |
| SC-BOARD-85 | pending — grille counter top-right red |
| SC-BOARD-86 | pending — trap clear leaves empty cell |
| SC-BOARD-87 | pending — correct removes tile empty cell |

## ADDED Requirements

### Requirement: Task cells are exclusively task grille or catapult

When phase becomes `playing`, the server SHALL partition still-present map task cells (`*`) into three disjoint sets: **task** (Q&A reward), **grille**, and **catapult**. A cell MUST NOT hold more than one of these. Let `n` be the task-cell count. Grille count MUST be `min(round(n * 0.12), n)`. Catapult count MUST be `min(round(n * 0.15), n - grilleCount)`. Remaining task cells are Q&A tasks. Start and finish cells MUST NOT receive traps or task binds. Placement MUST NOT be predictable from coordinates alone. Create-time density options MUST NOT affect this partition (server ignores them if present).

#### Scenario [SC-BOARD-80]: Seed partitions without overlap

- **GIVEN** a tourist room entering phase `playing` with 48 task cells
- **WHEN** traps and tasks are seeded
- **THEN** exactly `round(48 * 0.12)` = **6** cells are grille-only
- **AND** exactly `round(48 * 0.15)` = **7** cells are catapult-only
- **AND** no cell is both grille and catapult or trap-plus-task
- **AND** the remaining task cells are Q&A task cells

#### Scenario [SC-BOARD-81]: Fixed catapult ratio without create density

- **GIVEN** a tourist room created without client density options
- **WHEN** catapults are seeded at playing start
- **THEN** catapult count is `min(round(n * 0.15), n - grilleCount)` for task-cell count `n`
- **AND** those cells are not also grille or Q&A task cells

### Requirement: Landing opens the cell content

When a free unfinished piece lands on a still-covered exclusive cell (by move, return, or catapult fling), the server MUST reveal that cell’s content without a separate peek affordance or peeks budget (hook after the paced trap pipeline is idle for that land/hop):

- **Q&A task** (unbound or already open/flipped): bind if needed and open the shared task modal for every client; only the **landing** seat may edit slots (reopen replaces prior session owner). Standing on a cell without a new land MUST NOT reopen.
- **Grille**: trap that piece (`remainingTurns = 3`) and show grille presentation per existing grille anim rules; then advance turn per `game/move`.
- **Catapult**: consume and fling per existing catapult rules; the destination land MUST apply this same open-on-land rule recursively.

There MUST NOT be a board peek icon above tourists. Opening MUST NOT require or spend peeks. A client `peek` intent MUST NOT open a task; only landing (or fling land) opens/reopens.

#### Scenario [SC-BOARD-82]: Move onto unbound task opens modal

- **GIVEN** it is a seated player’s turn and a legal move lands an own free unfinished piece on an unbound Q&A task cell
- **WHEN** the move is accepted
- **THEN** a task is bound to that cell
- **AND** every client sees the shared task modal
- **AND** no peek icon was required to open it

#### Scenario [SC-BOARD-84]: No peek affordance on board

- **GIVEN** phase is `playing` and an own unfinished free piece stands on an open or covered task cell
- **WHEN** the current-turn client views the board
- **THEN** no peek icon affordance is shown above that piece

### Requirement: Wrong answer leaves open knowledge digit

When a task modal closes without a correct submit (wrong order or dismiss without correct), the server MUST keep the tile present and bound, and every client MUST show the flipped/open task skin with the knowledge value digit (**difficulty** 1|2|3). Another free unfinished piece (any seat) that later lands on that open cell MUST reopen the same bound task modal. A **correct** submit MUST grant that piece **difficulty** knowledge points, remove the task tile leaving an empty playable cell (same presentation family as today’s removed task hole), and flash «Правильно!» per `game/presence`. Wrong submit MUST flash «Ошибка!» and MUST NOT grant knowledge.

#### Scenario [SC-BOARD-83]: Wrong keeps open cell with knowledge digit

- **GIVEN** a seated player has an open task modal and submits an incorrect slot order
- **WHEN** the modal closes as incorrect
- **THEN** the cell remains present with the bound task
- **AND** every client sees the open skin with digit equal to the task difficulty
- **AND** no knowledge is granted

#### Scenario [SC-BOARD-87]: Correct grants knowledge and clears cell

- **GIVEN** a seated player submits the correct slot order for a bound task of difficulty 2
- **WHEN** the server accepts the answer
- **THEN** that piece’s knowledge increases by 2
- **AND** the task tile is removed leaving an empty playable cell
- **AND** every client sees the «Правильно!» flash

### Requirement: Grille remaining-turns badge

While a piece is trapped by a grille, every client that shows that piece (board and personal strip chrome) MUST display the remaining auto-free turns as a badge at the **top-right** of the piece chrome on a **red** background. The value MUST match synchronized public `Piece.remainingTurns` from `game/move` (starts at **3**). When the grille clears, the badge MUST disappear with the grille. This badge is **public** (all clients). Own carried-knowledge yellow circle (`game/presence`) is a separate private badge at bottom-right and MUST NOT replace the grille counter.

#### Scenario [SC-BOARD-85]: Trapped piece shows red top-right counter

- **GIVEN** a piece is trapped with remaining turns 3
- **WHEN** any client views the board
- **THEN** that piece shows a red-background counter at top-right with value 3
- **AND** other seats see the same red counter (public sync)

### Requirement: Clearing a trap leaves an empty cell

When a grille is cleared (rescue, auto-free, all-jail, or leave) or a catapult is consumed after fling, the underlying cell MUST become an **empty playable** cell (same family as a removed task hole). The server MUST NOT leave a Q&A task under a former exclusive trap cell.

#### Scenario [SC-BOARD-86]: Grille clear leaves empty playable cell

- **GIVEN** a piece is trapped on an exclusive grille cell
- **WHEN** that grille is cleared by rescue or auto-free
- **THEN** the cell is an empty playable cell with no task bind and no trap
- **AND** every client shows the grille rise/vanish presentation

## MODIFIED Requirements

### Requirement: Shared peek modal with question and answer slots

While a task modal is open after open-on-land (or reopen on an already-open task cell), every client in the room MUST see the shared modal chrome (difficulty header, question, answer slots, answer pool) as previously specified for peek modal layout. Only the seat whose piece opened the modal MAY place answers. A **correct** submit MUST require exact slot order, MUST add **difficulty** knowledge to that piece (not steps), MUST remove the task tile to an empty playable cell, and MUST flash «Правильно!». Wrong submit MUST KEEP the open bound cell and flash «Ошибка!». There is no turn-deadline force-close path (`game/move` has no turn timer); other force-closes (leave mid-modal) MUST KEEP the bind without the «Не успел!» flash.

#### Scenario [SC-BOARD-43]: Correct ordered slots grant difficulty steps

- **GIVEN** a seated player has an open task modal with slots filled in the task’s correct order
- **WHEN** that player submits the peek answer
- **THEN** the server accepts the answer as correct
- **AND** that piece gains difficulty knowledge points (not steps) and the task tile is removed
- **AND** every client in the room sees the match-status flash «Правильно!» for about ten seconds

#### Scenario [SC-BOARD-44]: Wrong order keeps the same bound task

- **GIVEN** the same setup as SC-BOARD-43 except the slot order is incorrect
- **WHEN** that player submits the peek answer
- **THEN** the server rejects the answer as incorrect
- **AND** the task tile and bind remain (KEEP) with the open knowledge digit visible
- **AND** every client in the room sees the match-status flash «Ошибка!» for about ten seconds

#### Scenario [SC-BOARD-46]: Everyone sees the peek modal and placements

- **GIVEN** a seated player opens a task modal by landing on a task cell
- **WHEN** that player places answer cards into slots
- **THEN** every other client in the room sees the same modal chrome and placements
- **AND** those other clients cannot edit the slots

#### Scenario [SC-BOARD-47]: Leave mid-peek keeps bind

- **GIVEN** a seated player has an open task modal on task cell C
- **WHEN** that player’s piece leaves C or the modal is force-resolved as incorrect without a wrong-submit flash path
- **THEN** the bind and open knowledge digit remain on C (KEEP)
- **AND** the task tile is not removed

### Requirement: Task deck binds on first peek

At playing start the server SHALL build a shuffled deck from all tasks in the room’s selected published task sets (same pack snapshot). Unrevealed still-present exclusive **Q&A task** cells MUST NOT have a bound task until the first open-on-land open on that cell. The first open on an unbound Q&A cell MUST take the next task from the deck and **bind** it to that cell for the rest of the match until the tile is removed. When the deck is exhausted, the server MUST reshuffle a new round from the same task pool and continue dealing. Bound tasks MUST NOT be reassigned to another cell while the tile remains. Exclusive grille/catapult cells MUST NOT receive a Q&A bind.

#### Scenario [SC-BOARD-41]: First peek binds next deck task

- **GIVEN** phase is playing, an unbound still-present Q&A task cell C, and a shuffled deck whose next task is T with difficulty 2
- **WHEN** a seated player’s free unfinished piece legally lands on C and open-on-land opens the task
- **THEN** T is bound to C
- **AND** every client observes C as flipped with difficulty digit 2

#### Scenario [SC-BOARD-48]: Exhausted deck starts a new round

- **GIVEN** the deck has no remaining unbound deals and at least one unbound Q&A task cell remains
- **WHEN** a first open-on-land open is performed on that cell
- **THEN** the server deals from a reshuffled round of the same task pool
- **AND** a task is bound to that cell

### Requirement: Peek close announces outcome reason

When the server closes an open task/peek session, it MUST broadcast a peek-close event to every client that includes whether the resolve was correct and, when incorrect, a close reason that distinguishes at least **wrong submit** from other force-closes. Clients MUST use that payload (not heuristics alone) to choose among the header flashes in `game/presence`. Because `game/move` has no turn timer, the server MUST NOT emit a **turn-timeout** force-close reason for this economy. Leave mid-modal and other non-wrong force-closes MUST NOT be labeled as turn-timeout and MUST NOT use the «Не успел!» flash.

#### Scenario [SC-BOARD-70]: Peek close carries correct flag and reason

- **GIVEN** an open task modal in a tourist room
- **WHEN** the server resolves that modal (correct submit, wrong submit, or leave/force-close without wrong submit)
- **THEN** every client receives a peek-close message with the correct outcome flag
- **AND** when the resolve is incorrect due to a wrong submit, the message identifies the wrong reason
- **AND** the server does not emit a turn-timeout close reason (no turn timer in this economy)

## REMOVED Requirements

### Requirement: Grilles seed on task cells by density

**Reason**: Replaced by fixed 12% exclusive grille partition (ADDED exclusive cell types / SC-BOARD-80).
**Migration**: Seed grilles at 12% exclusive; do not use create density presets.

### Requirement: Every peek open spends peeks (fresh and flipped)

**Reason**: Peeks budget removed; open-on-land opens tasks.
**Migration**: Land on cell to open; no peeks counter.

### Requirement: Eye affordance to open a peek

**Reason**: Peek icon removed; open-on-land opens tasks.
**Migration**: No eye/peek affordance; rescue remains above tourists when applicable.

### Requirement: Spent grille leaves the task tile peekable

**Reason**: Grille cells are exclusive; clear leaves empty cell.
**Migration**: After grille clear, cell is empty playable with no task.
