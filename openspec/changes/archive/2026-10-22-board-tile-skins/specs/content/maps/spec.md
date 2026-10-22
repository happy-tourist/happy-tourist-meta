## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-54 | modified (palette swatches use kind skins) |
| SC-MAP-71 | covered (client vitest ContentMaps — MapGridPreview kind skins) |
| SC-MAP-72 | covered (client vitest ContentMaps — shared skins list/editor preview) |

## MODIFIED Requirements

### Requirement: Map editor field size and under-map palette

On the map **edit** surface (not View-only), the lined 10×10 field MUST be presented at a size comparable to the in-game board (a large square usable for painting, not a small side thumbnail). The three paint tools (start, task, finish) MUST appear as a palette **below** the field. Each tool MUST show a tile-like control whose face uses the same product skin as that kind on the game board (start / unbound task / finish), with its label **under** that tile. Palette faces MUST NOT use a CSS corner radius that clips the skin silhouette (use `border-radius: 0` on the swatch face). Seat config controls MAY remain nearby but MUST NOT displace the under-map palette as the primary tool chrome.

#### Scenario [SC-MAP-54]: Palette under board-sized editor field

- **GIVEN** a verified editor on the map edit surface
- **WHEN** the user views the paint chrome
- **THEN** the map field is shown at a large board-comparable size
- **AND** start, task, and finish tools appear below the field
- **AND** each tool has a visible label under its tile
- **AND** each tool face shows the product skin for that kind (not a flat color swatch only)
- **AND** the swatch face does not clip the skin with a rounded CSS corner

## ADDED Requirements

### Requirement: Map grid previews use board tile skins

Wherever the client shows a mini or editor grid preview of a map (Maps list cards, map editor field, lobby create/room-card map previews, and other surfaces that reuse the shared `MapGridPreview`), painted start / task / finish cells MUST use the same product tile skins as the game board for those kinds. Empty/hole cells MUST remain empty chrome (not a start/task/finish skin). Flipped-task artwork is out of scope for static map previews (previews have no bind/flip state).

#### Scenario [SC-MAP-71]: Preview cells use kind skins

- **GIVEN** an authenticated user viewing a map mini preview (Maps list card, lobby create/room card, or shared `MapGridPreview`) whose grid contains start, task, and finish cells
- **WHEN** the preview is shown
- **THEN** start cells show the start skin
- **AND** task cells show the unbound task skin
- **AND** finish cells show the finish skin
- **AND** hole cells do not show those skins

#### Scenario [SC-MAP-72]: Editor field preview matches list preview skins

- **GIVEN** a verified editor on the map edit surface with painted start, task, and finish cells
- **WHEN** the user views the editor field
- **THEN** those cells use the same kind skins as Maps list and lobby mini previews
