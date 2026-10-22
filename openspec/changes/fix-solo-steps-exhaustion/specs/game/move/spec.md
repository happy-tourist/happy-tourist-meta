# Delta: game/move

Связано с main `openspec/specs/game/move/spec.md` (solo peeks∞ / SC-MOVE-48) и `game/finish` (ordinary center finish без полного finish place). Этот change закрывает: соло soft-lock при steps=0 без легального peek (остаток на старте / в тюрьме) после ordinary хода на центр — немедленный time-expired вместо ожидания пятиминутного таймера.

## Traceability

| ID | Coverage |
|----|----------|
| SC-MOVE-40 | covered-by-reuse (main — solo peeks∞) |
| SC-MOVE-41 | covered-by-reuse (main — solo no end-turn) |
| SC-MOVE-48 | modified-by-this-change (legal peek = free unfinished on live `*`; trapped/start do not keep seat alive) |
| SC-MOVE-52 | covered-by-reuse (main — trapped cannot move) |
| SC-MOVE-53 | covered-by-reuse (main — trapped cannot peek) |
| SC-MOVE-97 | covered (server mocha — last step ordinary center finish; remainder on start + trapped → timeExpired + deadline cleared) |
| SC-MOVE-98 | covered (server mocha — steps 0; only unfinished on live `*` is trapped → evaluateSoloStepsExhaustedForTests → timeExpired) |

## MODIFIED Requirements

### Requirement: Solo infinite peeks and finite steps

When phase is `playing` and exactly one seated player remains eligible to move, the server SHALL set that seat’s **peeks** budget to an **infinite** mode, MUST keep **steps** as a finite number (not infinite), MUST NOT accept or require end-turn to keep playing, and MUST NOT apply further turn grants while that same seat remains current under solo (successful moves do not re-grant). When another seat’s permanent leave or finish causes this seat to **become** current turn as the sole eligible seat, the private-budget grant for becoming current under solo (+1 step, no peek increment) MUST apply. When the seat was **already** current and merely becomes the sole eligible seat, steps MUST carry unchanged (no extra step solely for entering solo). The five-minute solo deadline remains; additionally, when steps reach **0** and the seat has **no legal peek** — no own **free** (non-trapped) unfinished piece standing on a still-present task cell — the server MUST mark that seat **time-expired** the same way as solo timer expiry (moves and peeks rejected) and MUST clear the active solo turn deadline **immediately**, without waiting for the five-minute timer to elapse. Unfinished pieces on start cells, non-task path cells, or trapped under a grille MUST NOT keep the seat alive when steps are 0. Infinite peeks alone MUST NOT keep the seat alive without a legal peek target. An unresolved room `openPeek` for that seat MUST NOT block this end: the server MUST clear or force-close that peek (incorrect resolve without removing the task tile is acceptable) and MUST still apply time-expired and deadline clear. This solo end evaluation MUST run after an accepted move that spends the last step — including an ordinary one-step move onto a center cell that finishes a piece without assigning a finish place (`game/finish`) — not only after moves that leave the mover unfinished off a task cell. Finishing all pieces follows `game/finish`. The solo client MUST be informed via a modal that peeks are unlimited while steps remain finite. Other clients MUST NOT see that seat’s budget values.

#### Scenario [SC-MOVE-40]: Becoming solo while already current carries steps

- **GIVEN** a tourist room in phase `playing` where it is non-finished seated player A’s turn with a finite steps budget greater than 0, and every other seated player either receives a finish place or permanently leaves, leaving A as the only eligible seat **without** changing current turn away from A
- **WHEN** that condition becomes true
- **THEN** A’s peeks are infinite and steps remain the prior finite value (no extra +1 step solely for entering solo)
- **AND** that client is shown the solo peeks-unlimited modal
- **AND** the synchronized five-minute solo deadline applies as previously specified

#### Scenario [SC-MOVE-41]: Solo needs no end-turn

- **GIVEN** a non-finished seated player under solo infinite peeks
- **WHEN** that player views Game controls for ending a turn
- **THEN** the end-turn control is not offered
- **AND** successful moves do not pass the turn to another seat
- **AND** successful moves do not grant an additional +1 step solely because the seat remains solo current

#### Scenario [SC-MOVE-48]: Solo steps exhaustion without a live task tile time-locks

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seated player under solo infinite peeks, steps 0, and no own free unfinished piece on a still-present task cell (trapped or start/path pieces do not count as a live peek target)
- **WHEN** the server evaluates solo end conditions
- **THEN** that seat becomes time-expired and further moves and peeks are rejected
- **AND** the synchronized solo turn deadline is cleared
- **AND** the client is shown the steps-exhausted end copy (distinct from the five-minute timer copy)

#### Scenario [SC-MOVE-97]: Last-step ordinary center finish with start and jail remainder time-locks

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seated player under solo infinite peeks, steps exactly 1, one own free unfinished piece adjacent to a free center cell, at least one other own unfinished free piece on a start cell, and one own unfinished trapped piece (grille), with finish place still 0
- **WHEN** that player successfully moves the free piece onto the center cell (ordinary move, not a trap-pipeline fling), spending the last step and finishing that piece without assigning a finish place
- **THEN** that seat’s steps are 0
- **AND** that seat becomes time-expired
- **AND** the synchronized solo turn deadline is cleared (MUST NOT keep ticking until the five-minute budget elapses)
- **AND** further moves and peeks from that seat are rejected

#### Scenario [SC-MOVE-98]: Trapped piece on live task does not block steps exhaustion

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seated player under solo infinite peeks, steps 0, and every own unfinished piece is either trapped or not on a still-present task cell (including the case where the only unfinished piece on a still-present task cell is trapped)
- **WHEN** the server evaluates solo end conditions (same gate as after an accepted last-step move / peek resolution — including via a mocha test hook that invokes that gate)
- **THEN** that seat becomes time-expired
- **AND** the synchronized solo turn deadline is cleared
- **AND** further peeks are rejected (trapped pieces remain non-peekable)
