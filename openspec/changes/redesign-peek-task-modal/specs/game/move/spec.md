## Traceability

| SC-ID | Coverage target |
|-------|-----------------|
| SC-MOVE-39 | modified (multi peek while peeks remain — flipped also spends) |
| SC-MOVE-94 | modified (flipped peek **does** consume peeks) |
| SC-MOVE-95 | modified (auto-end no longer waits for flipped peek at peeks 0) |

Related: `game/board` SC-BOARD-10/45 peeks spend. Schema/message payloads unchanged.

## MODIFIED Requirements

### Requirement: Multiple peeks per turn while peeks remain

While it is a seat’s turn in phase `playing`, that seat MAY successfully open as many peeks as its peeks budget (or solo infinite peeks) allows, each time an own unfinished free piece stands on a still-present task cell (unbound or flipped — see `game/board`). The server MUST NOT enforce a one-peek-per-turn limit. Each successful open (fresh **or** flipped) MUST consume one peek in finite peeks mode. With peeks 0 (and not infinite), no further peek open is allowed on any task cell.

#### Scenario [SC-MOVE-39]: Second peek in the same turn is allowed

- **GIVEN** it is a multiplayer turn and the current seat has already resolved one fresh peek this turn, still has peeks ≥ 1, and an own unfinished piece stands on a still-present unbound task cell
- **WHEN** that seat attempts another peek
- **THEN** the server accepts the peek open
- **AND** the shared peek modal is shown per `game/board`

### Requirement: Flipped peek consumes peeks budget

Opening a peek on an already flipped task cell MUST decrease the seat’s peeks budget by one in finite peeks mode (same as a fresh open). Solo infinite peeks MUST NOT decrement.

#### Scenario [SC-MOVE-94]: Flipped peek consumes peeks

- **GIVEN** it is a multiplayer turn with peeks 1 and an own piece on a flipped task cell
- **WHEN** that seat opens a peek on the flipped cell and later resolves it incorrectly
- **THEN** peeks become 0 (the flipped open consumed one peek)

### Requirement: Auto end-turn gates flipped peek by peeks budget

The server MUST treat a legal own peek on a flipped still-present task cell as an available action that blocks multiplayer auto end-turn **only when** peeks remain (or solo infinite peeks) — the same budget gate as a fresh peek. When peeks are 0 (and not infinite), a flipped cell MUST NOT block auto end-turn solely as a peek opportunity. When no legal move, rescue, return, push, or budgeted peek remains, auto end-turn MUST apply as today.

#### Scenario [SC-MOVE-95]: Auto end-turn advances when flipped peek at zero peeks

- **GIVEN** it is a seated player’s multiplayer turn with steps 0, peeks 0, and an own unfinished free piece on a flipped still-present task cell, and no other legal move/rescue/return/push
- **WHEN** the server evaluates auto end-turn
- **THEN** the turn MAY auto-advance (flipped peek at peeks 0 is not an available action)
- **AND** the player MUST NOT be able to open the flipped peek
