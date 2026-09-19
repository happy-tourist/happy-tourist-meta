## Purpose

Delta этого change: seed катапульт, стек ловушек, fling / broken, **paced** land resolve, **deferred** turn advance, **idle re-eval** auto-end after traps. Базовые budgets/turn/move/grille/push — main `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-78 | covered (server mocha — medium catapult seed count on task cells) |
| SC-MOVE-79 | covered (server mocha — catapult+grille may share a cell) |
| SC-MOVE-80 | covered (server mocha — move onto catapult flings to ring-2 when free) |
| SC-MOVE-81 | covered (server mocha — fling prefers ring-2 then ring-1) |
| SC-MOVE-82 | covered (server mocha — no dest → broken consume, piece stays) |
| SC-MOVE-83 | covered (server mocha — push onto catapult triggers same as move) |
| SC-MOVE-84 | covered (server mocha — return onto catapult triggers same as move) |
| SC-MOVE-85 | covered (server mocha — after fling, dest land effects apply) |
| SC-MOVE-86 | covered (server mocha — rescue then remaining catapult flings) |
| SC-MOVE-87 | covered (server mocha — stacked traps resolve in random order) |
| SC-MOVE-88 | covered (server mocha — catapult one-shot consumed) |
| SC-MOVE-89 | covered (server mocha — fling onto center finishes — see game/finish) |
| SC-MOVE-90 | covered (server mocha — paced: next hop only after presentation budget) |
| SC-MOVE-91 | covered (server mocha — auto-end deferred until trap pipeline idle) |
| SC-MOVE-92 | covered (server mocha — deadline advance deferred until trap pipeline idle) |
| SC-MOVE-93 | covered (server mocha — idle re-eval auto-end after grille removes peek) |

Related: board anim — `game/board`; center finish — `game/finish`; grille trap/rescue — main `game/move`.

## ADDED Requirements

### Requirement: Playing seeds hidden catapults by density

When the room start phase becomes `playing`, the server SHALL place a hidden catapult on a random subset of still-present brown task cells (`*`). The subset size MUST equal the create-time catapult density percent of the layout’s task-cell count (few **12%**, medium **22%**, many **35%**), rounded to the nearest integer and clamped to `[0, taskCount]`. Start cells and center cells MUST NOT receive catapults. Catapult placement MUST NOT be predictable from coordinates alone. The same task cell MAY also hold a grille (and vice versa); independent density draws MUST allow overlap.

#### Scenario [SC-MOVE-78]: Medium catapult density seeds expected count

- **GIVEN** a tourist room created with medium catapult density transitioning into phase `playing` with the standard tourist layout (48 task cells)
- **WHEN** catapults are seeded
- **THEN** exactly `round(48 * 0.22)` = **11** hidden catapults exist on distinct task cells
- **AND** no catapult is on a start or center cell

#### Scenario [SC-MOVE-79]: Catapult and grille may share a task cell

- **GIVEN** a tourist room whose grille and catapult densities both seed at least one trap
- **WHEN** playing begins
- **THEN** it is allowed that some task cell holds both an unspent grille and an unspent catapult

### Requirement: Landing resolves remaining traps on the cell

When phase is `playing` and a piece **lands** on a cell via an accepted move, push, or return-from-finish, the server SHALL resolve still-unspent traps on that cell (grille and/or catapult) in a **fresh random order** for that resolve. Landing after a catapult fling MUST use the same resolve rules as an ordinary move/push landing on the destination cell. While a piece is trapped by a grille on a cell, remaining unspent catapults on that cell MUST wait until the piece becomes free on that cell (successful rescue) and then MUST resolve as a new land-style resolve for that piece. Each trap type’s existing consume rules apply (grille → trap holding; catapult → fling or broken, then spent).

The server MUST NOT apply an entire multi-hop catapult/grille chain in a single synchronous tick. It MUST resolve **one** next trap effect at a time, waiting a presentation budget (shared with client timing in `game/board`) after reveal / before applying relocate, grille holding, or scheduling the next cell’s resolve — so synced state exposes only the current hop, not the final chain outcome early.

#### Scenario [SC-MOVE-87]: Stacked traps shuffle order each resolve

- **GIVEN** a free unfinished piece lands on a cell that still holds both an unspent grille and an unspent catapult
- **WHEN** the server resolves that landing
- **THEN** either the grille or the catapult may fire first according to a random shuffle for that resolve
- **AND** a later resolve on the same cell (if traps remain) MUST shuffle again rather than reusing a fixed seed order

#### Scenario [SC-MOVE-86]: Rescue then remaining catapult flings

- **GIVEN** a piece is trapped on a cell that still holds an unspent catapult after the grille is holding
- **WHEN** that piece is successfully rescued and becomes free on that same cell
- **THEN** the remaining catapult resolves for that piece (fling or broken) under the paced pipeline
- **AND** the catapult is consumed

#### Scenario [SC-MOVE-90]: Paced resolve applies only the next hop after budget

- **GIVEN** a free unfinished piece lands on a cell with an unspent catapult that will fling onto another cell that still holds an unspent grille
- **WHEN** the catapult hop is revealed
- **THEN** synced piece coordinates MUST remain on the catapult cell until that hop’s presentation budget elapsed
- **AND** the destination grille MUST NOT appear in `holdingGrilleKeys` until after the fling relocation is applied and that destination land hop begins
- **AND** the server MUST NOT pre-compute and sync the final trapped position in the same tick as the initial land

### Requirement: Catapult flings to a free landable ring cell

When a catapult resolves for a free unfinished piece on its cell, the server SHALL consume that catapult (one-shot) and MUST attempt to relocate the piece as follows: pick uniformly at random among **free legal landing** cells at Chebyshev distance **2** from the catapult cell; if that set is empty, pick uniformly among free legal landing cells at Chebyshev distance **1**. A legal landing cell MUST be playable on the tourist layout, MUST NOT be a removed-task hole, MUST NOT be outside the board, and MUST NOT be occupied by any unfinished piece (including trapped). The piece’s own former cell MUST NOT count as occupied against itself once the fling relocates it. Destination selection MUST use occupancy and holes **at fire time** (when the hop’s relocate is applied after the presentation budget). If both rings are empty, the catapult MUST still be consumed, the piece MUST remain on the catapult cell, and clients MUST present the broken vanish per `game/board`. A successful fling MUST NOT spend an extra step beyond the step already spent for the triggering move/push/return (rescue-triggered fling spends no additional step beyond the rescue). After a successful fling relocation, destination side-effects MUST follow ordinary landing rules (including further traps and center finish), paced as above.

#### Scenario [SC-MOVE-80]: Move onto catapult flings when ring-2 is free

- **GIVEN** it is a seated player’s turn with steps ≥ 1 and a legal move lands a free unfinished piece on a cell with an unspent catapult and at least one free legal landing cell at Chebyshev-2
- **WHEN** that move is accepted and the catapult hop completes (reveal + presentation budget + relocate)
- **THEN** the piece occupies one uniformly chosen free Chebyshev-2 legal landing cell
- **AND** the catapult on the origin cell is spent
- **AND** steps decreased by 1 for the move only

#### Scenario [SC-MOVE-81]: Fling falls back to ring-1 when ring-2 empty

- **GIVEN** a catapult resolves and no free legal landing cell exists at Chebyshev-2, but at least one exists at Chebyshev-1
- **WHEN** the destination is chosen at fire time
- **THEN** the piece is relocated to a uniformly chosen free Chebyshev-1 legal landing cell

#### Scenario [SC-MOVE-82]: No destination leaves piece and spends broken catapult

- **GIVEN** a catapult resolves and no free legal landing cell exists at Chebyshev-2 or Chebyshev-1
- **WHEN** the catapult is consumed
- **THEN** the piece remains on the catapult cell
- **AND** the catapult is spent
- **AND** clients present broken vanish per `game/board`

#### Scenario [SC-MOVE-83]: Push onto catapult triggers the same resolve

- **GIVEN** a successful push relocates a free unfinished target onto a cell with an unspent catapult
- **WHEN** that landing is resolved
- **THEN** the catapult resolves for the target as on an ordinary move landing (paced)

#### Scenario [SC-MOVE-84]: Return onto catapult triggers the same resolve

- **GIVEN** a successful return-from-finish places a piece onto a cell with an unspent catapult
- **WHEN** that landing is resolved
- **THEN** the catapult resolves for that piece as on an ordinary move landing (paced)

#### Scenario [SC-MOVE-85]: After fling, destination land effects apply

- **GIVEN** a catapult flings a piece onto a destination that still holds an unspent grille
- **WHEN** destination landing is resolved (after the fling hop’s budget and relocate)
- **THEN** that grille traps the piece per existing grille rules

#### Scenario [SC-MOVE-88]: Catapult is one-shot

- **GIVEN** a catapult on a cell has already been consumed (successful fling or broken)
- **WHEN** another piece later lands on that cell
- **THEN** that spent catapult MUST NOT fire again in this match

#### Scenario [SC-MOVE-89]: Fling onto center finishes the piece

- **GIVEN** a catapult flings a piece onto a center cell that is a legal landing at fire time
- **WHEN** that destination landing is applied
- **THEN** the piece finishes per `game/finish` as if it had moved onto that center cell

### Requirement: Turn does not advance during trap presentation pipeline

While a trap presentation pipeline is active for the room (paced catapult/grille hops after a land), the server MUST NOT call `advanceTurn` for multiplayer auto-end-turn or for a turn-deadline expiry that fired during that pipeline. If auto-end-turn would otherwise apply, or the turn deadline fires while the pipeline is active, the server MUST record that a turn advance is pending and MUST perform that advance only after the pipeline becomes idle. When the pipeline becomes idle **without** a pending deadline/auto-end flag already set, the server MUST **re-evaluate** auto-end-turn (and solo steps-exhausted) against the seat’s available actions **after** the last hop’s effects — so a grille (or other trap) that removed the only legal peek or move can still advance the turn. The server MUST NOT require a client presentation-ack message to release the turn. Step/peek economy MUST remain unchanged. Manual `endTurn` while the pipeline is active MUST NOT skip pending presentation (reject or defer consistently with board non-interactive rules).

#### Scenario [SC-MOVE-91]: Auto-end waits for trap pipeline idle

- **GIVEN** multiplayer playing and a seated player’s last available action is a move that starts a catapult presentation pipeline
- **WHEN** after accepting that move the seat would otherwise auto-end-turn
- **THEN** `currentTurnSessionId` MUST remain that seat until the trap pipeline is idle
- **AND** only then MAY the server advance to the next eligible seat

#### Scenario [SC-MOVE-92]: Deadline during pipeline advances only after idle

- **GIVEN** a turn deadline fires while a trap presentation pipeline is still active for that turn’s seat
- **WHEN** the deadline handler runs
- **THEN** the server MUST NOT advance the turn immediately
- **AND** after the pipeline becomes idle the server MUST advance the turn (pending deadline advance)
- **AND** clients MUST still be able to complete the in-flight hop presentations before the turn chrome switches

#### Scenario [SC-MOVE-93]: Idle re-eval after grille removes the only peek

- **GIVEN** multiplayer playing, it is a seat’s turn with steps 0, peeks ≥ 1, and exactly one own unfinished piece on a still-present task cell (so a legal peek exists), and the other own pieces are not on live task cells
- **WHEN** that piece lands on the task cell and a grille traps it as part of the trap pipeline
- **THEN** after the pipeline becomes idle the server MUST re-evaluate available actions
- **AND** because the trapped piece can no longer peek and no other legal peek/move/rescue/return/push remains, the server MUST auto-advance the turn to the next eligible seat
- **AND** a prior `pendingTurnAdvance` from deadline MUST still force advance on idle without being skipped by this re-eval path
