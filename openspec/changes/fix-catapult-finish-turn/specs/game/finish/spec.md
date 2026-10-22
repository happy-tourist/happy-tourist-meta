# Delta: game/finish

Связано с main `openspec/specs/game/finish/spec.md` (place assignment / catapult→center) и delta `game/move` SC-MOVE-96 (turn release). Этот change уточняет: полный финиш через катапульту назначает место **и** не оставляет ход на финишере при наличии eligible соперника.

## Traceability

| ID | Coverage |
|----|----------|
| SC-FINISH-03 | covered-by-reuse (main — first full finisher place + modal; ordinary move onto center) |
| SC-FINISH-04 | covered-by-reuse (main — later full finisher gets next place; unchanged) |
| SC-FINISH-05 | covered-by-reuse (main — room stays open when all seats finished; unchanged) |
| SC-FINISH-19 | covered-by-reuse (main — catapult fling onto center finishes the piece) |
| SC-FINISH-21 | covered (server mocha — fourth piece via catapult→center: place + turn leave; client modal covered-by-reuse) |

## MODIFIED Requirements

### Requirement: Completing all four pieces assigns finish place

When a seated player’s fourth piece becomes finished, the server SHALL assign that seat a finish place equal to one plus the number of seats that already hold a finish place in this room (monotonic order of full finishes). The place MUST be synchronized. The client of that seat MUST show a modal congratulating the player on finishing in that place (product Russian sense: first / second / …). Closing the modal MUST leave the player in the room as a finished seat. The room MUST NOT disconnect or dispose solely because one or all seats became finished. When that fourth-piece finish happens via a catapult fling onto a center cell under paced trap rules (`game/move` / `game/board`), the same place assignment and modal rules MUST apply; after the trap pipeline is idle, turn eligibility MUST follow `game/move` (finished seat MUST NOT keep the current turn while another eligible seat remains — SC-MOVE-96).

#### Scenario [SC-FINISH-03]: First full finisher gets place one

- **GIVEN** a tourist room in phase `playing` where no seat yet has a finish place, and a seated player has exactly three finished pieces and one unfinished piece adjacent to a free center cell
- **WHEN** that player’s move finishes the fourth piece
- **THEN** that seat’s synchronized finish place is `1`
- **AND** that player’s client shows a place congratulation modal
- **AND** the seat remains in the room after the modal is closed

#### Scenario [SC-FINISH-04]: Later full finisher gets the next place

- **GIVEN** a tourist room where one seat already has finish place `1` and another seated player finishes their fourth piece
- **WHEN** that fourth piece becomes finished
- **THEN** that second seat’s finish place is `2`
- **AND** that player’s client shows a congratulation modal for place `2`

#### Scenario [SC-FINISH-05]: Room stays open when all seats are finished

- **GIVEN** every seated player in the room has a finish place
- **WHEN** no further consented leave or grace timeout has removed seats
- **THEN** the room remains available
- **AND** seated clients are not forcibly disconnected solely because all seats are finished

#### Scenario [SC-FINISH-21]: Catapult fourth-piece finish assigns place and releases turn

- **GIVEN** a tourist room in phase `playing` with at least two eligible seated players and no seat yet has a finish place
- **AND** seated player A has exactly three finished pieces and one unfinished free piece (`finishPlace` still `0`) whose paced catapult fling lands on a free center cell
- **WHEN** that catapult relocate finishes A’s fourth piece (place assigned mid-pipeline via center finish) and the presentation pipeline then becomes idle
- **THEN** A’s synchronized finish place is `1`
- **AND** A’s client shows the place congratulation modal
- **AND** after the trap pipeline is idle the synchronized current turn is not A while another eligible seat remains
