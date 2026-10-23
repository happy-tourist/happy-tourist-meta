## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-09 | modified (finish indicator = illustrated flag icon) |
| SC-FINISH-10 | unchanged (carried) |
| SC-FINISH-13 | modified (return = illustrated icon; no green circular chrome) |
| SC-FINISH-16 | unchanged (carried) |

## MODIFIED Requirements

### Requirement: Finished strip slots and place chrome

For a seated client, each personal strip slot whose matching piece is finished MUST show a finish indicator at the top-right of that slot and MUST NOT be usable to select or submit a **move**. The finish indicator MUST use the **illustrated board-action flag icon** (not a Material flag glyph). When return-from-finish is **not** currently available for that side (not the user’s interactive turn, steps < 1, finish place ≠ 0, or no legal free non-hole ring cell), that finished slot MUST be visually dimmed to signal that no action is available. When return **is** available for that side, the slot MUST NOT be dimmed solely because the piece is finished. Sides for unfinished pieces keep existing selection behavior while it is that seat’s turn and the seat is not finished. After the seat has a finish place, all four strip slots MUST remain visible with finish indicators (typically dimmed because return is unavailable).

#### Scenario [SC-FINISH-09]: Finished strip slot is inactive with indicator

- **GIVEN** a seated player has finished exactly one of four pieces and return is not available for that side
- **WHEN** that player views the personal strip
- **THEN** the finished piece’s strip slot shows a finish indicator (illustrated flag icon) at the top-right
- **AND** that slot is visually dimmed
- **AND** activating that finished slot does not select a piece, open a return modal, or submit a move

#### Scenario [SC-FINISH-10]: All strip slots stay after full finish

- **GIVEN** a seated player has finish place > 0 and all four pieces finished
- **WHEN** that player views the personal strip
- **THEN** all four strip slots remain visible with finish indicators

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST show a return affordance centered above that tourist in the personal strip (same visual family as board rescue/push affordances). The affordance MUST use the **illustrated board-action return icon** and MUST NOT paint a solid colored (including green) circular chrome behind the icon (no solid fill and no medallion-like `box-shadow`); the hit target remains ~32 CSS pixels. Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above) and MUST NOT show the return affordance. Activating the finished strip slot body (the tourist image area) MUST NOT open a confirmation dialog and MUST NOT enter return-mode by itself — only the return affordance starts return. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a return affordance with the illustrated return icon is shown centered above that tourist (same visual family as board rescue/push; no solid colored circular chrome / medallion-like `box-shadow`)
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation
