## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-65 | covered (client vitest GameBoardWire — per-cell finish click / outline; REMOVED obsolete + ADDED new under same id)

## REMOVED Requirements

### Requirement: Finish-block click resolves to nearest legal center cell

**Reason:** Finish is no longer one visual 2×2 block; each finish cell is its own tile. Nearest-legal remapping and block-wide click are obsolete.
**Migration:** Use `Finish-cell click submits that cell only` (SC-MOVE-65) below. Scenario title «Any click on finish picks nearest legal center» no longer applies.

## ADDED Requirements

### Requirement: Finish-cell click submits that cell only

Finish cells on the Game board MUST behave like other playable tiles for local move selection chrome and move submission (they are **not** one visual 2×2 block). While the client is submitting a move (not return-mode) and a finish cell F is a legal one-step destination for the selected piece, activating **that** cell MUST submit a move to F. Activating a different finish cell that is not legal MUST NOT submit a move to F via nearest-center remapping. Red destination outlines MUST appear only on finish cells that are legal for the selected piece. White selection outline MUST follow the selected piece’s cell only. Room message `move` payload `{ pieceId, row, col }` and server validation of that destination cell remain unchanged (existing per-cell `7` playable centers); this requirement is client chrome only.

#### Scenario [SC-MOVE-65]: Finish click targets the activated cell

- **GIVEN** it is the user’s turn, an own unfinished piece is selected, and exactly one of several finish cells is a legal one-step destination for that piece
- **WHEN** the user activates that legal finish cell
- **THEN** the client submits a move to that same cell
- **AND** only that legal finish cell shows the red destination outline among finish cells
- **AND WHEN** the user activates a different finish cell that is not a legal destination
- **THEN** the client MUST NOT submit a move to the legal finish cell via nearest-center remapping
