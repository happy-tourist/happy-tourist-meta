# game/pieces Delta

Related: bottom HUD layout — `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-09 | pending (client strip in bottom HUD without caption) |
| SC-PIECE-10 | covered-by-reuse (spectator still no strip — presence layout) |

## MODIFIED Requirements

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current unfinished pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. While a seated player has no pieces yet (phase `waiting` or `countdown`), no pieces for that seat MUST appear on the board. Finished pieces MUST NOT be rendered on the board (see `game/finish`). A seated client SHALL see a personal strip once their four pieces exist, with exactly four slots corresponding one-to-one to that client’s four pieces by side (same kind image per slot). That strip MUST appear inside the bottom Game HUD panel (`game/presence`), after the seated user’s own presence/budgets cluster and before opponent markers, and MUST NOT use a separate caption label such as «Мои туристы». Until pieces exist, the seated client MUST NOT be shown moveable strip slots for that seat’s missing pieces. Each strip slot whose piece is finished MUST show a finish indicator at the top-right and MUST NOT participate in move selection. Slots for unfinished pieces keep existing move-selection behavior while applicable (`game/move`). The strip MUST remain visible for finished seats. The strip MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal strip.

#### Scenario [SC-PIECE-09]: Seated client sees own four-slot strip

- **GIVEN** the user is on the Game screen as a seated player with four pieces of one kind
- **WHEN** the game UI is shown
- **THEN** the bottom HUD panel displays exactly four images of that user’s tourist kind after the user’s own presence/budgets cluster
- **AND** no separate «Мои туристы» (or equivalent) caption labels that strip
- **AND** the strip does not display other players’ tourist kinds
- **AND** the four strip slots correspond to the user’s four pieces (one per side)
- **AND** any finished piece’s slot shows a finish indicator and is not used for move selection

#### Scenario [SC-PIECE-10]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the game UI is shown
- **THEN** seated players’ unfinished pieces are visible on the board at their cells
- **AND** no personal tourist strip is shown for that user

#### Scenario [SC-PIECE-17]: No board pieces before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with one or more seated players who have tourist kinds
- **WHEN** any client views the Game board
- **THEN** no tourist pieces for those seats are shown on the board
- **AND** when phase becomes `playing` and pieces are materialized, those pieces appear at their synced cells
