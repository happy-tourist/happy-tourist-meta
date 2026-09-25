## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-39 | modified (content peek, not stub reward-only) |
| SC-MOVE-94 | pending (server mocha — flipped peek no spend) |
| SC-MOVE-95 | pending (server mocha — auto-end considers flipped free peek) |

Related: peek modal / bind — `game/board`; budgets — existing private steps/peeks requirements.

## MODIFIED Requirements

### Requirement: Multiple peeks per turn while peeks remain

While it is a seat’s turn in phase `playing`, that seat MAY successfully open as many **fresh (unbound)** peeks as its peeks budget (or solo infinite peeks) allows, each time an own unfinished free piece stands on a still-present **unbound** task cell (see `game/board`). Additionally, that seat MAY open peeks on **already flipped (bound)** still-present task cells under the free-peek rules of `game/board` even when peeks are 0. The server MUST NOT enforce a one-peek-per-turn limit. Each successful **fresh** open MUST still consume one peek in finite peeks mode; flipped opens MUST NOT.

#### Scenario [SC-MOVE-39]: Second peek in the same turn is allowed

- **GIVEN** it is a multiplayer turn and the current seat has already resolved one fresh peek this turn, still has peeks ≥ 1, and an own unfinished piece stands on a still-present unbound task cell
- **WHEN** that seat attempts another peek
- **THEN** the server accepts the peek open
- **AND** the shared peek modal is shown per `game/board`

## ADDED Requirements

### Requirement: Flipped peek does not consume peeks budget

Opening a peek on an already flipped task cell MUST NOT decrease the seat’s peeks budget.

#### Scenario [SC-MOVE-94]: Flipped peek does not consume peeks

- **GIVEN** it is a multiplayer turn with peeks 1 and an own piece on a flipped task cell
- **WHEN** that seat opens a peek on the flipped cell and later resolves it incorrectly
- **THEN** peeks remain 1 (no consumption for the flipped open)

### Requirement: Auto end-turn treats flipped peek as available action

The server MUST treat a legal own peek on a flipped still-present task cell (even with peeks 0) as an available action that blocks multiplayer auto end-turn, the same way a fresh peek with peeks ≥ 1 blocks auto end-turn. When no legal move, rescue, return, push, fresh peek, or flipped peek remains, auto end-turn MUST apply as today.

#### Scenario [SC-MOVE-95]: Auto end-turn waits while flipped peek exists at zero peeks

- **GIVEN** it is a seated player’s multiplayer turn with steps 0, peeks 0, and an own unfinished free piece on a flipped still-present task cell
- **WHEN** the server evaluates auto end-turn
- **THEN** the turn MUST NOT auto-advance solely because peeks are 0
- **AND** the player MAY open the flipped peek
