## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-23 | modified (successful catapult ~1500 ms) |
| SC-BOARD-24 | modified (broken holds ×1.5 → ~450+450 then vanish ~300 ms; **keep historic scenario title**) |
| SC-BOARD-25 | unchanged (carried in MODIFIED successful block) |
| SC-BOARD-26 | unchanged (carried) |
| SC-BOARD-31 | unchanged (carried) |
| SC-BOARD-58 | covered (client — overlay separation from tile skins; board + strip chrome) |

Related unchanged: grille drop/rise ~1000 ms (SC-BOARD-18/19); land-before-overlay (SC-BOARD-28); paced budget uses these presentation times (`game/move` SC-MOVE-90).

## MODIFIED Requirements

### Requirement: Successful catapult presentation then piece travel

When a piece triggers an unspent catapult that has at least one legal fling destination, every client that displays the board MUST, **after** SC-BOARD-28 arrival: (1) show the intact catapult appear and fully disappear on that cell (about **1500 ms** total presentation sense); (2) while that overlay is visible, keep the piece visually on the catapult cell; (3) **only after** the overlay is fully gone, animate the piece to the fling destination with the same travel sense as an ordinary move (`MOVE_ANIM_MS` order), following authoritative relocate when it arrives for that hop. After a successful consume, the catapult MUST NOT remain visible. Cleared/spent catapults MUST NOT remain visible afterward. The server paced presentation budget for a successful catapult hop MUST match that about **1500 ms** client sense (`game/move`) — production default `CATAPULT_ANIM_MS = 1500` (and the same value restored by `resetTrapPresentationBudgets`).

#### Scenario [SC-BOARD-23]: Successful catapult vanishes before travel

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult that has at least one legal fling destination
- **WHEN** that catapult is revealed and consumed
- **THEN** every client shows the intact catapult appear-then-vanish on that cell lasting about 1500 ms only after arrival on that cell completed
- **AND** the catapult artwork is gone afterward
- **AND** the piece travel to the fling destination starts only after that vanish completes

#### Scenario [SC-BOARD-25]: Piece stays on catapult cell during overlay

- **GIVEN** a successful catapult reveal is presenting on a cell
- **WHEN** the overlay is still visible
- **THEN** every client shows that piece on the catapult cell (not yet at the fling destination)

#### Scenario [SC-BOARD-26]: Catapult chain is sequential

- **GIVEN** a fling lands a piece onto another cell that still holds an unspent catapult
- **WHEN** that next catapult resolves for presentation
- **THEN** clients complete the prior catapult vanish and prior fling travel, then arrival onto the next catapult cell, before starting the next catapult overlay
- **AND** there is no product cap on chain length beyond one-shot consume per catapult

#### Scenario [SC-BOARD-31]: Every hop animates including final fling to center

- **GIVEN** a multi-hop catapult chain whose last fling destination is a center cell (finish)
- **WHEN** clients present that pipeline
- **THEN** each hop shows land → overlay → travel (or finish travel on the last hop after its vanish)
- **AND** the piece MUST NOT disappear from an intermediate catapult cell without travel
- **AND** finish travel to center starts only after the last successful catapult vanish

### Requirement: Broken catapult hold timeline

When a piece triggers an unspent catapult with **no** legal fling destination, every client that displays the board MUST, **after** SC-BOARD-28 arrival, present: intact appear → hold about **450 ms** intact → switch to broken artwork → hold about **450 ms** broken → vanish (vanish fade about **300 ms**). The piece MUST remain on that cell (no fling travel). Cleared/spent catapults MUST NOT remain visible afterward. The server paced presentation budget for a broken catapult hop MUST match the sum of those holds plus vanish (`game/move`) — production default `CATAPULT_BROKEN_PRESENTATION_MS = 450 + 450 + 300` (**1200** ms; and the same value restored by `resetTrapPresentationBudgets`).

#### Scenario [SC-BOARD-24]: Broken catapult 300+300 then vanish

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult with no legal fling destination
- **WHEN** the broken presentation runs
- **THEN** every client shows intact appear, about 450 ms intact, broken art about 450 ms, then vanish (about 300 ms fade), only after arrival on that cell completed
- **AND** the catapult artwork is gone afterward
- **AND** the piece remains on that cell

## ADDED Requirements

### Requirement: Catapult and grille overlays stay readable on tile skins

Revealed catapult overlays and revealed grille overlays on the board MUST remain visually separable from the underlying task (and other) tile skins so players can tell the trap art from the cell face. The same separation MUST apply to the personal strip chrome grille that mirrors a trapped piece (`game/pieces`). Separation MUST use a non-filling edge treatment — CSS `filter: drop-shadow(...)` on `.catapult-overlay`, `.grille-overlay`, and `.my-tourist-chrome-grille` (starter values in change `design.md` D1; tweak in apply if needed) — so grille bar openings stay alpha-clear. Clients MUST NOT paint an opaque underlay behind the grille that fills those openings and hides the tourist.

#### Scenario [SC-BOARD-58]: Overlays read on skinned tiles without hiding tourist under grille

- **GIVEN** phase is `playing`, task cells use product tile skins, and a piece triggers a catapult or holding grille on a task cell (or a strip slot shows chrome grille for a trapped own piece)
- **WHEN** the overlay or chrome grille is visible
- **THEN** the catapult or grille artwork remains distinguishable from the tile skin behind it via CSS `filter: drop-shadow` (or equivalent non-filling edge treatment) on the overlay / chrome-grille element
- **AND** for a grille over a tourist, the tourist remains visible through the grille openings
- **AND** no opaque underlay fills those grille openings
