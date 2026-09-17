## Purpose

Delta этого change: return с финиша снова выводит piece в игру. Полный finish place / strip flag — main `game/finish`; стоимость/кольцо — `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-12 | done (server mocha — return clears finished) |
| SC-FINISH-13 | done (client UX — return control beside flag) |
| SC-FINISH-14 | done (server mocha — strip selectable after return) |

Related: return geometry and steps — `game/move`.

## ADDED Requirements

### Requirement: Return from finish puts the piece back in play

When the server accepts a legal return-from-finish for a seat with finish place **0**, the targeted piece MUST become unfinished and MUST occupy the chosen ring cell. That piece MUST again count toward board occupancy and MUST be eligible for later moves under `game/move`. The strip slot for that side MUST no longer show a finished-only lock for selection once the piece is unfinished (subject to turn rules).

#### Scenario [SC-FINISH-12]: Returned piece is unfinished on the board

- **GIVEN** a seated player with finish place 0 has a finished piece and successfully returns it onto a legal ring cell
- **WHEN** synced state updates
- **THEN** that piece’s finished flag is false
- **AND** the piece occupies the chosen cell
- **AND** a later legal move of that piece MAY be accepted on a subsequent action of that turn or a later turn

#### Scenario [SC-FINISH-14]: Returned strip slot is usable again on turn

- **GIVEN** the same seat as SC-FINISH-12 still has the current turn after return
- **WHEN** that player activates the strip slot for the returned side
- **THEN** the slot is treated as an unfinished piece slot for selection
- **AND** it does not remain locked as a finished-only slot

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each personal strip slot whose piece is finished MUST offer a return control adjacent to the existing finish indicator when at least one legal ring cell exists. Activating return MUST enter ring-target selection on the board; choosing a legal cell MUST submit the return. When steps are 0 or no legal ring cell exists, the return control MUST NOT be actionable.

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished strip slot, and at least one legal free non-hole ring cell
- **WHEN** the user views the personal strip
- **THEN** a return control is available next to that slot’s finish indicator
- **AND** activating it allows choosing a highlighted legal ring cell to place the tourist
