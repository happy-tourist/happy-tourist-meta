## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-13 | modified (peek affordance ~2× illustrated glyph / larger hit; still no solid colored chrome) |
| SC-BOARD-14 | modified (same enlarged peek family after keep-focus) |
| SC-BOARD-43 | modified (correct close → room-wide header flash «Правильно!» via presence; KEEP/remove rules unchanged) |
| SC-BOARD-44 | modified (wrong submit → room-wide header flash «Ошибка!»; KEEP bind unchanged) |
| SC-BOARD-70 | added (peekClose carries close reason so clients distinguish wrong vs turn-timeout) |

## MODIFIED Requirements

### Requirement: Eye affordance to open a peek

While it is the user’s multiplayer or solo turn and a peek is currently allowed for one of their unfinished pieces on a still-present task cell, activating that piece MUST offer a peek affordance to open the peek modal. The peek affordance MUST use the **illustrated board-action peek icon** (not a Material eye glyph) and MUST be shown **centered above** that tourist on the board (same top-center family as strip return and board rescue/push affordances). The affordance MUST NOT paint a solid colored circular chrome behind the icon (no solid fill and no medallion-like shadow). The illustrated glyph MUST render at about **twice** the prior board-action glyph size (product sense ~36 CSS pixels), and the hit target MUST be enlarged in the same family (~48–56 CSS pixels) while remaining touchable. When local selection is already kept on such a piece after a move (`game/move` keep-focus), the peek affordance MUST appear without requiring another activation click. Clients that are not allowed to peek MUST NOT show that affordance as a way to open a peek. A removed-task hole under the piece MUST NOT offer the peek affordance.

#### Scenario [SC-BOARD-13]: Current player sees eye on a peekable tourist

- **GIVEN** it is the user’s turn with an allowed peek and an own unfinished piece on a still-present task cell
- **WHEN** the user activates that piece
- **THEN** a peek affordance with the illustrated peek icon is available centered above that tourist to open the peek
- **AND** the icon glyph is about twice the prior board-action glyph size and the hit target is enlarged accordingly
- **AND** the affordance has no solid colored circular chrome behind the icon
- **AND** other clients do not gain that peek affordance for the user’s piece

#### Scenario [SC-BOARD-14]: Eye after move without re-select when still on a present task tile

- **GIVEN** it is the user’s turn with peeks remaining (or solo infinite peeks) and after a successful move the same unfinished piece remains selected on a still-present task cell
- **WHEN** the move animation finishes and interaction returns
- **THEN** the peek affordance (illustrated peek icon, enlarged ~2× family) is available on that piece without requiring another selection click
- **AND** if the cell under the piece is a removed-task hole or not a task cell, the peek affordance is not shown for that piece

### Requirement: Shared peek modal with question and answer slots

While it is a seated client’s own turn and a peek is allowed for an own unfinished free piece on a still-present task cell, opening a peek MUST show a modal synchronized so **every** client in the room sees it: a difficulty header (three difficulty dots plus a word difficulty badge — see SC-BOARD-50), the bound task’s question (see SC-BOARD-54), ordered answer slots above, and below them an answer pool of mini-cards from that pack’s live/snapshot answer cards (see SC-BOARD-51/52). Only the peeking seat MAY place answers into slots; other clients MUST see placements in realtime and MUST NOT edit slots. Closing without a correct submit (wrong order, leave cell, turn timeout, end-turn with open peek) MUST keep the tile and keep the same bound task. A **correct** submit MUST require the slot order to match the task’s defined slot order exactly; on success the server MUST add **difficulty** steps to that seat’s private steps budget and MUST remove the task tile. Forced incorrect closes and wrong submits MUST NOT remove the tile. When a peek closes as **correct**, every client in the room MUST show the match-status flash «Правильно!» per `game/presence`. When a peek closes because the peeker submitted an **incorrect** slot order, every client MUST show the match-status flash «Ошибка!» per `game/presence`. When a peek closes because the **turn deadline** elapsed while the peek was open, every client MUST show «Не успел!» per `game/presence` (and `game/move`). Other force-closes (for example leave mid-peek) MUST NOT use the «Не успел!» flash.

#### Scenario [SC-BOARD-43]: Correct ordered slots grant difficulty steps

- **GIVEN** a seated player has an open peek with slots filled in the task’s correct order
- **WHEN** that player submits the peek answer
- **THEN** the server accepts the answer as correct
- **AND** that seat gains difficulty steps and the task tile is removed per existing rules
- **AND** every client in the room sees the match-status flash «Правильно!» for about ten seconds

#### Scenario [SC-BOARD-44]: Wrong order keeps the same bound task

- **GIVEN** the same setup as SC-BOARD-43 except the slot order is incorrect
- **WHEN** that player submits the peek answer
- **THEN** the server rejects the answer as incorrect
- **AND** the task tile and bind remain (KEEP)
- **AND** every client in the room sees the match-status flash «Ошибка!» for about ten seconds

#### Scenario [SC-BOARD-46]: Everyone sees the peek modal and placements

- **GIVEN** a seated player opens a peek on a task cell
- **WHEN** that player places answer cards into slots
- **THEN** every other client in the room sees the same modal chrome and placements
- **AND** those other clients cannot edit the slots

#### Scenario [SC-BOARD-47]: Leave mid-peek keeps bind

- **GIVEN** a seated player has an open peek on task cell C
- **WHEN** that player’s piece leaves C or the peek is force-resolved as incorrect without a wrong submit or turn-timeout flash path
- **THEN** the bind and flipped digit remain on C (KEEP)
- **AND** the task tile is not removed

## ADDED Requirements

### Requirement: Peek close announces outcome reason

When the server closes an open peek session, it MUST broadcast a peek-close event to every client that includes whether the resolve was correct and, when incorrect, a close reason that distinguishes at least **wrong submit** from **turn-timeout** force-close. Clients MUST use that payload (not heuristics alone) to choose among the header flashes in `game/presence`. Leave mid-peek and other non-timeout force-closes MUST NOT be labeled as turn-timeout.

#### Scenario [SC-BOARD-70]: Peek close carries correct flag and reason

- **GIVEN** an open peek in a tourist room
- **WHEN** the server resolves that peek (correct submit, wrong submit, or turn-timeout force-close)
- **THEN** every client receives a peek-close message with the correct outcome flag
- **AND** when the resolve is incorrect due to turn timeout, the message identifies the timeout reason
- **AND** when the resolve is incorrect due to a wrong submit, the message identifies the wrong reason
