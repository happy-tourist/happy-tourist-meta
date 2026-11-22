# game/board Specification

## Purpose

Игровое поле «Счастливый турист» на экране игры: форма доски, типы тайлов и адаптивная вёрстка; фигурки туристов — по sync seats (см. `game/pieces`); выбор/подсказки/ход — по `game/move` для текущего хода.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-01 | covered (client — map snapshot + product tile skins) |
| SC-BOARD-02 | covered (client GamePage gap 2; border-radius 0; skin silhouette) |
| SC-BOARD-03 | covered (client full-width board without side presence gutters) |
| SC-BOARD-04 | covered (client — theme-independent tile skins, transparent holes) |
| SC-BOARD-05 | covered (client — non-movers still non-interactive) |
| SC-BOARD-06 | covered (client — current-turn mover may select/hint/submit) |
| SC-BOARD-10 | removed — peeks budget gate |
| SC-BOARD-11 | covered (client UX + server — hole not landable) |
| SC-BOARD-12 | covered (client UX — piece may stand on hole) |
| SC-BOARD-13 | removed — peek affordance icon |
| SC-BOARD-14 | removed — peek affordance after keep-focus |
| SC-BOARD-15 | covered (server mocha — land on hole rejected; mirrors SC-MOVE-49) |
| SC-BOARD-16 | removed — medium density seed; see SC-BOARD-80 |
| SC-BOARD-17 | covered (client UX — hidden until land) |
| SC-BOARD-18 | covered (client UX — drop anim ~1000 ms all clients) |
| SC-BOARD-19 | covered (client UX — rise+vanish ~1000 ms on clear) |
| SC-BOARD-20 | removed — spent grille leaves task peekable |
| SC-BOARD-21 | covered (client UX — rise on leave clear all clients) |
| SC-BOARD-22 | covered (client UX — hidden catapult until land) |
| SC-BOARD-23 | covered (client UX — land then ~1500 ms overlay then fling travel) |
| SC-BOARD-24 | covered (client UX — land then broken 450+450 then vanish ~300 ms; piece stays) |
| SC-BOARD-25 | covered (client UX — piece on catapult cell during overlay after land) |
| SC-BOARD-26 | covered (client UX — chain: land→overlay→travel per hop) |
| SC-BOARD-27 | covered (client GamePage continuous isBoardBusy + grille hold unlock) |
| SC-BOARD-28 | covered (client UX — land arrives before overlay; all viewers) |
| SC-BOARD-29 | covered (client UX — grille drop only on its hop after prior catapult hops) |
| SC-BOARD-30 | covered (client UX — single sequential trap timeline; no early final grille) |
| SC-BOARD-31 | covered (client UX — every hop animated; finish travel after last vanish to center) |
| SC-BOARD-32 | covered (client strip under isInteractive; say remains) |
| SC-BOARD-40 | covered (server mocha — layout from map) |
| SC-BOARD-41 | modified — bind on first open-on-land |
| SC-BOARD-42 | covered (server mocha + client — flipped skin + digit `#212121`) |
| SC-BOARD-55 | covered (client vitest boardGeometry — one tile per finish, no span) |
| SC-BOARD-56 | covered (client vitest GameBoardWire — unbound vs flipped task skins) |
| SC-BOARD-57 | covered (client vitest GameBoardWire — start/finish skins on board) |
| SC-BOARD-58 | covered (client — overlay drop-shadow on skinned tiles; board + strip chrome) |
| SC-BOARD-43 | modified — correct grants knowledge not steps |
| SC-BOARD-44 | covered (server mocha — wrong submit → KEEP bind + open knowledge digit; toast «Ошибка!») |
| SC-BOARD-45 | removed — flipped peek spends peeks |
| SC-BOARD-46 | covered (client UX — shared modal chrome) |
| SC-BOARD-47 | covered (server mocha — leave mid-peek keeps bind) |
| SC-BOARD-48 | modified — deck reshuffle on open-on-land |
| SC-BOARD-49 | covered (client UX — pool reuse without used-chrome) |
| SC-BOARD-50 | covered (client UX — dots left, word-only badge center) |
| SC-BOARD-51 | covered (client UX — empty/filled half size + primary border) |
| SC-BOARD-52 | covered (client UX — pool 4×4 half-size) |
| SC-BOARD-53 | covered (client UX — submit primary full-width) |
| SC-BOARD-54 | covered (client UX — game question ×1.5) |
| SC-BOARD-70 | modified — timeout close reason restored with turn timer |
| SC-BOARD-71 | covered (client UX — peek/rescue/push hits must not cover the tourist figure) |
| SC-BOARD-80 | modified — exclusive seed grille 15% / catapult 20% |
| SC-BOARD-81 | pending — fixed catapult 20% exclusive |
| SC-BOARD-82 | pending — open-on-land binds/opens task |
| SC-BOARD-83 | pending — wrong keeps open knowledge digit |
| SC-BOARD-84 | pending — no peek icon affordance |
| SC-BOARD-85 | modified — grille counter TR red above overlay |
| SC-BOARD-86 | pending — trap clear leaves empty cell |
| SC-BOARD-87 | pending — correct removes tile empty cell |
| SC-BOARD-88 | pending — outcome toast top-center (Notify) |

