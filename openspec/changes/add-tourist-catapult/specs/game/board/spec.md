## Purpose

Delta этого change: публичная анимация катапульты на доске и sequential piece travel. Базовая геометрия и решётки — main `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-22 | covered (client UX — hidden until land) |
| SC-BOARD-23 | covered (client UX — successful overlay vanish then piece travel) |
| SC-BOARD-24 | covered (client UX — broken 300+300 hold then vanish; piece stays) |
| SC-BOARD-25 | covered (client UX — piece pinned on catapult cell until overlay gone) |
| SC-BOARD-26 | covered (client UX — fling→catapult chain sequential) |
| SC-BOARD-27 | covered (client UX — board non-interactive during any board anim) |

Related: seed / fling / consume — `game/move`; finish travel — `game/finish`.

## ADDED Requirements

### Requirement: Hidden catapults are invisible until triggered

Until a piece lands on a cell that still holds an unspent catapult (including via push or return-from-finish), clients MUST NOT render that catapult. Spectators and other seats MUST NOT learn catapult locations from synced state before reveal.

#### Scenario [SC-BOARD-22]: Board shows no catapult before land

- **GIVEN** phase is `playing` and hidden catapults exist on some task cells
- **WHEN** the board is shown before any piece has landed on those cells
- **THEN** no catapult artwork is shown on those cells

### Requirement: Successful catapult presentation then piece travel

When a piece triggers an unspent catapult that has at least one legal fling destination, every client that displays the board MUST: (1) show the intact catapult appear and fully disappear on that cell (about **1000 ms** total presentation sense, same order of magnitude as grille drop/rise); (2) while that overlay is visible, keep the piece visually on the catapult cell even if authoritative sync already relocated it; (3) **only after** the overlay is fully gone, animate the piece to the fling destination with the same travel sense as an ordinary move (`MOVE_ANIM_MS` order). After a successful consume, the catapult MUST NOT remain visible. Cleared/spent catapults MUST NOT remain visible afterward.

#### Scenario [SC-BOARD-23]: Successful catapult vanishes before travel

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult that has at least one legal fling destination
- **WHEN** that catapult is revealed and consumed
- **THEN** every client shows the intact catapult appear-then-vanish on that cell lasting about 1000 ms
- **AND** the catapult artwork is gone afterward
- **AND** the piece travel to the fling destination starts only after that vanish completes

#### Scenario [SC-BOARD-25]: Piece stays on catapult cell during overlay

- **GIVEN** a successful catapult reveal is presenting on a cell
- **WHEN** the overlay is still visible
- **THEN** every client shows that piece on the catapult cell (not yet at the fling destination)

#### Scenario [SC-BOARD-26]: Catapult chain is sequential

- **GIVEN** a fling lands a piece onto another cell that still holds an unspent catapult
- **WHEN** that next catapult resolves for presentation
- **THEN** clients complete the prior catapult vanish and prior travel before starting the next catapult overlay
- **AND** there is no product cap on chain length beyond one-shot consume per catapult

### Requirement: Broken catapult hold timeline

When a piece triggers an unspent catapult with **no** legal fling destination, every client that displays the board MUST present: intact appear → hold about **300 ms** intact → switch to broken artwork → hold about **300 ms** broken → vanish. The piece MUST remain on that cell (no fling travel). Cleared/spent catapults MUST NOT remain visible afterward.

#### Scenario [SC-BOARD-24]: Broken catapult 300+300 then vanish

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult with no legal fling destination
- **WHEN** the broken presentation runs
- **THEN** every client shows intact appear, about 300 ms intact, broken art about 300 ms, then vanish
- **AND** the catapult artwork is gone afterward
- **AND** the piece remains on that cell

### Requirement: Board is non-interactive during board animations

While any board presentation animation is running for the local client (piece move travel, rescue/push approach, grille drop/rise, catapult overlay including broken holds, deferred fling travel after catapult vanish, finish travel/disappear), the seated current player MUST NOT be able to click the board to select pieces, submit moves, peek, rescue, push, return-from-finish, or end-turn board targets. Non-board chrome (e.g. leave/status outside board) is out of this requirement’s scope unless already gated elsewhere.

#### Scenario [SC-BOARD-27]: Clicks blocked while board animates

- **GIVEN** the local seated player’s turn and a board animation is in progress (including catapult overlay or pending fling travel)
- **WHEN** the player attempts a board click that would otherwise submit or select
- **THEN** that board interaction is ignored until the animation sequence finishes
