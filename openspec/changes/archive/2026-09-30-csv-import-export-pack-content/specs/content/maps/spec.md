## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-53 | covered (client vitest map editor paint/save) |
| SC-MAP-54 | covered (client vitest / page smoke layout) |
| SC-MAP-55 | covered (client vitest maps list cards) |
| SC-MAP-56 | covered (client vitest centered editor seats) |

Related: map grid and paint editor — main `content/maps` (SC-MAP-04 and bottom palette wording). This change hardens quiet-save races, aligns editor layout (board-sized field, under-map palette, centered column), and presents the Maps section as cards.

## ADDED Requirements

### Requirement: Map paint does not lose cells during quiet save

While the map editor is applying a quiet or staff save of the working grid, the client MUST NOT allow a later paint to be discarded when an in-flight save response returns. The client MUST either ignore further cell paints until the in-flight save settles, or apply save responses only when they are not older than the local grid the user has painted since that request started. Rapid consecutive paints MUST leave every accepted paint visible after saves complete.

#### Scenario [SC-MAP-53]: Second paint survives first save round-trip

- **GIVEN** an editable map editor with an in-progress quiet or staff save after painting cell A
- **WHEN** the user paints cell B before that save response is applied
- **THEN** after saves settle both cell A and cell B remain painted as the user left them
- **AND** the editor MUST NOT replace the local grid with an older server snapshot that omits cell B

### Requirement: Map editor field size and under-map palette

On the map **edit** surface (not View-only), the lined 10×10 field MUST be presented at a size comparable to the in-game board (a large square usable for painting, not a small side thumbnail). The three paint tools (start, task, finish) MUST appear as a palette **below** the field. Each tool MUST show a tile-like control with its label **under** that tile. Seat config controls MAY remain nearby but MUST NOT displace the under-map palette as the primary tool chrome.

#### Scenario [SC-MAP-54]: Palette under board-sized editor field

- **GIVEN** a verified editor on the map edit surface
- **WHEN** the user views the paint chrome
- **THEN** the map field is shown at a large board-comparable size
- **AND** start, task, and finish tools appear below the field
- **AND** each tool has a visible label under its tile

### Requirement: Maps list uses card tiles with mini preview

The Maps section list MUST render each map as a card in a wrapping row. Each card MUST show a **mini preview** of the grid in the upper area and the seat config as **players×tourists** below that preview. Author identity and status chrome MAY appear on the card. Action controls (Edit, soft-unpublish, republish, and similar) MUST appear at the **bottom** as stacked full-width **text** buttons when available. Existing open/navigation and visibility rules for unpublished maps MUST remain.

#### Scenario [SC-MAP-55]: Maps list renders cards with mini preview and capacity

- **GIVEN** an authenticated user on the Maps section with at least one listed map
- **WHEN** the user views the list
- **THEN** each map is shown as a card
- **AND** the card shows a mini grid preview above the players×tourists capacity
- **AND** Edit when available is a bottom full-width text control

### Requirement: Map editor column is centered with usable seat selects

On the map **edit** surface, the editor column that contains the field, under-map palette, and seat-count controls MUST be **horizontally centered** on the page (comparable to how the in-game board is presented). The players and tourists-per-player selects MUST remain usable at a readable width and MUST NOT appear collapsed into unusably narrow controls.

#### Scenario [SC-MAP-56]: Editor column centered and seat selects usable

- **GIVEN** a verified editor on the map edit surface
- **WHEN** the user views the editor chrome
- **THEN** the field-plus-palette-plus-seat column is centered horizontally
- **AND** the players and tourists selects are wide enough to read and operate