Related: create snapshot — `lobby/rooms`; piece count — `game/pieces`; peeks budget — `game/move`; stub removal — `content/packs`; presence row layout — `game/presence` (opponents top / self bottom); turn budgets / multi peek / paced traps / deferred turn — `game/move`; finish travel after catapult — `game/finish`; trapped / leave clear — `game/pieces`; peek chip reuse — SC-BOARD-49 (compose reuse — `content/packs` SC-PACK-263).

## Requirements

### Requirement: Tourist board layout on the game screen

The system SHALL present on the Game screen a board whose playable cells follow the **room’s map snapshot** on a 10-by-10 grid. Each cell MUST be one of: empty/hole, start, task (playable field), or finish, matching the snapshot taken at create from an in-catalog map. Start, unbound task, flipped task, and finish cells MUST use the shipped product tile skins for those kinds (not flat CSS color fills). When synced tourist pieces exist, the board SHALL render every piece at its current synchronized cell (each seated player contributes exactly `touristsPerPlayer` pieces from that map snapshot — see `game/pieces`). Piece placement authority, seating, and movement rules are defined by capabilities `game/pieces` and `game/move`. The legacy fixed tourist ASCII layout MUST NOT be used when a map snapshot is present.

#### Scenario [SC-BOARD-01]: Board geometry and tile kinds

- **GIVEN** the user is authenticated and on the Game screen for a tourist room created from map M
- **WHEN** the board is shown
- **THEN** the 10×10 cells match M’s snapshot grid (start / task / finish / hole) with the product tile skins for those kinds
- **AND** if synced pieces exist, tourist pieces appear on their synchronized cells
- **AND** if no synced pieces exist, no player pieces are rendered on the board

### Requirement: Room ignores hardcoded tourist layout

Creating a room from an in-catalog map whose grid differs from the legacy fixed tourist layout MUST use that map’s snapshot for movement and rendering.

#### Scenario [SC-BOARD-40]: Room ignores hardcoded tourist layout

- **GIVEN** an in-catalog map M whose grid differs from the legacy fixed tourist layout
- **WHEN** a room is created from M and clients view Game in playing
- **THEN** movement and rendering use M’s snapshot
- **AND** the legacy fixed layout MUST NOT override M

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

### Requirement: Flipped task cells show difficulty to everyone

After a task is bound to a still-present task cell, every client in the room MUST render that cell with the **flipped** task skin and that cell’s **difficulty** digit (1, 2, or 3) on the board without requiring the peek modal to be open. The digit MUST use dark ink **`#212121`** (no light text-shadow) so it remains readable on the flipped skin’s beige center. Unbound task cells MUST use the unbound task skin and MUST NOT show a difficulty digit. Removed task holes MUST NOT show a difficulty digit or task skin.

