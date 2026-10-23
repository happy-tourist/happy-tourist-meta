## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-01 | modified (enter-center travel ~550 ms then longer fade; no drop without travel) |
| SC-FINISH-02 | unchanged (carried) |
| SC-FINISH-09 | unchanged (flag size not scaled) |
| SC-FINISH-13 | modified (36×36 return; hit clear of strip tourist figure) |
| SC-FINISH-16 | unchanged (carried) |
| SC-FINISH-17 | modified (same longer finish presentation as move→center) |
| SC-FINISH-18 | modified (same longer finish presentation) |
| SC-FINISH-19 | modified (catapult→center: longer travel after vanish; no race-drop) |
| SC-FINISH-20 | modified (finish travel even when finish sync late; longer timing) |

## MODIFIED Requirements

### Requirement: Entering any center cell finishes a piece

When the room start phase is `playing` and the server accepts a legal **move or push** whose landing target is any of the four center cells of the tourist layout, the server SHALL mark that piece finished. A finished piece MUST NOT occupy any board cell for subsequent move validation (the center cell becomes free for other pieces immediately). Every client that displays the board MUST remove that piece from the board after travel onto the landing cell and a disappear animation on that cell (no travel toward the strip) — for finish caused by a **move** and by a **push**, with the same presentation. The travel onto the center cell MUST be clearly visible for about **550 ms** (product default), and the subsequent disappear fade MUST last about **300 ms** after that travel. The piece MUST NOT vanish from its pre-finish board cell without that travel. Finished piece state MUST be synchronized to all clients in the room.

#### Scenario [SC-FINISH-01]: Move onto a center cell finishes the piece

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** that piece is marked finished in synced state
- **AND** that center cell is not occupied by that piece for later moves
- **AND** every client observes the piece travel onto the center cell for about 550 ms then leave the board after about a 300 ms disappear animation

#### Scenario [SC-FINISH-02]: Another piece may reuse the same center cell

- **GIVEN** a piece has just finished on a specific center cell and no other unfinished piece occupies that cell
- **WHEN** another unfinished piece legally moves onto that same center cell on a later turn
- **THEN** the server accepts the move and finishes that second piece
- **AND** the first finished piece remains finished off the board

#### Scenario [SC-FINISH-17]: Push onto a center cell finishes with the same presentation

- **GIVEN** a legal push whose far-side cell is a free center cell
- **WHEN** the push is accepted and the target finishes
- **THEN** every client that displays the board observes the target travel onto that center cell then leave after the same ~550 ms travel / ~300 ms disappear presentation as a move finish
- **AND** the target does not vanish from its pre-push cell without that travel

#### Scenario [SC-FINISH-18]: Own move onto center travels from the pre-move cell

- **GIVEN** it is a seated player’s turn and an own unfinished piece has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell and the server accepts
- **THEN** clients that display the board show that piece traveling from its pre-move cell onto the center cell
- **AND** then leaving after the same disappear presentation used for push→center
- **AND** the piece does not vanish from its pre-move cell without that travel

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST show a return affordance centered above that tourist in the personal strip (same visual family as board rescue/push affordances). The affordance MUST use the **illustrated board-action return icon** and MUST NOT paint a solid colored (including green) circular chrome behind the icon. The illustrated return glyph MUST render at **36×36 CSS pixels** for both the illustrated glyph and the hit target (glyph fills the hit). The return hit MUST sit entirely **above** that strip tourist figure (about an 8 CSS-pixel gap above the slot — same clearance as board peek/rescue/push). The finished-strip **flag** indicator MUST keep its existing size (not scaled by this change). Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above) and MUST NOT show the return affordance. Activating the finished strip slot body (the tourist image area) MUST NOT open a confirmation dialog and MUST NOT enter return-mode by itself — only the return affordance starts return. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a return affordance with the illustrated return icon is shown centered above that tourist (same visual family as board rescue/push; no solid colored circular chrome)
- **AND** the return glyph / hit are the 36×36 glyph-fills-hit size family
- **AND** the return hit does not cover that strip tourist figure
- **AND** the finished-flag indicator size is unchanged by this scale
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules (after that hop’s presentation budget), the server SHALL finish that piece with the same authority and presentation expectations as an ordinary move or push onto that center cell (`game/finish` enter-center rules). Clients MUST run finish travel and disappear **after** the triggering catapult’s land arrival and overlay have fully completed (`game/board` successful presentation: land → overlay → then finish travel), not in parallel with that overlay. Finish travel MUST use the same about **550 ms** slide and about **300 ms** disappear family as move/push onto center. Under server-paced sync the piece MAY become `finished` only when the hop’s relocate fires — **after** reveal enqueue — and clients MUST still present finish travel from the catapult cell after that hop’s vanish (they MUST NOT drop the piece from the board without travel solely because `finished` was not known at reveal time, and MUST NOT drop it without travel when `finished` arrives early). In a multi-hop catapult chain ending on center, every hop MUST animate; finish travel MUST start only after the last successful catapult vanish.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** that catapult’s arrival and overlay have fully completed and the deferred fling / finish presentation runs
- **THEN** the piece is finished with the same authority as a move onto that center
- **AND** clients present finish travel (~550 ms) and disappear (~300 ms) consistent with move/push onto center, starting only after the catapult vanish
- **AND** the piece does not disappear from the catapult cell without that travel

#### Scenario [SC-FINISH-20]: Finish travel even when finish sync arrives after reveal

- **GIVEN** a successful catapult reveal is presenting and the authoritative finish onto center is applied only after the overlay presentation budget (paced fire)
- **WHEN** the overlay vanish completes
- **THEN** every client MUST animate finish travel from the catapult cell to center and then disappear
- **AND** MUST NOT leave the piece absent from the board without that travel solely because `finished` was false at reveal enqueue time
- **AND** in a catapult→…→center chain, prior hops complete their overlay and fling travel before the final hop’s finish travel
