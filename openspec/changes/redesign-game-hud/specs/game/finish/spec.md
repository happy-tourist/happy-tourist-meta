# game/finish Delta

Related: compact tourist chrome / picker — `game/pieces`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-09 | pending (client finish indicator on chip + menu) |
| SC-FINISH-10 | pending (client chip/menu keep four sides after full finish) |
| SC-FINISH-13 | pending (client return only in picker menu) |
| SC-FINISH-14 | covered-by-reuse (server — strip selectable after return) |

## MODIFIED Requirements

### Requirement: Finished strip slots and place chrome

For a seated client, each personal tourist side whose matching piece is finished MUST show a finish indicator on the corresponding mini slot of the compact HUD chip and on the corresponding full-size slot in the picker menu, and MUST NOT be usable to select or submit a move. Sides for unfinished pieces keep existing selection behavior from the picker or board while it is that seat’s turn and the seat is not finished. After the seat has a finish place, all four sides MUST remain represented on the chip and in the menu with finish indicators.

#### Scenario [SC-FINISH-09]: Finished strip slot is inactive with indicator

- **GIVEN** a seated player has a finished piece for one side
- **WHEN** that player views the compact tourist chip and opens the picker menu
- **THEN** that side’s mini slot and full-size menu slot show a finish indicator
- **AND** that side is not used for move selection

#### Scenario [SC-FINISH-10]: All strip slots stay after full finish

- **GIVEN** a seated player has finish place > 0 and all four pieces finished
- **WHEN** that player views personal tourist chrome
- **THEN** the compact chip and picker menu still represent all four sides of that player’s tourist kind with finish indicators

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST offer a return control adjacent to the finish indicator **only inside the full-size picker menu** (not on the compact HUD chip). Activating return MUST enter ring-target selection on the board; choosing a legal cell MUST submit the return. When steps are 0 or no legal ring cell exists, the return control MUST NOT be actionable. The compact chip MUST still show the finish indicator for that side without a return control.

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user opens the tourist picker menu from the compact chip
- **THEN** a return control is available next to that side’s finish indicator in the menu
- **AND** the compact chip shows the finish indicator for that side without a return control