#### Scenario [SC-BOARD-42]: All clients see flipped difficulty

- **GIVEN** task cell C is bound with difficulty 3
- **WHEN** any client views the board
- **THEN** C shows the flipped task skin with a readable digit `3`
- **AND** unbound still-present task cells show the unbound task skin without a digit

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

### Requirement: Shared peek modal with question and answer slots

While a task modal is open after open-on-land (or reopen on an already-open task cell), every client in the room MUST see the shared modal chrome (difficulty header, question, answer slots, answer pool) as previously specified for peek modal layout. Only the seat whose piece opened the modal MAY place answers. A **correct** submit MUST require exact slot order, MUST add **difficulty** knowledge to that piece (not steps), MUST remove the task tile to an empty playable cell, and MUST show toast «Правильно!». Wrong submit MUST KEEP the open bound cell and show toast «Ошибка!». Turn-deadline force-close MUST KEEP the bind and show toast «Не успел!» (`game/move` / `game/presence`). Leave mid-modal and other non-wrong, non-timeout force-closes MUST KEEP the bind without the «Не успел!» toast.

#### Scenario [SC-BOARD-43]: Correct ordered slots grant difficulty steps

- **GIVEN** a seated player has an open task modal with slots filled in the task’s correct order
- **WHEN** that player submits the peek answer
- **THEN** the server accepts the answer as correct
- **AND** that piece gains difficulty knowledge points (not steps) and the task tile is removed
- **AND** every client in the room sees a top-center toast «Правильно!»

#### Scenario [SC-BOARD-44]: Wrong order keeps the same bound task

- **GIVEN** the same setup as SC-BOARD-43 except the slot order is incorrect
- **WHEN** that player submits the peek answer
- **THEN** the server rejects the answer as incorrect
- **AND** the task tile and bind remain (KEEP) with the open knowledge digit visible
- **AND** every client in the room sees a top-center toast «Ошибка!»

#### Scenario [SC-BOARD-88]: Outcome uses top-center toast

- **GIVEN** a task modal resolves as correct, wrong, or turn-timeout
- **WHEN** clients receive peek-close
- **THEN** each client shows a conspicuous top-center toast for that outcome
- **AND** the toast is not limited to the quiet header match-status line alone

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

### Requirement: Peek close announces outcome reason

When the server closes an open task/peek session, it MUST broadcast a peek-close event to every client that includes whether the resolve was correct and, when incorrect, a close reason that distinguishes at least **wrong submit** from **turn-timeout** and from other force-closes. Clients MUST use that payload (not heuristics alone) to choose among the toasts in `game/presence`. When the turn deadline expires while a modal is open (`game/move`), the server MUST emit incorrect + turn-timeout reason and clients MUST show «Не успел!». Leave mid-modal and other non-wrong, non-timeout force-closes MUST NOT be labeled as turn-timeout and MUST NOT use the «Не успел!» toast.

#### Scenario [SC-BOARD-70]: Peek close carries correct flag and reason

- **GIVEN** an open task modal in a tourist room
- **WHEN** the server resolves that modal (correct submit, wrong submit, turn-timeout, or leave/force-close without wrong submit)
- **THEN** every client receives a peek-close message with the correct outcome flag
- **AND** when the resolve is incorrect due to a wrong submit, the message identifies the wrong reason
- **AND** when the resolve is turn-timeout, the message identifies the timeout reason

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

When a task modal closes without a correct submit (wrong order or dismiss without correct), the server MUST keep the tile present and bound, and every client MUST show the flipped/open task skin with the knowledge value digit (**difficulty** 1|2|3). Another free unfinished piece (any seat) that later lands on that open cell MUST reopen the same bound task modal. A **correct** submit MUST grant that piece **difficulty** knowledge points, remove the task tile leaving an empty playable cell (same presentation family as today’s removed task hole), and show a top-center toast «Правильно!» per `game/presence`. Wrong submit MUST show toast «Ошибка!» and MUST NOT grant knowledge.

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
- **AND** every client sees the «Правильно!» top-center toast

