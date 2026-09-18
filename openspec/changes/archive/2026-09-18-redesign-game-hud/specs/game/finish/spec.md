# game/finish Delta

Related: personal strip — `game/pieces`; return geometry / step cost — `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-09 | implemented (client finish indicator on strip; conditional dim) |
| SC-FINISH-10 | implemented (client strip keeps four sides after full finish) |
| SC-FINISH-13 | implemented (client modal return flow) |
| SC-FINISH-14 | covered-by-reuse (server — strip selectable after return) |
| SC-FINISH-15 | implemented (client return travel animation from nearest center) |

## MODIFIED Requirements

### Requirement: Finished strip slots and place chrome

For a seated client, each personal strip slot whose matching piece is finished MUST show a finish indicator at the top-right of that slot and MUST NOT be usable to select or submit a **move**. When return-from-finish is **not** currently available for that side (not the user’s interactive turn, steps < 1, finish place ≠ 0, or no legal free non-hole ring cell), that finished slot MUST be visually dimmed to signal that no action is available. When return **is** available for that side, the slot MUST NOT be dimmed solely because the piece is finished. Sides for unfinished pieces keep existing selection behavior while it is that seat’s turn and the seat is not finished. After the seat has a finish place, all four strip slots MUST remain visible with finish indicators (typically dimmed because return is unavailable).

#### Scenario [SC-FINISH-09]: Finished strip slot is inactive with indicator

- **GIVEN** a seated player has finished exactly one of four pieces and return is not available for that side
- **WHEN** that player views the personal strip
- **THEN** the finished piece’s strip slot shows a finish indicator at the top-right
- **AND** that slot is visually dimmed
- **AND** activating that finished slot does not select a piece, open a return modal, or submit a move

#### Scenario [SC-FINISH-10]: All strip slots stay after full finish

- **GIVEN** a seated player has finish place > 0 and all four pieces finished
- **WHEN** that player views the personal strip
- **THEN** all four strip slots remain visible with finish indicators

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST be actionable via a **click on that strip slot** (no separate undo control on the slot). Activating such a slot MUST open a confirmation dialog asking whether to return the tourist to the field (product Russian sense «Вернуть на поле?»). Confirming MUST enter ring-target selection on the board; dismissing MUST leave selection and return-mode unchanged. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when the dialog opens or ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above). Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user activates that finished strip slot
- **THEN** a confirmation dialog asks whether to return that tourist to the field
- **AND** after the user confirms, legal ring cells are highlighted like ordinary move targets
- **AND** choosing a highlighted ring cell submits the return
- **AND** no separate undo icon control is required on the strip slot

## ADDED Requirements

### Requirement: Return-from-finish travel animation

When the server accepts a legal return-from-finish, every client that displays the board MUST animate that piece traveling from the **nearest** of the four center finish cells to the chosen ring cell (Chebyshev distance to the ring cell; ties broken by lower row, then lower column), using the same travel timing family as ordinary piece moves. The animation MUST be visible to all clients in the room. Input locking for the submitting client during travel follows the same rules as move travel (`game/move`).

#### Scenario [SC-FINISH-15]: Everyone sees return leave the finish block

- **GIVEN** a seated player successfully returns a finished piece onto a legal ring cell
- **WHEN** clients update the board
- **THEN** every client shows that piece animating from the nearest center cell to that ring cell
- **AND** after the animation the piece occupies the ring cell as an unfinished piece
