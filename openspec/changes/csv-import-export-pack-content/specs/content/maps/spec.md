## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-53 | covered (client vitest map editor paint/save) |
| SC-MAP-54 | covered (client vitest / page smoke layout) |

Related: map grid and paint editor — main `content/maps` (SC-MAP-04 and bottom palette wording). This change hardens quiet-save races and aligns editor layout with a board-sized field and under-map palette.

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