### Requirement: Peek answer chips stay reusable without used-state chrome

While a shared peek modal is open, the peeker MUST be allowed to place the same pack answer card into any number of empty answer slots (including every slot). After a card is placed in one slot, its pool mini-card MUST remain selectable for other empty slots. The client MUST NOT disable that pool card solely because it is already used in a slot, and MUST NOT present a distinct «already used» highlight or consumed-vs-resting state that marks the card as spent (including MUST NOT apply the filled-slot primary border to the pool card). Clearing a filled slot (peeker) remains allowed. Spectators still MUST NOT edit slots. Correct submit still requires the slot order to match the bound task’s defined slot order exactly (including when that definition repeats the same answer card).

#### Scenario [SC-BOARD-49]: Same answer card may fill multiple peek slots

- **GIVEN** a seated peeker has an open peek with two or more empty slots and at least one answer-pool card
- **WHEN** that peeker places the same answer card into more than one slot
- **THEN** each of those slots shows that card as a filled mini-card with the filled-slot primary border treatment
- **AND** the answer-pool card for that answer remains available for further empty slots
- **AND** the pool card is not shown in a distinct already-used disabled or highlighted consumed state solely because it appears in a slot
- **AND** other clients still see the placements in realtime and cannot edit slots

### Requirement: Peek modal difficulty header

While the shared peek modal is open, the client MUST show exactly **three** difficulty dots matching the pack task-set difficulty indicator pattern (filled count equals the bound task’s difficulty 1|2|3; **resting** color for all three dots is the single level color for that difficulty — green / amber / red for 1 / 2 / 3 in light and dark, same as pack task-set summary cards — not a per-index rainbow) and a status badge whose copy is **only** the difficulty **word** (лёгкая / средняя / сложная), **without** a `Сложность:` prefix and not the bare digit alone. The dots MUST sit at the **top-left** of the modal content (left edge). The word badge MUST be **horizontally centered** in the modal (not grouped with the dots as a single centered cluster). In the game peek modal the dots MUST be read-only (no hover preview, no click to change difficulty).

#### Scenario [SC-BOARD-50]: Peek header shows left dots and centered word-only badge

- **GIVEN** a shared peek is open for a bound task with difficulty 2
- **WHEN** any client views the peek modal
- **THEN** the header shows three dots with the first two filled in the amber (medium) difficulty color and the third empty outline in that color family
- **AND** those dots are aligned to the left edge of the modal content
- **AND** the badge text is the medium difficulty word only (not `Сложность:` and not only the digit `2`)
- **AND** the badge is horizontally centered in the modal
- **AND** interacting with the dots does not change the task difficulty

### Requirement: Peek slots empty dashed and filled as answer mini-cards

Empty answer slots in the **shared peek modal** MUST use a dashed border treatment and MUST NOT show placeholder slot labels (no «Слот» / «СЛОТ N» text). Empty and filled slots MUST use the same aspect ratio as pack task-set summary cards (portrait ≈156∶206) at approximately **half** the first-shipped peek tile width band (~43 CSS px max-width vs prior ~86). A filled slot MUST render the placed answer as a mini-card visually comparable to the answer-pool cards **and** MUST use a thin solid primary border (`#1976d2`, same weight as the resting pool answer border) so an occupied slot is distinct from empty dashed slots and from pool cards. Clearing a filled slot (peeker) MUST restore the empty dashed state. This slot chrome MUST NOT force the same unlabeled portrait treatment onto content list task tiles outside the peek modal.

#### Scenario [SC-BOARD-51]: Empty slots are blank dashed; filled slots have primary border

- **GIVEN** a peek modal with two slots and at least one answer in the pool
- **WHEN** both slots are empty
- **THEN** each empty slot shows a dashed outline with no slot-label text inside
- **AND** each empty slot’s aspect ratio matches the pack task-set card proportion at the half-size tile band
- **WHEN** the peeker places an answer into the first slot
- **THEN** that slot shows the answer as a mini-card comparable to the pool cards with a thin primary border
- **AND** the second slot remains empty dashed without label text
- **AND** the pool card for that answer does not gain the filled-slot primary border solely because it was placed

