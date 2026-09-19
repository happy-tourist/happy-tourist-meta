## Purpose

Delta этого change: seed катапульт, стек ловушек на клетке, fling / broken, триггеры land. Базовые budgets/turn/move/grille/push — main `game/move`.

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

When phase is `playing` and a piece **lands** on a cell via an accepted move, push, or return-from-finish, the server SHALL resolve every still-unspent trap on that cell (grille and/or catapult) in a **fresh random order** for that resolve. Landing after a catapult fling MUST use the same resolve rules as an ordinary move/push landing on the destination cell. While a piece is trapped by a grille on a cell, remaining unspent catapults on that cell MUST wait until the piece becomes free on that cell (successful rescue) and then MUST resolve as a new land-style resolve for that piece. Each trap type’s existing consume rules apply (grille → trap holding; catapult → fling or broken, then spent).

#### Scenario [SC-MOVE-87]: Stacked traps shuffle order each resolve

- **GIVEN** a free unfinished piece lands on a cell that still holds both an unspent grille and an unspent catapult
- **WHEN** the server resolves that landing
- **THEN** either the grille or the catapult may fire first according to a random shuffle for that resolve
- **AND** a later resolve on the same cell (if traps remain) MUST shuffle again rather than reusing a fixed seed order

#### Scenario [SC-MOVE-86]: Rescue then remaining catapult flings

- **GIVEN** a piece is trapped on a cell that still holds an unspent catapult after the grille is holding
- **WHEN** that piece is successfully rescued and becomes free on that same cell
- **THEN** the remaining catapult resolves immediately for that piece (fling or broken)
- **AND** the catapult is consumed

### Requirement: Catapult flings to a free landable ring cell

When a catapult resolves for a free unfinished piece on its cell, the server SHALL consume that catapult (one-shot) and MUST attempt to relocate the piece as follows: pick uniformly at random among **free legal landing** cells at Chebyshev distance **2** from the catapult cell; if that set is empty, pick uniformly among free legal landing cells at Chebyshev distance **1**. A legal landing cell MUST be playable on the tourist layout, MUST NOT be a removed-task hole, MUST NOT be outside the board, and MUST NOT be occupied by any unfinished piece (including trapped). The piece’s own former cell MUST NOT count as occupied against itself once the fling relocates it. Destination selection MUST use occupancy and holes **at fire time**. If both rings are empty, the catapult MUST still be consumed, the piece MUST remain on the catapult cell, and clients MUST present the broken vanish per `game/board`. A successful fling MUST NOT spend an extra step beyond the step already spent for the triggering move/push/return (rescue-triggered fling spends no additional step beyond the rescue). After a successful fling relocation, destination side-effects MUST follow ordinary landing rules (including further traps and center finish).

#### Scenario [SC-MOVE-80]: Move onto catapult flings when ring-2 is free

- **GIVEN** it is a seated player’s turn with steps ≥ 1 and a legal move lands a free unfinished piece on a cell with an unspent catapult and at least one free legal landing cell at Chebyshev-2
- **WHEN** that move is accepted and the catapult resolves
- **THEN** the piece occupies one uniformly chosen free Chebyshev-2 legal landing cell
- **AND** the catapult on the origin cell is spent
- **AND** steps decreased by 1 for the move only

#### Scenario [SC-MOVE-81]: Fling falls back to ring-1 when ring-2 empty

- **GIVEN** a catapult resolves and no free legal landing cell exists at Chebyshev-2, but at least one exists at Chebyshev-1
- **WHEN** the destination is chosen
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
- **THEN** the catapult resolves for the target as on an ordinary move landing

#### Scenario [SC-MOVE-84]: Return onto catapult triggers the same resolve

- **GIVEN** a successful return-from-finish places a piece onto a cell with an unspent catapult
- **WHEN** that landing is resolved
- **THEN** the catapult resolves for that piece as on an ordinary move landing

#### Scenario [SC-MOVE-85]: After fling, destination land effects apply

- **GIVEN** a catapult flings a piece onto a destination that still holds an unspent grille
- **WHEN** destination landing is resolved
- **THEN** that grille traps the piece per existing grille rules

#### Scenario [SC-MOVE-88]: Catapult is one-shot

- **GIVEN** a catapult on a cell has already been consumed (successful fling or broken)
- **WHEN** another piece later lands on that cell
- **THEN** that spent catapult MUST NOT fire again in this match

#### Scenario [SC-MOVE-89]: Fling onto center finishes the piece

- **GIVEN** a catapult flings a piece onto a center cell that is a legal landing at fire time
- **WHEN** that destination landing is applied
- **THEN** the piece finishes per `game/finish` as if it had moved onto that center cell
