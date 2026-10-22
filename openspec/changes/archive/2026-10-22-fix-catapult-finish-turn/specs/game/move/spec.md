# Delta: game/move

Связано с main `openspec/specs/game/move/spec.md` (turn order / trap pipeline) и `game/finish` (finish place). Этот change закрывает: seat, получивший finish place внутри paced trap pipeline, не остаётся `currentTurn`, пока есть другой eligible seat.

## Traceability

| ID | Coverage |
|----|----------|
| SC-MOVE-21 | covered-by-reuse (main — finished seats skipped in rotation) |
| SC-MOVE-89 | covered-by-reuse (main — fling onto center finishes piece) |
| SC-MOVE-91 | covered-by-reuse (main — auto-end waits for pipeline idle) |
| SC-MOVE-92 | modified-by-this-change (pending idle: at most one logical turn leave; absorb if ensure already moved current — design D1) |
| SC-MOVE-93 | covered-by-reuse (main — idle re-eval auto-end) |
| SC-MOVE-96 | covered (server mocha — finish place via catapult→center clears current turn; pending absorb) |

## MODIFIED Requirements

### Requirement: Turn does not advance during trap presentation pipeline

While a trap presentation pipeline is active for the room (paced catapult/grille hops after a land), the server MUST NOT call `advanceTurn` for multiplayer auto-end-turn or for a turn-deadline expiry that fired during that pipeline. If auto-end-turn would otherwise apply, or the turn deadline fires while the pipeline is active, the server MUST record that a turn advance is pending and MUST perform that advance only after the pipeline becomes idle — **except** when idle eligibility repair (`ensureCurrentEligibleOrAdvance` / equivalent) has already moved `currentTurnSessionId` off the seat that held the turn at the start of idle (e.g. finish place assigned mid-pipeline): then the pending flag MUST be cleared **without** a second `advanceTurn` (pending absorb; at most one logical turn leave). When the pipeline becomes idle **without** a pending deadline/auto-end flag already set, the server MUST **re-evaluate** auto-end-turn (and solo steps-exhausted) against the seat’s available actions **after** the last hop’s effects — so a grille (or other trap) that removed the only legal peek or move can still advance the turn. The server MUST NOT require a client presentation-ack message to release the turn. Step/peek economy MUST remain unchanged. Manual `endTurn` while the pipeline is active MUST NOT skip pending presentation (reject or defer consistently with board non-interactive rules).

#### Scenario [SC-MOVE-91]: Auto-end waits for trap pipeline idle

- **GIVEN** multiplayer playing and a seated player’s last available action is a move that starts a catapult presentation pipeline
- **WHEN** after accepting that move the seat would otherwise auto-end-turn
- **THEN** `currentTurnSessionId` MUST remain that seat until the trap pipeline is idle
- **AND** only then MAY the server advance to the next eligible seat

#### Scenario [SC-MOVE-92]: Deadline during pipeline advances only after idle

- **GIVEN** a turn deadline fires while a trap presentation pipeline is still active for that turn’s seat
- **WHEN** the deadline handler runs
- **THEN** the server MUST NOT advance the turn immediately
- **AND** after the pipeline becomes idle the server MUST leave the turn off that seat via a single logical advance (pending `advanceTurn`, or pending absorb if eligibility repair already moved `currentTurnSessionId`)
- **AND** clients MUST still be able to complete the in-flight hop presentations before the turn chrome switches

#### Scenario [SC-MOVE-93]: Idle re-eval after grille removes the only peek

- **GIVEN** multiplayer playing, it is a seat’s turn with steps 0, peeks ≥ 1, and exactly one own unfinished piece on a still-present task cell (so a legal peek exists), and the other own pieces are not on live task cells
- **WHEN** that piece lands on the task cell and a grille traps it as part of the trap pipeline
- **THEN** after the pipeline becomes idle the server MUST re-evaluate available actions
- **AND** because the trapped piece can no longer peek and no other legal peek/move/rescue/return/push remains, the server MUST auto-advance the turn to the next eligible seat
- **AND** a prior `pendingTurnAdvance` from deadline MUST still force advance on idle without being skipped by this re-eval path when eligibility repair did not already change the current seat

## ADDED Requirements

### Requirement: Finish place assigned in trap pipeline clears current turn

When the trap presentation pipeline becomes idle and the seat that currently holds the turn has a synchronized finish place (`game/finish`) while at least one other eligible (non-finished, non-time-expired) seated player remains, the server SHALL advance the synchronized current turn to the next eligible seat in join order (same skip rules as finished-seat rotation). This MUST apply when the finish place was assigned during that pipeline — including a catapult fling onto a center cell that finishes the seat’s last piece — whether or not a pending auto-end or deadline advance flag was already set for that pipeline. The idle path MUST release the finished seat **at most once**: if eligibility advance already moved the current turn off that seat, a pending auto-end/deadline flag MUST be cleared **without** a second `advanceTurn` (pending absorb). Idle auto-end re-evaluation alone MUST NOT leave a finished seat as current turn when another eligible seat exists. While the pipeline is still active, the turn MAY remain on that seat until idle (unchanged paced rules). Solo (exactly one eligible seat) is unchanged: no rotation to another player.

#### Scenario [SC-MOVE-96]: Catapult full-finish releases the turn after pipeline idle

- **GIVEN** a tourist room in phase `playing` with at least two eligible seated players, it is seat A’s turn, A has exactly three finished pieces and one unfinished free piece with `finishPlace` still `0`, and that piece’s land starts a catapult presentation whose fling destination is a free center cell
- **AND** after accepting the land move A still has remaining available actions (so no pending auto-end is required solely from that move)
- **WHEN** the trap pipeline becomes idle after the paced catapult relocate finishes A’s fourth piece and assigns A a finish place
- **THEN** A’s synchronized finish place is greater than `0`
- **AND** the synchronized current turn is no longer A
- **AND** the current turn is the next eligible seat in join order (A is skipped)