### Requirement: Peek answer pool grid with scroll

The peek answer pool MUST present each available answer as a mini-card with the same aspect ratio as pack task-set summary cards, scaled to the **half-size** tile band shared with peek slots. The pool MUST lay out in a grid of at most **four columns** and at most **four visible rows**; when more than sixteen answers exist, the pool MUST scroll vertically within the modal. The pool MUST NOT use the catalog playing-card list tile layout (content|description split tiles).

#### Scenario [SC-BOARD-52]: Answer pool is a scrollable mini-card grid up to 4×4

- **GIVEN** a peek modal whose pack snapshot has more than sixteen answer cards
- **WHEN** the modal is shown
- **THEN** the answer pool shows mini-cards in at most four columns
- **AND** at most four rows are visible without scrolling
- **AND** the remaining cards are reachable by scrolling the pool downward
- **AND** each pool card uses the pack task-set portrait aspect ratio at the half-size tile band

### Requirement: Peek submit is full-width primary in pack-action height rhythm

For the peeker, the peek modal MUST show a full-width primary submit control with the existing answer label. The control’s height MUST follow the pack task-set card action button rhythm (about 28–36 CSS px, canon action height 30) and MAY be proportionally larger than the pack-card action, but MUST remain clearly related to that height. Spectators MUST NOT see an enabled submit control.

#### Scenario [SC-BOARD-53]: Peeker sees full-width primary submit

- **GIVEN** a seated peeker has an open peek with all slots filled
- **WHEN** the modal is shown
- **THEN** a full-width primary submit control with the answer label is available
- **AND** its height is in the pack action-button height band (about 28–36 CSS px)
- **WHEN** a non-peeker client views the same open peek
- **THEN** that client does not get an enabled submit control and sees the spectator hint instead

### Requirement: Peek question uses enlarged type in game modal

In the **game** shared peek modal, the bound task’s question text MUST render at approximately **1.5×** the first-shipped peek question size (about 24–27 CSS px when the prior band was 16–18). This type scale MUST NOT be required for the task compose dialog question control.

#### Scenario [SC-BOARD-54]: Game peek question is enlarged

- **GIVEN** a shared peek is open with a non-empty question string
- **WHEN** any client views the peek modal
- **THEN** the question text uses the enlarged game-peek type band (about 24–27 CSS px)
- **AND** the question remains horizontally centered

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

### Requirement: Board action hits clear the tourist figure

Board peek, rescue, and push affordances that sit in the top-center family above a tourist MUST place their hit targets so the hit does **not** cover that tourist’s figure on the board (same clearance family as strip return in `game/finish`). The product default is about an **8 CSS-pixel** gap between the bottom of the hit and the top of the cell.

#### Scenario [SC-BOARD-71]: Peek rescue and push sit above the sprite

- **GIVEN** the board shows a peek affordance and at least one rescue or push affordance above tourists
- **WHEN** those affordances are rendered
- **THEN** each affordance hit is centered above its tourist
- **AND** none of those hits cover the corresponding tourist figure

### Requirement: Task cells are exclusively task grille or catapult

When phase becomes `playing`, the server SHALL partition still-present map task cells (`*`) into three disjoint sets: **task** (Q&A reward), **grille**, and **catapult**. A cell MUST NOT hold more than one of these. Let `n` be the task-cell count. Grille count MUST be `min(round(n * 0.15), n)`. Catapult count MUST be `min(round(n * 0.20), n - grilleCount)`. Remaining task cells are Q&A tasks. Start and finish cells MUST NOT receive traps or task binds. Placement MUST NOT be predictable from coordinates alone. Create-time density options MUST NOT affect this partition (server ignores them if present).

#### Scenario [SC-BOARD-80]: Seed partitions without overlap

