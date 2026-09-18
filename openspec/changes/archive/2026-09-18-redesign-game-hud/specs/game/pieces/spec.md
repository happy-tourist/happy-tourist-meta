# game/pieces Delta

Related: bottom HUD — `game/presence`; return — `game/finish`; grille chrome — `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-09 | implemented (client full strip in bottom HUD — row or 2×2) |
| SC-PIECE-10 | covered-by-reuse (spectator still no strip) |
| SC-PIECE-29 | REMOVED intent — no compact chip / picker menu |
| SC-PIECE-30 | REMOVED intent — select from strip slots / board only |
| SC-PIECE-31 | implemented (client grille on strip slots when trapped) |
| SC-PIECE-32 | implemented (client strip layout row vs 2×2 by width) |

## MODIFIED Requirements

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current unfinished pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. While a seated player has no pieces yet (phase `waiting` or `countdown`), no pieces for that seat MUST appear on the board. Finished pieces MUST NOT be rendered on the board (see `game/finish`). A seated client SHALL see a personal four-slot strip once their four pieces exist, corresponding one-to-one to that client’s four pieces by side (same kind image per side). That strip MUST appear inside the bottom Game HUD panel (`game/presence`) beside the seated user’s own presence/budgets cluster and MUST NOT use a separate caption label such as «Мои туристы».

The strip MUST present four interactive slots (not a compact status-only chip and not a picker menu). Slot order for a **2×2** layout MUST be **N, E** on the top row and **W, S** on the bottom row; for a **single row** layout MUST be **N, E, W, S** left-to-right. Each slot MUST reflect status: finish indicator when finished (`game/finish`); grille presentation when the piece is trapped. Until pieces exist, the seated client MUST NOT be shown the strip. The strip MUST remain visible for finished seats. The chrome MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal strip.

Activating an unfinished non-trapped slot MUST select that piece for move UX while applicable (`game/move`). Finished and trapped pieces MUST NOT be used for move selection (finished return flow is `game/finish`). Board piece click MUST still select without opening any menu. The Game UI MUST NOT use a `q-menu` (or equivalent) full-size picker opened from a compact chip.

On viewports / HUD widths where a single row of four slots plus the own avatar cluster cannot fit without horizontal scrolling as the primary layout (target content width about **320** CSS pixels, preferably about **300**), the strip MUST use the **2×2** grid with slot size **smaller than** the own presence avatar image box. When width allows, the strip MUST use a single horizontal row of four slots.

#### Scenario [SC-PIECE-09]: Seated client sees own four-slot strip

- **GIVEN** the user is on the Game screen as a seated player with four pieces of one kind
- **WHEN** the game UI is shown
- **THEN** the bottom HUD panel shows four personal tourist slots after the user’s own presence/budgets cluster
- **AND** no compact chip-only chrome and no tourist picker menu are required to select a piece
- **AND** no separate «Мои туристы» (or equivalent) caption labels that strip
- **AND** the strip does not display other players’ tourist kinds

#### Scenario [SC-PIECE-10]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the Game presence layout is shown
- **THEN** seated players’ unfinished pieces are visible on the board at their cells
- **AND** no personal tourist strip is shown for that user

#### Scenario [SC-PIECE-17]: No board pieces before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with one or more seated players who have tourist kinds
- **WHEN** any client views the Game board
- **THEN** no tourist pieces for those seats are shown on the board
- **AND** when phase becomes `playing` and pieces are materialized, those pieces appear at their synced cells

#### Scenario [SC-PIECE-32]: Narrow width uses 2×2 strip

- **GIVEN** the seated user views Game at about 320 CSS pixels content width (or narrower about 300)
- **WHEN** the bottom HUD with own avatar and personal strip is shown
- **THEN** the four tourist slots are laid out as a 2×2 grid with slots smaller than the own avatar image
- **AND** the own avatar cluster plus strip fit without relying on horizontal scrolling as the primary UX

## ADDED Requirements

### Requirement: Trap grille mirrored on personal tourist chrome

When a seated user’s piece is trapped, the matching strip slot MUST show a grille presentation over that tourist (same asset family as the board grille). When the piece is freed, that grille presentation MUST clear. The chrome grille MUST use the same about **1000 ms** drop/rise timing as board revealed grilles (`game/board`).

#### Scenario [SC-PIECE-31]: Trapped side shows grille on strip slot

- **GIVEN** the user’s piece for side N is trapped and unfinished
- **WHEN** the user views the personal tourist strip
- **THEN** the N strip slot shows a grille over that tourist
- **AND** when that piece is no longer trapped, that grille presentation is cleared

## REMOVED Requirements

### Requirement: Compact chip opens full-size picker menu

**Reason:** Product rejected chip + `q-menu` picker; selection returns to in-HUD strip slots (row or 2×2).

**Migration:** Remove compact chip and picker menu UX; keep four strip slots in the bottom HUD per MODIFIED strip requirement. Scenarios SC-PIECE-29 / SC-PIECE-30 no longer apply.
