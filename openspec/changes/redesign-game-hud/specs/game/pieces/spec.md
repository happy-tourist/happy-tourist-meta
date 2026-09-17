# game/pieces Delta

Related: bottom HUD — `game/presence`; return — `game/finish`; grille chrome — `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-09 | pending (client compact chip + menu row in bottom HUD) |
| SC-PIECE-10 | covered-by-reuse (spectator still no strip — presence layout) |
| SC-PIECE-29 | pending (client chip tap opens menu only) |
| SC-PIECE-30 | pending (client select from menu closes; board select without menu) |
| SC-PIECE-31 | pending (client grille status on chip/menu when trapped) |

## MODIFIED Requirements

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current unfinished pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. While a seated player has no pieces yet (phase `waiting` or `countdown`), no pieces for that seat MUST appear on the board. Finished pieces MUST NOT be rendered on the board (see `game/finish`). A seated client SHALL see personal tourist chrome once their four pieces exist, corresponding one-to-one to that client’s four pieces by side (same kind image per side). That chrome MUST appear inside the bottom Game HUD panel (`game/presence`), after the seated user’s own presence/budgets cluster and before opponent markers, and MUST NOT use a separate caption label such as «Мои туристы».

In the HUD the seated client MUST see a **compact status chip** about the size of a presence avatar image, laid out as a **2×2** grid of mini slots ordered **N, E** on the top row and **W, S** on the bottom row. Each mini slot MUST reflect that side’s status: finish indicator when finished (`game/finish`); grille presentation when the piece is trapped. Until pieces exist, the seated client MUST NOT be shown that chip. The chip MUST remain visible for finished seats. The chrome MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal chrome.

Full-size per-side controls for selection and return MUST live in a picker opened from the chip (`game/finish` for return). Slots for unfinished non-trapped pieces keep existing move-selection behavior while applicable (`game/move`) when activated from the picker or from the board. Finished and trapped pieces MUST NOT be used for move selection from the picker.

#### Scenario [SC-PIECE-09]: Seated client sees own four-slot strip

- **GIVEN** the user is on the Game screen as a seated player with four pieces of one kind
- **WHEN** the game UI is shown
- **THEN** the bottom HUD panel shows one compact 2×2 tourist status chip after the user’s own presence/budgets cluster
- **AND** the four mini slots correspond to sides N, E, W, S of that user’s pieces
- **AND** no separate «Мои туристы» (or equivalent) caption labels that chrome
- **AND** the chrome does not display other players’ tourist kinds
- **AND** any finished piece’s mini slot shows a finish indicator and is not used for move selection from the chip itself

#### Scenario [SC-PIECE-10]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the Game presence layout is shown
- **THEN** seated players’ unfinished pieces are visible on the board at their cells
- **AND** no personal tourist strip or compact chip is shown for that user

#### Scenario [SC-PIECE-17]: No board pieces before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with one or more seated players who have tourist kinds
- **WHEN** any client views the Game board
- **THEN** no tourist pieces for those seats are shown on the board
- **AND** when phase becomes `playing` and pieces are materialized, those pieces appear at their synced cells

## ADDED Requirements

### Requirement: Compact chip opens full-size picker menu

Activating the compact tourist chip MUST open a menu anchored toward the board (above the HUD) that shows the four tourists in a **single horizontal row** at full strip slot size, without a dialog title. Activating the chip MUST NOT by itself change the selected piece. Dismissing the menu (outside click or Escape) without choosing a tourist MUST leave selection unchanged. While the client is not in move-interactive state, the menu MAY still open for status viewing but MUST NOT apply move selection.

#### Scenario [SC-PIECE-29]: Chip opens picker without selecting

- **GIVEN** the seated user’s compact tourist chip is visible
- **WHEN** the user activates the chip
- **THEN** a menu opens toward the board showing four full-size tourist slots in one row
- **AND** the currently selected piece (if any) does not change solely because the menu opened

#### Scenario [SC-PIECE-30]: Select from menu or board

- **GIVEN** it is the user’s move-interactive turn and the tourist picker menu is open
- **WHEN** the user activates an unfinished non-trapped full-size slot
- **THEN** that side becomes the selected piece for move UX
- **AND** the menu closes
- **AND** activating an own unfinished non-trapped piece on the board still selects that piece without opening the menu

### Requirement: Trap grille mirrored on personal tourist chrome

When a seated user’s piece is trapped, the matching mini slot on the compact chip and the matching full-size slot in the picker menu MUST show a grille presentation over that tourist (same asset family as the board grille). When the piece is freed, that grille presentation MUST clear. The chrome grille MUST use the same about **1000 ms** drop/rise timing as board revealed grilles (`game/board`).

#### Scenario [SC-PIECE-31]: Trapped side shows grille on chip and in menu

- **GIVEN** the user’s piece for side N is trapped and unfinished
- **WHEN** the user views the compact chip and opens the picker menu
- **THEN** the N mini slot and the N full-size menu slot show a grille over that tourist
- **AND** when that piece is no longer trapped, those grille presentations are cleared