- **GIVEN** a tourist room entering phase `playing` with 48 task cells
- **WHEN** traps and tasks are seeded
- **THEN** exactly `round(48 * 0.15)` = **7** cells are grille-only
- **AND** exactly `min(round(48 * 0.20), 48 - 7)` = **10** cells are catapult-only
- **AND** no cell is both grille and catapult or trap-plus-task
- **AND** the remaining task cells are Q&A task cells

#### Scenario [SC-BOARD-81]: Fixed catapult ratio without create density

- **GIVEN** a tourist room created without client density options
- **WHEN** catapults are seeded at playing start
- **THEN** catapult count is `min(round(n * 0.20), n - grilleCount)` for task-cell count `n`
- **AND** those cells are not also grille or Q&A task cells

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

### Requirement: Clearing a trap leaves an empty cell

When a grille is cleared (rescue, auto-free, or leave) or a catapult is consumed after fling, the underlying cell MUST become an **empty playable** cell (same family as a removed task hole). All-jail MUST NOT clear grilles by itself. The server MUST NOT leave a Q&A task under a former exclusive trap cell.

#### Scenario [SC-BOARD-86]: Grille clear leaves empty playable cell

- **GIVEN** a piece is trapped on an exclusive grille cell
- **WHEN** that grille is cleared by rescue or auto-free
- **THEN** the cell is an empty playable cell with no task bind and no trap
- **AND** every client shows the grille rise/vanish presentation

### Requirement: Grille remaining-turns badge

While a piece is trapped by a grille, every client that shows that piece (board and personal strip chrome) MUST display the remaining auto-free turns as a badge at the **top-right** of the piece chrome on a **red** background. The value MUST match synchronized public `Piece.remainingTurns` from `game/move` (starts at **3**). The badge MUST paint **above** the grille overlay / strip chrome grille (`z-index` higher than the grille artwork) so the digit remains readable. When the grille clears, the badge MUST disappear with the grille. This badge is **public** (all clients). Own carried-knowledge chrome (`game/presence`) lives on the strip only and MUST NOT replace the grille counter.

#### Scenario [SC-BOARD-85]: Trapped piece shows red top-right counter

- **GIVEN** a piece is trapped with remaining turns 3
- **WHEN** any client views the board
- **THEN** that piece shows a red-background counter at top-right with value 3
- **AND** the counter remains visible above the grille overlay
- **AND** other seats see the same red counter (public sync)

### Requirement: Hidden catapults are invisible until triggered

Until a piece lands on a cell that still holds an unspent catapult (including via push or return-from-finish), clients MUST NOT render that catapult. Spectators and other seats MUST NOT learn catapult locations from synced state before reveal.

#### Scenario [SC-BOARD-22]: Board shows no catapult before land

- **GIVEN** phase is `playing` and hidden catapults exist on some task cells
- **WHEN** the board is shown before any piece has landed on those cells
- **THEN** no catapult artwork is shown on those cells

### Requirement: Land on catapult cell before overlay for every viewer

When a piece triggers an unspent catapult (via move, push, return-from-finish, or post-rescue resolve while free on the cell), every client that displays the board — including spectators and non-acting seats — MUST complete a visual arrival onto that catapult cell with ordinary piece-travel sense **before** starting the catapult overlay. Under server-paced sync, piece coordinates SHOULD remain on the catapult cell until relocate; if a client still observes early relocate, it MUST synthesize arrival. If the piece is already visually on that cell and no arrival animation is in progress (e.g. post-rescue on the same cell), clients MUST NOT invent a fake step and MAY start the overlay immediately after any in-flight arrival animation ends.

#### Scenario [SC-BOARD-28]: Arrival completes before catapult overlay

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult on a cell (including when the acting player is not the local viewer)
- **WHEN** clients present that trigger
- **THEN** every client finishes visual arrival onto that catapult cell before the catapult overlay begins
- **AND** spectators use the same order as the acting player’s clients

### Requirement: Successful catapult presentation then piece travel

