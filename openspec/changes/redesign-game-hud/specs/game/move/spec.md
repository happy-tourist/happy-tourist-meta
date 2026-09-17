# game/move Delta

Related: finish return UX — `game/finish`; center block presentation — `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-11 | pending (client selection; return-mode cancel on reselect) |
| SC-MOVE-12 | pending (client legal targets; return targets same red outline) |
| SC-MOVE-13 | pending (client reselection cancels return-mode) |
| SC-MOVE-63 | pending (client center-block click → nearest legal center cell) |
| SC-MOVE-57 | covered-by-reuse (server — return costs 1 step on accept) |

## MODIFIED Requirements

### Requirement: Local selection and move hints for the current player only

While it is a seated client’s own turn, that client SHALL be able to select one of their own pieces by activating it on the board or the matching slot in their personal strip. The selected piece’s cell MUST show a white selection outline, and every currently legal destination for that piece MUST show a red outline. Legal **return-from-finish** ring cells shown while return-mode is active MUST use that **same** red destination outline presentation (not a distinct orange-only chrome). Selection and red destination hints MUST be local UX only (not authoritative truth) and MUST NOT be shown to other seated players or spectators. The current player MAY change selection among their own pieces until a move is submitted; changing selection MUST cancel return-mode without spending a step. Clients that are not the current-turn seated player MUST NOT show white or red move chrome and MUST NOT submit a move as a result of board activation. Activating a red destination cell MUST submit the corresponding move for the selected piece (or the return when return-mode is active).

#### Scenario [SC-MOVE-11]: Current player sees white selection chrome

- **GIVEN** it is the user’s turn as a seated player with at least one piece
- **WHEN** the user selects one of their pieces on the board or in the personal strip
- **THEN** that piece’s cell shows a white selection outline on that user’s client
- **AND** other clients do not show that white selection outline as a shared requirement

#### Scenario [SC-MOVE-12]: Current player sees red legal destinations

- **GIVEN** it is the user’s turn and one of their pieces is selected
- **WHEN** legal destinations exist for that piece under the move rules
- **THEN** those destination cells show a red outline on that user’s client only
- **AND** occupied or non-playable neighbors are not outlined as destinations
- **AND** when return-mode is active with legal ring cells, those cells show the same red outline presentation

#### Scenario [SC-MOVE-13]: Reselection before commit

- **GIVEN** it is the user’s turn and one of their pieces is already selected
- **WHEN** the user selects a different own piece before submitting a move
- **THEN** the white outline moves to the newly selected piece’s cell
- **AND** red destination outlines update for the newly selected piece
- **AND** any prior return-mode is cancelled without spending a step

## ADDED Requirements

### Requirement: Finish-block click resolves to nearest legal center cell

The central finish area MAY be presented as one visual 2×2 block. While the client is submitting a move (not return-mode) and the selected piece has at least one legal destination among the four center cells, activating **any** point on that finish block MUST submit a move to the **nearest** of those **legal** center cells relative to the selected piece’s current cell (Chebyshev distance; ties broken by lower row, then lower column). The client MUST NOT require the user to hit a specific quadrant of the block matching that cell. If none of the four center cells is a legal destination for the selected piece, activating the finish block MUST NOT submit a move.

#### Scenario [SC-MOVE-63]: Any click on finish picks nearest legal center

- **GIVEN** it is the user’s turn, an own piece is selected, and exactly one of the four center cells is a legal one-step destination for that piece
- **WHEN** the user activates any point on the visual finish 2×2 block
- **THEN** the client submits a move to that legal center cell
- **AND** the submission does not depend on which quadrant of the block was clicked
