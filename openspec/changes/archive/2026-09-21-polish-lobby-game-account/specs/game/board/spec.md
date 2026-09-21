## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-27 | covered (client GamePage continuous isBoardBusy + grille hold unlock) |
| SC-BOARD-32 | covered (client strip under isInteractive; say remains) |

## MODIFIED Requirements

### Requirement: Board is non-interactive during board animations

While any board presentation animation is running for the local client (piece land/move travel, rescue/push approach, grille drop/rise including the pipeline settle into hold so there is no unlock flash before the grille drop, catapult overlay including broken holds, deferred fling travel after catapult vanish, finish travel/disappear), **and** during any gap between consecutive presentation steps of the same pipeline (for example after move travel ends and before catapult/grille presentation starts after a successful move), the seated current player MUST NOT be able to click the board to select pieces, submit moves, peek, rescue, push, return-from-finish, or end-turn board targets. After grille **drop has settled** into a static hold, the board/strip MUST unlock again so rescue (and other legal turn actions) remain possible while the holding grille stays visible. The same lock MUST apply to the personal tourist strip beside the own presence marker (select / return affordances) for the duration of the busy window above. Say send from the own presence avatar MUST remain available under existing `game/say` rules. Non-board chrome outside board/strip (e.g. leave/status in the app header) is out of this requirement’s scope unless already gated elsewhere. Turn chrome MAY still show the acting seat until the server advances after pipeline idle (`game/move`).

#### Scenario [SC-BOARD-27]: Clicks blocked while board animates

- **GIVEN** the local seated player’s turn and a board animation is in progress (including land-before-catapult, catapult overlay, pending fling travel, grille drop/rise or the brief window between move travel and the next trap presentation — not a settled static grille hold after drop)
- **WHEN** the player attempts a board click that would otherwise submit or select
- **THEN** that board interaction is ignored until the animation sequence finishes
- **AND** after grille drop has settled into hold, rescue remains available while the grille is still shown

#### Scenario [SC-BOARD-32]: Strip tourists locked while board is busy

- **GIVEN** the local seated player’s turn and board presentation is busy per SC-BOARD-27
- **WHEN** the player activates a personal strip tourist or return affordance
- **THEN** that strip interaction is ignored until the board is no longer busy
- **AND** the say send affordance on the own presence avatar remains usable per `game/say`