When a piece triggers an unspent catapult that has at least one legal fling destination, every client that displays the board MUST, **after** SC-BOARD-28 arrival: (1) show the intact catapult appear and fully disappear on that cell (about **1500 ms** total presentation sense); (2) while that overlay is visible, keep the piece visually on the catapult cell; (3) **only after** the overlay is fully gone, animate the piece to the fling destination with the same travel sense as an ordinary move (`MOVE_ANIM_MS` order), following authoritative relocate when it arrives for that hop. After a successful consume, the catapult MUST NOT remain visible. Cleared/spent catapults MUST NOT remain visible afterward. The server paced presentation budget for a successful catapult hop MUST match that about **1500 ms** client sense (`game/move`) — production default `CATAPULT_ANIM_MS = 1500` (and the same value restored by `resetTrapPresentationBudgets`).

#### Scenario [SC-BOARD-23]: Successful catapult vanishes before travel

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult that has at least one legal fling destination
- **WHEN** that catapult is revealed and consumed
- **THEN** every client shows the intact catapult appear-then-vanish on that cell lasting about 1500 ms only after arrival on that cell completed
- **AND** the catapult artwork is gone afterward
- **AND** the piece travel to the fling destination starts only after that vanish completes

#### Scenario [SC-BOARD-25]: Piece stays on catapult cell during overlay

- **GIVEN** a successful catapult reveal is presenting on a cell
- **WHEN** the overlay is still visible
- **THEN** every client shows that piece on the catapult cell (not yet at the fling destination)

#### Scenario [SC-BOARD-26]: Catapult chain is sequential

- **GIVEN** a fling lands a piece onto another cell that still holds an unspent catapult
- **WHEN** that next catapult resolves for presentation
- **THEN** clients complete the prior catapult vanish and prior fling travel, then arrival onto the next catapult cell, before starting the next catapult overlay
- **AND** there is no product cap on chain length beyond one-shot consume per catapult

#### Scenario [SC-BOARD-31]: Every hop animates including final fling to center

- **GIVEN** a multi-hop catapult chain whose last fling destination is a center cell (finish)
- **WHEN** clients present that pipeline
- **THEN** each hop shows land → overlay → travel (or finish travel on the last hop after its vanish)
- **AND** the piece MUST NOT disappear from an intermediate catapult cell without travel
- **AND** finish travel to center starts only after the last successful catapult vanish

### Requirement: Broken catapult hold timeline

When a piece triggers an unspent catapult with **no** legal fling destination, every client that displays the board MUST, **after** SC-BOARD-28 arrival, present: intact appear → hold about **450 ms** intact → switch to broken artwork → hold about **450 ms** broken → vanish (vanish fade about **300 ms**). The piece MUST remain on that cell (no fling travel). Cleared/spent catapults MUST NOT remain visible afterward. The server paced presentation budget for a broken catapult hop MUST match the sum of those holds plus vanish (`game/move`) — production default `CATAPULT_BROKEN_PRESENTATION_MS = 450 + 450 + 300` (**1200** ms; and the same value restored by `resetTrapPresentationBudgets`).

#### Scenario [SC-BOARD-24]: Broken catapult 300+300 then vanish

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult with no legal fling destination
- **WHEN** the broken presentation runs
- **THEN** every client shows intact appear, about 450 ms intact, broken art about 450 ms, then vanish (about 300 ms fade), only after arrival on that cell completed
- **AND** the catapult artwork is gone afterward
- **AND** the piece remains on that cell

### Requirement: Catapult and grille overlays stay readable on tile skins

Revealed catapult overlays and revealed grille overlays on the board MUST remain visually separable from the underlying task (and other) tile skins so players can tell the trap art from the cell face. The same separation MUST apply to the personal strip chrome grille that mirrors a trapped piece (`game/pieces`). Separation MUST use a non-filling edge treatment — CSS `filter: drop-shadow(...)` on `.catapult-overlay`, `.grille-overlay`, and `.my-tourist-chrome-grille` (neutral multi-layer light/dark edge shadow) — so grille bar openings stay alpha-clear. Clients MUST NOT paint an opaque underlay behind the grille that fills those openings and hides the tourist.

