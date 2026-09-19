## Purpose

Delta этого change: UX возврата с финиша — зелёная иконка над strip вместо модалки; disappear на центре при finish от push как при обычном ходе. Wire return и геометрия кольца — main `game/finish` / `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-01 | covered-by-reuse (server + client move finish travel+disappear) |
| SC-FINISH-02 | covered-by-reuse (server mocha — center reuse) |
| SC-FINISH-13 | covered (client UX — green return icon; no confirm modal) |
| SC-FINISH-16 | covered (client UX — finished strip slot click does not start return) |
| SC-FINISH-17 | covered (client UX — push onto center same travel+disappear as move) |

Related: return geometry and steps — `game/move`; push finish — delta `game/move` SC-MOVE-76; strip dimming — main `game/finish`.

## MODIFIED Requirements

### Requirement: Entering any center cell finishes a piece

When the room start phase is `playing` and the server accepts a legal **move or push** whose landing target is any of the four center cells of the tourist layout, the server SHALL mark that piece finished. A finished piece MUST NOT occupy any board cell for subsequent move validation (the center cell becomes free for other pieces immediately). Every client that displays the board MUST remove that piece from the board after travel onto the landing cell and a short disappear animation on that cell (no travel toward the strip) — including when finish was caused by a **push**. Finished piece state MUST be synchronized to all clients in the room.

#### Scenario [SC-FINISH-01]: Move onto a center cell finishes the piece

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** that piece is marked finished in synced state
- **AND** that center cell is not occupied by that piece for later moves
- **AND** every client observes the piece travel onto the center cell then leave the board after a short disappear animation

#### Scenario [SC-FINISH-02]: Another piece may reuse the same center cell

- **GIVEN** a piece has just finished on a specific center cell and no other unfinished piece occupies that cell
- **WHEN** another unfinished piece legally moves onto that same center cell on a later turn
- **THEN** the server accepts the move and finishes that second piece
- **AND** the first finished piece remains finished off the board

#### Scenario [SC-FINISH-17]: Push onto a center cell finishes with the same presentation

- **GIVEN** a legal push whose far-side cell is a free center cell
- **WHEN** the push is accepted and the target finishes
- **THEN** every client that displays the board observes the target travel onto that center cell then leave after the same short disappear animation as a move finish
- **AND** the target does not vanish from its pre-push cell without that travel

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST show a green return affordance centered above that tourist in the personal strip (same visual family as board rescue/push affordances). Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above) and MUST NOT show the return affordance. Activating the finished strip slot body (the tourist image area) MUST NOT open a confirmation dialog and MUST NOT enter return-mode by itself — only the return affordance starts return. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a green return affordance is shown centered above that tourist (same visual family as board rescue/push)
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation
