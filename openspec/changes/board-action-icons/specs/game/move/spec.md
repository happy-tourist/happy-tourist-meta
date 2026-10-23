## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-42 | modified (timeout + open peek → incorrect KEEP + peekClose timeout reason → «Не успел!» flash) |
| SC-MOVE-74 | modified (pre-scale push; hit clear of tourist figure) |
| SC-MOVE-77 | modified (pre-scale rescue; hit clear of tourist figure) |

## MODIFIED Requirements

### Requirement: Turn timeout forces open peek as incorrect

When a multiplayer 60-second turn deadline elapses, or when a solo turn deadline elapses while a peek is open, the server SHALL resolve that open peek as **incorrect** (no step reward) and MUST NOT remove the peeked task tile (same KEEP as a wrong submit in `game/board`) before or as part of advancing / expiring the turn. That peek-close MUST identify the **turn-timeout** reason so every client can show the match-status flash «Не успел!» per `game/presence`. Wrong-submit closes MUST NOT use the timeout reason.

#### Scenario [SC-MOVE-42]: Timeout with open peek is incorrect and keeps the tile

- **GIVEN** it is a seated player’s turn with an unresolved peek modal open
- **WHEN** the server applies the turn timeout
- **THEN** that peek is resolved as incorrect with no steps added
- **AND** the peeked task tile is not removed
- **AND** every client receives a peek-close identifying the timeout reason
- **AND** every client shows the match-status flash «Не успел!» for about ten seconds

### Requirement: Push affordances on targets of the selected pusher

While it is the user’s own turn with steps ≥ 1 and a free unfinished own piece is locally selected, the client MUST show a push affordance **centered above** each unfinished free piece (own or other) that that selected piece can legally push under the push requirement (same top-center family as strip return). Each push affordance MUST use the **illustrated board-action push icon** and MUST NOT paint a solid colored (including green) circular chrome behind the icon. The illustrated glyph MUST render at about **18 CSS pixels** (pre-scale family), and the hit target MUST be about **32 CSS pixels**. Each push hit MUST sit entirely **above** its target tourist figure (same clearance as peek / strip return — about an 8 CSS-pixel gap above the cell top). The peek affordance MUST remain on the selected own piece when peek is available (illustrated peek icon centered above that piece per `game/board`). Push affordances MUST NOT appear for trapped targets, when no piece is selected, when it is not the user’s turn, or when steps are 0. Activating a push affordance MUST submit that push. Destination cells for push MUST NOT use red move-target rings; only the affordance initiates push. After a successful push the submitting client MUST keep selection on the pusher. The client MUST present an approach-and-back animation of the pusher toward the target and ordinary piece travel of the target to the destination (visible to the submitting client for approach; travel visible to clients that display the board). When the destination is a center cell and the target finishes, every client that displays the board MUST present the same travel-to-landing then disappear animation as for an ordinary move onto center (`game/finish` / SC-FINISH-01) — the target MUST NOT vanish without traveling to the center cell.

#### Scenario [SC-MOVE-74]: Push icons appear over pushable neighbors of the selection

- **GIVEN** it is the user’s turn with steps ≥ 1, one own free piece is selected, and two different free neighbors are each legally pushable through their far-side cells
- **WHEN** the board renders affordances
- **THEN** a push affordance with the illustrated push icon is shown centered above each of those two targets
- **AND** those affordances use the pre-scale (~18 / ~32) glyph / hit family
- **AND** those affordance hits do not cover the target tourist figures
- **AND** those affordances have no solid colored circular chrome behind the icon
- **AND** no push affordance is shown over the selected pusher for those actions
- **AND** if a peek is also available for the selected piece, the peek affordance remains centered above the selected piece

#### Scenario [SC-MOVE-75]: Push click animates and keeps selection

- **GIVEN** the same turn with a selected pusher and one push affordance visible
- **WHEN** the user activates that affordance and the server accepts the push
- **THEN** the pusher shows an approach-and-back presentation
- **AND** the target travels to the destination
- **AND** local selection remains on the pusher

#### Scenario [SC-MOVE-76]: Push onto center travels then disappears

- **GIVEN** it is a seated player’s turn with a legal push whose far-side cell is a free center cell
- **WHEN** that player successfully pushes the target onto that center cell
- **THEN** clients that display the board show the target traveling onto the center cell
- **AND** then removing that piece with the same short disappear animation as a center finish from a move
- **AND** the target does not disappear from its pre-push cell without that travel

### Requirement: Rescue affordance centered above trapped tourist

While it is the user’s own turn with steps ≥ 1 and a legal own rescue exists for a trapped unfinished piece, the client MUST show the rescue affordance **centered above** that trapped tourist on the board (same top-center family as push / peek / strip return). The affordance MUST use the **illustrated board-action rescue icon** and MUST NOT paint a solid colored (including green) circular chrome behind the icon. The illustrated glyph MUST render at about **18 CSS pixels** (pre-scale family), and the hit target MUST be about **32 CSS pixels**. The rescue hit MUST sit entirely **above** that trapped tourist figure (same clearance as peek / push).

#### Scenario [SC-MOVE-77]: Rescue icon centered above trapped

- **GIVEN** it is the user’s turn with steps ≥ 1 and a legal own rescue exists
- **WHEN** the board renders affordances
- **THEN** a rescue affordance with the illustrated rescue icon is shown centered above that trapped tourist
- **AND** the affordance uses the pre-scale (~18 / ~32) glyph / hit family
- **AND** the affordance hit does not cover that tourist figure
- **AND** the affordance has no solid colored circular chrome behind the icon