#### Scenario [SC-BOARD-58]: Overlays read on skinned tiles without hiding tourist under grille

- **GIVEN** phase is `playing`, task cells use product tile skins, and a piece triggers a catapult or holding grille on a task cell (or a strip slot shows chrome grille for a trapped own piece)
- **WHEN** the overlay or chrome grille is visible
- **THEN** the catapult or grille artwork remains distinguishable from the tile skin behind it via CSS `filter: drop-shadow` (or equivalent non-filling edge treatment) on the overlay / chrome-grille element
- **AND** for a grille over a tourist, the tourist remains visible through the grille openings
- **AND** no opaque underlay fills those grille openings

### Requirement: Grille presentation follows the same sequential trap timeline

When a paced land hop resolves a grille after zero or more prior catapult hops in the same pipeline, every client MUST present the grille drop on that grille’s cell only as part of that hop — **not** in parallel with earlier catapult overlays/travels, and not on a cell the piece has not yet visually reached in the sequence. Clients MUST treat catapult hops and the subsequent grille hop as one sequential trap timeline.

#### Scenario [SC-BOARD-29]: Grille drop waits for prior catapult hops

- **GIVEN** a piece is flung through one or more catapults and then lands on a cell whose grille traps it
- **WHEN** clients present that pipeline
- **THEN** no grille drop artwork appears on the final cell until prior catapult vanish and fling travel hops for that pipeline have completed
- **AND** the grille drop runs only when presenting that grille land hop

#### Scenario [SC-BOARD-30]: No early empty grille on the final cell

- **GIVEN** the same multi-hop pipeline as SC-BOARD-29
- **WHEN** an intermediate catapult overlay is still presenting
- **THEN** clients MUST NOT show a holding grille on the eventual trap cell ahead of the piece’s arrival in the sequence

### Requirement: Board is non-interactive during board animations

While any board presentation animation is running for the local client (piece land/move travel, rescue/push approach, grille drop/rise including the pipeline settle into hold so there is no unlock flash before the grille drop, catapult overlay including broken holds, deferred fling travel after catapult vanish, finish travel/disappear), **and** during any gap between consecutive presentation steps of the same pipeline (for example after move travel ends and before catapult/grille presentation starts after a successful move), the seated current player MUST NOT be able to click the board to select pieces, submit moves, peek, rescue, push, return-from-finish, or end-turn board targets. After grille **drop has settled** into a static hold, the board/strip MUST unlock again so rescue (and other legal turn actions) remain possible while the holding grille stays visible. The same lock MUST apply to the personal tourist strip beside the own presence marker (select / return affordances) for the duration of the busy window above. Say send from the own presence avatar MUST remain available under existing `game/say` rules. Non-board chrome outside board/strip (e.g. leave/status in the app header) is out of this requirement’s scope unless already gated elsewhere. Turn chrome MAY still show the acting seat until the server advances after pipeline idle (`game/move`).

#### Scenario [SC-BOARD-27]: Clicks blocked while board animates

- **GIVEN** the local seated player’s turn and a board animation is in progress (including land-before-catapult, catapult overlay, pending fling travel, grille drop/rise or the brief window between move travel and the next trap presentation — not a settled static grille hold after drop)
- **WHEN** the player attempts a board click that would otherwise submit or select
- **THEN** that board interaction is ignored until the animation sequence finishes
- **AND** after grille drop has settled into hold, rescue remains available while the grille is still shown

#### Scenario [SC-BOARD-32]: Strip tourists locked while board is busy

- **GIVEN** the local seated player’s turn and board presentation is busy per SC-BOARD-27
- **WHEN** the player activates a personal strip tourist or return affordance
- **THEN** that strip interaction is ignored until the board is no longer busy
- **AND** the say send affordance on the own presence avatar remains usable per `game/say`
