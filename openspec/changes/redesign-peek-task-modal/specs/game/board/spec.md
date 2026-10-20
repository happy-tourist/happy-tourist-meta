## Traceability

| SC-ID | Coverage target |
|-------|-----------------|
| SC-BOARD-46 | modified (modal chrome — client) |
| SC-BOARD-49 | modified (pool tiles reuse — client) |
| SC-BOARD-50 | pending (client — difficulty header centered word-only) |
| SC-BOARD-51 | pending (client — empty/filled slots half size + primary filled border) |
| SC-BOARD-52 | pending (client — pool grid 4×4 half-size) |
| SC-BOARD-53 | covered (submit primary full-width — first apply) |
| SC-BOARD-54 | pending (client — game question ×1.5 type) |

Related unchanged server peek rules: SC-BOARD-41…45, 47–48 (deck bind, flipped digit, correct/wrong resolve, leave mid-peek, cycle). **Apply scope for this change:** client chrome only — do **not** change room messages (`peek` / `peekPlace` / `peekSubmit`), schema (`peekDifficulty` / `peekPlacements` / `answerCards` / …), or resolve algorithms in `happy-tourist-server`. Scenarios SC-BOARD-43/44/47 appear under the MODIFIED parent requirement only so chrome (SC-BOARD-50…54 / 46/49) stays attached to the same modal contract — their **server** THEN clauses are baseline restatement, not new BE work (no server tests/tasks for this change). Compose parity: `content/packs` SC-PACK-260/263/276+.

## MODIFIED Requirements

### Requirement: Shared peek modal with question and answer slots

While it is a seated client’s own turn and a peek is allowed for an own unfinished free piece on a still-present task cell, opening a peek MUST show a modal synchronized so **every** client in the room sees it: a difficulty header (three difficulty dots plus a word difficulty badge — see SC-BOARD-50), the bound task’s question (see SC-BOARD-54), ordered answer slots above, and below them an answer pool of mini-cards from that pack’s live/snapshot answer cards (see SC-BOARD-51/52). Only the peeking seat MAY place answers into slots; other clients MUST see placements in realtime and MUST NOT edit slots. Closing without a correct submit (wrong order, leave cell, turn timeout, end-turn with open peek) MUST keep the tile and keep the same bound task. A **correct** submit MUST require the slot order to match the task’s defined slot order exactly; on success the server MUST add **difficulty** steps to that seat’s private steps budget and MUST remove the task tile (**unchanged** room behavior — not part of this change’s client apply). Forced incorrect resolves MUST keep bind and tile. The peeker MUST have a full-width primary submit control labeled for answering; spectators MUST see a non-editable hint instead of that control (SC-BOARD-53).

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
- **WHEN** that player places an answer into a slot
- **THEN** every other client in the room observes the same question, slots, and that placement
- **AND** those other clients cannot change the slots
- **AND** every client sees the same difficulty header, empty/filled slot chrome, and answer-pool mini-cards

#### Scenario [SC-BOARD-47]: Leave mid-peek keeps bind

- **GIVEN** a seated player has an open peek on bound cell C
- **WHEN** that player’s piece leaves C or the peek is force-resolved as incorrect
- **THEN** C remains present with the same bound task
- **AND** a later peek on C shows the same question

### Requirement: Peek answer chips stay reusable without used-state chrome

While a shared peek modal is open, the peeker MUST be allowed to place the same pack answer card into any number of empty answer slots (including every slot). After a card is placed in one slot, its pool mini-card MUST remain selectable for other empty slots. The client MUST NOT disable that pool card solely because it is already used in a slot, and MUST NOT present a distinct «already used» highlight or consumed-vs-resting state that marks the card as spent (including MUST NOT apply the filled-slot primary border to the pool card). Clearing a filled slot (peeker) remains allowed. Spectators still MUST NOT edit slots. Correct submit still requires the slot order to match the bound task’s defined slot order exactly (including when that definition repeats the same answer card).

#### Scenario [SC-BOARD-49]: Same answer card may fill multiple peek slots

- **GIVEN** a seated peeker has an open peek with two or more empty slots and at least one answer-pool card
- **WHEN** that peeker places the same answer card into more than one slot
- **THEN** each of those slots shows that card as a filled mini-card with the filled-slot primary border treatment
- **AND** the answer-pool card for that answer remains available for further empty slots
- **AND** the pool card is not shown in a distinct already-used disabled or highlighted consumed state solely because it appears in a slot
- **AND** other clients still see the placements in realtime and cannot edit slots

## ADDED Requirements

### Requirement: Peek modal difficulty header

While the shared peek modal is open, the client MUST show a **centered** header with exactly **three** difficulty dots matching the pack task-set difficulty indicator pattern (filled count equals the bound task’s difficulty 1|2|3; **resting** color for all three dots is the single level color for that difficulty — green / amber / red for 1 / 2 / 3 in light and dark, same as `PackTaskSetCardTile` — not a per-index rainbow) and a status badge whose copy is **only** the difficulty **word** (лёгкая / средняя / сложная), **without** a `Сложность:` prefix and not the bare digit alone. In the game peek modal the dots MUST be read-only (no hover preview, no click to change difficulty).

#### Scenario [SC-BOARD-50]: Peek header shows three difficulty dots and word-only badge centered

- **GIVEN** a shared peek is open for a bound task with difficulty 2
- **WHEN** any client views the peek modal
- **THEN** the header shows three dots with the first two filled in the amber (medium) difficulty color and the third empty outline in that color family
- **AND** the badge text is the medium difficulty word only (not `Сложность:` and not only the digit `2`)
- **AND** the dots+badge group is horizontally centered in the modal
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
