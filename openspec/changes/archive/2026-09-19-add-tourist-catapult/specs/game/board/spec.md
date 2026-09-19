## Purpose

Delta этого change: публичная анимация катапульты, land-before-overlay, sequential hops под **server-paced** sync, grille не раньше своего land, always-animated hops incl. finish. Базовая геометрия и решётки — main `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-22 | covered (client UX — hidden until land) |
| SC-BOARD-23 | covered (client UX — land then ~1000 ms overlay then fling travel) |
| SC-BOARD-24 | covered (client UX — land then broken 300+300 then vanish; piece stays) |
| SC-BOARD-25 | covered (client UX — piece on catapult cell during overlay after land) |
| SC-BOARD-26 | covered (client UX — chain: land→overlay→travel per hop) |
| SC-BOARD-27 | covered (client UX — board non-interactive during full sequence) |
| SC-BOARD-28 | covered (client UX — land arrives before overlay; all viewers) |
| SC-BOARD-29 | covered (client UX — grille drop only on its hop after prior catapult hops) |
| SC-BOARD-30 | covered (client UX — single sequential trap timeline; no early final grille) |
| SC-BOARD-31 | covered (client UX — every hop animated; finish travel after last vanish to center) |

Related: seed / fling / paced resolve / deferred turn / idle re-eval — `game/move`; finish travel — `game/finish`.

## ADDED Requirements

### Requirement: Hidden catapults are invisible until triggered

Until a piece lands on a cell that still holds an unspent catapult (including via push or return-from-finish), clients MUST NOT render that catapult. Spectators and other seats MUST NOT learn catapult locations from synced state before reveal.

#### Scenario [SC-BOARD-22]: Board shows no catapult before land

- **GIVEN** phase is `playing` and hidden catapults exist on some task cells
- **WHEN** the board is shown before any piece has landed on those cells
- **THEN** no catapult artwork is shown on those cells

### Requirement: Land on catapult cell before overlay for every viewer

When a piece triggers an unspent catapult (via move, push, return-from-finish, or post-rescue resolve while free on the cell), every client that displays the board — including spectators and non-acting seats — MUST complete a visual arrival onto that catapult cell with ordinary piece-travel sense **before** starting the catapult overlay. Under server-paced sync, piece coordinates SHOULD remain on the catapult cell until relocate; if a client still observes early relocate, it MUST synthesize arrival. If the piece is already visually on that cell and no arrival animation is in progress (e.g. post-rescue on the same cell), clients MUST NOT invent a fake step and MAY start the overlay immediately after any in-flight arrival animation ends.

#### Scenario [SC-BOARD-28]: Arrival completes before catapult overlay

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult on a cell (including when the acting player is not the local viewer)
- **WHEN** clients present that trigger
- **THEN** every client finishes visual arrival onto that catapult cell before the catapult overlay begins
- **AND** spectators use the same order as the acting player’s clients

### Requirement: Successful catapult presentation then piece travel

When a piece triggers an unspent catapult that has at least one legal fling destination, every client that displays the board MUST, **after** SC-BOARD-28 arrival: (1) show the intact catapult appear and fully disappear on that cell (about **1000 ms** total presentation sense); (2) while that overlay is visible, keep the piece visually on the catapult cell; (3) **only after** the overlay is fully gone, animate the piece to the fling destination with the same travel sense as an ordinary move (`MOVE_ANIM_MS` order), following authoritative relocate when it arrives for that hop. After a successful consume, the catapult MUST NOT remain visible. Cleared/spent catapults MUST NOT remain visible afterward.

#### Scenario [SC-BOARD-23]: Successful catapult vanishes before travel

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult that has at least one legal fling destination
- **WHEN** that catapult is revealed and consumed
- **THEN** every client shows the intact catapult appear-then-vanish on that cell lasting about 1000 ms only after arrival on that cell completed
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

When a piece triggers an unspent catapult with **no** legal fling destination, every client that displays the board MUST, **after** SC-BOARD-28 arrival, present: intact appear → hold about **300 ms** intact → switch to broken artwork → hold about **300 ms** broken → vanish. The piece MUST remain on that cell (no fling travel). Cleared/spent catapults MUST NOT remain visible afterward.

#### Scenario [SC-BOARD-24]: Broken catapult 300+300 then vanish

- **GIVEN** phase is `playing` and a piece triggers an unspent catapult with no legal fling destination
- **WHEN** the broken presentation runs
- **THEN** every client shows intact appear, about 300 ms intact, broken art about 300 ms, then vanish, only after arrival on that cell completed
- **AND** the catapult artwork is gone afterward
- **AND** the piece remains on that cell

### Requirement: Grille presentation follows the same sequential trap timeline

When a paced land hop resolves a grille after zero or more prior catapult hops in the same pipeline, every client MUST present the grille drop on that grille’s cell only as part of that hop — **not** in parallel with earlier catapult overlays/travels, and not on a cell the piece has not yet visually reached in the sequence. Clients MUST treat catapult hops and the subsequent grille hop as one sequential trap timeline.

#### Scenario [SC-BOARD-29]: Grille drop waits for prior catapult hops

- **GIVEN** a piece is flung through one or more catapults and then lands on a cell whose grille traps it
- **WHEN** clients present that pipeline
- **THEN** no grille drop artwork appears on the final cell until prior catapult vanish and fling travel hops for that pipeline have completed
- **AND** the grille drop runs only when presenting that grille land hop

#### Scenario [SC-BOARD-30]: No early empty grille on the final cell

- **GIVEN** the same multi-hop pipeline as SC-BOARD-29
- **WHEN** an intermediate catapult overlay is still presenting
- **THEN** clients MUST NOT show a holding grille on the eventual trap cell ahead of the piece’s arrival in the sequence

### Requirement: Board is non-interactive during board animations

While any board presentation animation is running for the local client (piece land/move travel, rescue/push approach, grille drop/rise, catapult overlay including broken holds, deferred fling travel after catapult vanish, finish travel/disappear), the seated current player MUST NOT be able to click the board to select pieces, submit moves, peek, rescue, push, return-from-finish, or end-turn board targets. Non-board chrome (e.g. leave/status outside board) is out of this requirement’s scope unless already gated elsewhere. Turn chrome MAY still show the acting seat until the server advances after pipeline idle (`game/move`).

#### Scenario [SC-BOARD-27]: Clicks blocked while board animates

- **GIVEN** the local seated player’s turn and a board animation is in progress (including land-before-catapult, catapult overlay, pending fling travel, or grille drop in the trap pipeline)
- **WHEN** the player attempts a board click that would otherwise submit or select
- **THEN** that board interaction is ignored until the animation sequence finishes
