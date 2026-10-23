## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-13 | modified (peek = illustrated board-action icon; no solid colored circular chrome) |
| SC-BOARD-14 | modified (same visual family; keep-focus → illustrated peek icon) |

## MODIFIED Requirements

### Requirement: Eye affordance to open a peek

While it is the user’s multiplayer or solo turn and a peek is currently allowed for one of their unfinished pieces on a still-present task cell, activating that piece MUST offer a peek affordance to open the peek modal. The peek affordance MUST use the **illustrated board-action peek icon** (not a Material eye glyph) and MUST be shown **centered above** that tourist on the board (same top-center family as strip return and board rescue/push affordances). The affordance MUST NOT paint a solid colored circular chrome behind the icon (no solid fill and no medallion-like `box-shadow`); the hit target remains large enough for touch (~32 CSS pixels). When local selection is already kept on such a piece after a move (`game/move` keep-focus), the peek affordance MUST appear without requiring another activation click. Clients that are not allowed to peek MUST NOT show that affordance as a way to open a peek. A removed-task hole under the piece MUST NOT offer the peek affordance.

#### Scenario [SC-BOARD-13]: Current player sees eye on a peekable tourist

- **GIVEN** it is the user’s turn with an allowed peek and an own unfinished piece on a still-present task cell
- **WHEN** the user activates that piece
- **THEN** a peek affordance with the illustrated peek icon is available centered above that tourist to open the peek
- **AND** the affordance has no solid colored circular chrome behind the icon (no solid fill / medallion-like `box-shadow`)
- **AND** other clients do not gain that peek affordance for the user’s piece

#### Scenario [SC-BOARD-14]: Eye after move without re-select when still on a present task tile

- **GIVEN** it is the user’s turn with peeks remaining (or solo infinite peeks) and after a successful move the same unfinished piece remains selected on a still-present task cell
- **WHEN** the move animation finishes and interaction returns
- **THEN** the peek affordance (illustrated peek icon) is available on that piece without requiring another selection click
- **AND** if the cell under the piece is a removed-task hole or not a task cell, the peek affordance is not shown for that piece
