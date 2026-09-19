## Purpose

Delta этого change: публичная анимация катапульты на доске. Базовая геометрия и решётки — main `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-22 | covered (client UX — hidden until land) |
| SC-BOARD-23 | covered (client UX — fade in then fade out ~1000 ms all clients) |
| SC-BOARD-24 | covered (client UX — broken sprite on vanish when no fling dest) |

Related: seed / fling / consume — `game/move`.

## ADDED Requirements

### Requirement: Hidden catapults are invisible until triggered

Until a piece lands on a cell that still holds an unspent catapult (including via push or return-from-finish), clients MUST NOT render that catapult. Spectators and other seats MUST NOT learn catapult locations from synced state before reveal.

#### Scenario [SC-BOARD-22]: Board shows no catapult before land

- **GIVEN** phase is `playing` and hidden catapults exist on some task cells
- **WHEN** the board is shown before any piece has landed on those cells
- **THEN** no catapult artwork is shown on those cells

### Requirement: Revealed catapult fade animations are public

When a piece triggers an unspent catapult on a cell, every client that displays the board MUST show the intact catapult fading in (opacity about **0 → 1**) and then fading out (opacity about **1 → 0**) on that cell for about **1000 ms** total presentation sense (same order of magnitude as grille drop/rise). After a successful consume, the catapult MUST NOT remain visible. When the catapult fires with **no** legal fling destination, every such client MUST switch to the **broken** catapult artwork for the fade-out portion. Cleared/spent catapults MUST NOT remain visible afterward.

#### Scenario [SC-BOARD-23]: Successful catapult fades in then out

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult that has at least one legal fling destination
- **WHEN** that catapult is revealed and consumed
- **THEN** every client shows the intact catapult fade-in then fade-out on that cell lasting about 1000 ms
- **AND** the catapult artwork is gone afterward

#### Scenario [SC-BOARD-24]: Broken catapult on vanish when no destination

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult with no legal fling destination
- **WHEN** the catapult vanish presentation runs
- **THEN** every client shows the broken catapult artwork during the fade-out
- **AND** the catapult artwork is gone afterward
- **AND** the piece remains on that cell
