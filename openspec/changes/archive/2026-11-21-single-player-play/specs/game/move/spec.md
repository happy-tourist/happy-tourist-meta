# Delta: game/move

Связано с main `openspec/specs/game/move/spec.md` и `game/start` / `game/presence`. Этот change: режим **одиночной** комнаты (`maxSeats === 1`) — finite +1/+1, без turn deadline, авто-+1/+1 при отсутствии действий. **Соло в конце** (`maxSeats ≥ 2`, один eligible) — без изменений (∞ + 60 с).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MOVE-03 | covered-by-reuse (keep turn when sole eligible — single-player or leftover) |
| SC-MOVE-25 | modified — authoritative deadline excludes maxSeats=1 |
| SC-MOVE-26 | covered-by-reuse (deadline ticks in grace — multi only) |
| SC-MOVE-27 | covered-by-reuse (no auto-pass before playing) |
| SC-MOVE-29 | covered-by-reuse (solo leftover 60s — maxSeats≥2 only) |
| SC-MOVE-40 | covered-by-reuse (solo leftover ∞) |
| SC-MOVE-41 | covered-by-reuse (solo leftover no end-turn) |
| SC-MOVE-50 | covered-by-reuse (solo leftover become-current → ∞) |
| SC-MOVE-100 | added — server mocha (single-player finite grant, not ∞) |
| SC-MOVE-101 | added — server mocha (single-player no turn deadline / onTurnTimeout gate) |
| SC-MOVE-102 | added — server mocha (single-player spends steps) |
| SC-MOVE-103 | added — server mocha (no actions → +1 step +1 peek) |
| SC-MOVE-104 | added — server mocha (legal peek blocks auto-+1/+1) |
| SC-MOVE-105 | added — server mocha (legal rescue/return/push blocks auto-+1/+1) |
| SC-MOVE-106 | added — server mocha (solo leftover still ∞ when maxSeats≥2) |

## ADDED Requirements

### Requirement: Single-player room budgets without turn timer

When phase is `playing` and the room’s **maxSeats is exactly 1**, the seat’s private budgets MUST remain in **finite** mode (`infinite: false`). The server MUST NOT enter solo infinite-budgets mode and MUST NOT start a synchronized turn deadline for that seat solely because exactly one eligible seat remains. At materialize into `playing`, budgets start at **0**; when that seat **becomes** current turn the server MUST add exactly **1** step and exactly **1** peek. Successful moves and peeks MUST spend finite budgets as in multiplayer. The server MUST NOT offer or require end-turn among seats. While the seat remains current under single-player rules, a successful move alone MUST NOT grant further +1/+1.

#### Scenario [SC-MOVE-100]: Single-player become-current grants finite +1/+1

- **GIVEN** a tourist room created with maxSeats 1 that has just entered phase `playing` with one eligible seated player
- **WHEN** that seat becomes current turn
- **THEN** that seat’s private budgets are finite with steps 1 and peeks 1
- **AND** `infinite` is false

#### Scenario [SC-MOVE-101]: Single-player has no turn deadline

- **GIVEN** a tourist room in phase `playing` with maxSeats 1 and one eligible seated player under finite budgets
- **WHEN** clients observe synchronized turn deadline fields
- **THEN** there is no active turn deadline for that seat
- **AND** the seat does not become time-expired solely due to elapsed wall-clock turn time
- **AND** even if a turn-timeout handler were invoked, the server MUST NOT mark that seat time-expired while maxSeats is 1

#### Scenario [SC-MOVE-102]: Single-player move spends one step

- **GIVEN** it is the sole seat’s turn in a maxSeats-1 `playing` room with finite steps ≥ 1 and a legal neighbor cell
- **WHEN** that player submits a legal move
- **THEN** the piece position updates
- **AND** that seat’s steps budget decreases by 1
- **AND** the current turn remains that same seat
- **AND** no additional +1/+1 is granted solely because the move succeeded

### Requirement: Single-player refresh when no actions remain

While phase is `playing`, maxSeats is 1, and it is the sole eligible seat’s turn, when that seat has **no available actions** — cannot legally move (steps 0 or no legal destinations; removed task cells **are** legal destinations when otherwise playable) **and** does not have a legal peek (peeks ≥ 1 and an own unfinished free piece on a still-present task cell, or solo-infinite peeks which do not apply here) **and** has no legal own rescue, return (finish place 0), or push — the server MUST add exactly **1** step and exactly **1** peek to that seat’s private budgets without changing piece positions and without waiting for a turn deadline. The server MUST **not** advance current turn to another seat (none exist). While any of those actions remain available, the server MUST NOT apply that +1/+1 refresh. An open peek modal MUST defer the refresh until the peek is resolved (skip evaluation / return without granting). While the trap presentation pipeline is active the server MUST likewise defer (skip evaluation / return without granting) and MUST **not** set multiplayer `pendingTurnAdvance` for this purpose. When evaluation runs again after peek close or trap-pipeline idle (same call sites that re-invoke multiplayer auto end-turn evaluation), the server MUST perform this +1/+1 refresh if still no actions — MUST **not** pass the turn via `advanceTurn` / multi `pendingTurnAdvance` resolution. Grant at most once per evaluation call; if after the grant the seat still has no available actions, do not grant again until a later game event re-triggers evaluation (including all-jail reset, which MUST re-run this evaluation via the idle path).

#### Scenario [SC-MOVE-103]: No actions refreshes +1 step and +1 peek

- **GIVEN** it is the sole seat’s turn in a maxSeats-1 `playing` room with finite steps 0 and peeks 0 (or peeks remain but no own unfinished free piece stands on a still-present task cell), and no legal rescue, return, or push
- **WHEN** the server evaluates available actions
- **THEN** that seat’s private budgets increase by 1 step and 1 peek
- **AND** piece positions are unchanged
- **AND** the current turn remains that same seat

#### Scenario [SC-MOVE-104]: Legal peek blocks single-player refresh

- **GIVEN** it is the sole seat’s turn in a maxSeats-1 `playing` room with steps 0, peeks ≥ 1, and an own unfinished free piece on a still-present task cell
- **WHEN** the server evaluates available actions
- **THEN** the server does not add +1 step and +1 peek solely because steps are 0

#### Scenario [SC-MOVE-105]: Legal rescue return or push blocks single-player refresh

- **GIVEN** it is the sole seat’s turn in a maxSeats-1 `playing` room with finite steps ≥ 1 and a legal own rescue, return, or push available, and no other reason to refresh
- **WHEN** the server evaluates available actions
- **THEN** the server does not add +1 step and +1 peek while that action remains available

## MODIFIED Requirements

### Requirement: Authoritative turn deadline while playing

While the start phase is `playing` and there is an eligible current-turn seat that is not time-expired, the server SHALL maintain a synchronized turn deadline for that seat **except when maxSeats is exactly 1** (single-player: no turn deadline — see single-player requirements; `clearTurnDeadline` / skip `startTurnDeadline`; `onTurnTimeout` MUST NOT mark `timeExpired`). When **maxSeats ≥ 2** and two or more seated players are eligible to move (non-finished and not time-expired), each time that seat becomes current turn the deadline MUST be exactly **60 seconds** from that moment. The deadline MUST keep counting down while the current-turn seat is offline within reconnect grace. While the phase is `waiting` or `countdown`, the server MUST NOT run a turn deadline that auto-advances the turn. When the turn advances because of end-turn, auto end-turn, a permanent leave, or a normal 60-second timeout with another eligible seat remaining, the new current-turn seat MUST receive a fresh deadline per this requirement (or the solo leftover budget requirement when applicable).

#### Scenario [SC-MOVE-25]: Sixty-second deadline on each multiplayer turn

- **GIVEN** a tourist room in phase `playing` with maxSeats ≥ 2, at least two non-finished seated players, and it becomes one of their turns
- **WHEN** every client observes the synchronized turn state
- **THEN** that current-turn seat has a turn deadline 60 seconds from becoming current
- **AND** every client in the room observes that synchronized deadline

#### Scenario [SC-MOVE-26]: Deadline ticks during reconnect grace

- **GIVEN** a non-finished current-turn seat in phase `playing` with a remaining turn deadline and at least one other eligible seat
- **WHEN** that current-turn player disconnects unexpectedly within reconnect grace
- **THEN** the synchronized turn remains on that offline seat
- **AND** the turn deadline continues to count down without pausing for the disconnect

#### Scenario [SC-MOVE-27]: No turn auto-pass before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with a current-turn seated player
- **WHEN** more than 60 seconds elapse without a move
- **THEN** the current turn does not advance solely because of elapsed wall time
- **AND** no turn-deadline auto-pass applies until phase `playing`

### Requirement: Private step and peek budgets per seat

While phase is `playing`, the server SHALL maintain for each non-finished seated player a private **steps** budget and a private **peeks** budget that are visible only to that seat’s client (not to other seated players or spectators). The private room message `budgets` continues to carry `{ steps, peeks, infinite, peekedThisTurn }`. In multiplayer (`maxSeats ≥ 2` with two or more eligible seats), `infinite` is **false** and both counters are finite numbers. In **solo leftover** (`maxSeats ≥ 2` and exactly one eligible seat), `infinite` MUST be **true** and SHALL mean **both** peeks and steps are unlimited (client shows ∞ for both; server MUST NOT decrement steps while `infinite`). In **single-player** (`maxSeats === 1`), `infinite` MUST remain **false** and budgets follow the single-player requirements of this capability. At materialize into `playing` both budgets MUST start at **0**. Each time a seat **becomes** current turn:

- while **two or more** eligible seats remain, the server MUST add exactly **1** step and exactly **1** peek to that seat’s budgets before play continues;
- while **exactly one** eligible seat remains and **maxSeats ≥ 2** (solo leftover), the server MUST set that seat’s **steps** and **peeks** to **infinite** mode (`infinite: true`) and MUST **not** apply a finite +1 step or +1 peek grant;
- while **maxSeats === 1**, the server MUST apply the finite +1 step and +1 peek grant (single-player requirement) and MUST **not** set `infinite: true`.

Budgets MUST persist across that seat’s turns (remainder carries over in finite mode). Becoming solo leftover **without** a change of current turn (another seat permanently left or finished while this seat was already current, and maxSeats ≥ 2) MUST switch that seat to infinite steps and peeks without a finite +1 grant. While the same seat remains current under solo leftover rules, successful moves MUST NOT trigger further turn grants and MUST NOT decrement steps. Reconnect within grace MUST restore the same budgets to the reclaiming client. Spectators MUST NOT receive step or peek budgets.

#### Scenario [SC-MOVE-33]: Budgets start at zero until first turn grant

- **GIVEN** a tourist room that has just entered phase `playing` with at least two non-finished seated players
- **WHEN** seated clients observe their private budgets before any turn grant animation settles
- **THEN** each non-current seat that has not yet been granted a turn this game has steps 0 and peeks 0
- **AND** the seat that becomes current turn receives +1 step and +1 peek as part of becoming current

#### Scenario [SC-MOVE-34]: Remainder carries and next turn adds one each

- **GIVEN** a non-finished seated player ends a multiplayer turn with 2 steps and 1 peek remaining
- **WHEN** that seat becomes current turn again while two or more eligible seats remain
- **THEN** that seat’s private budgets become 3 steps and 2 peeks
- **AND** other clients do not observe those budget values

#### Scenario [SC-MOVE-50]: Becoming current as the solo leftover grants one step

- **GIVEN** a tourist room with maxSeats ≥ 2 in phase `playing` with two eligible seated players A then B, it is A’s turn, and B still has steps 0 and peeks 0 (B has not yet received a turn grant this game)
- **WHEN** A permanently leaves **or** A receives a finish place so that B is the only remaining eligible seat and becomes current turn
- **THEN** B’s peeks are infinite
- **AND** B’s steps are infinite
- **AND** B is not given a finite +1 step grant
- **AND** B may legally move without spending steps

#### Scenario [SC-MOVE-106]: Solo leftover still infinite when maxSeats is at least two

- **GIVEN** a tourist room created with maxSeats 2 in phase `playing` where every other seated player has finished or permanently left, leaving exactly one eligible seat
- **WHEN** solo leftover rules apply
- **THEN** that seat’s peeks and steps are infinite
- **AND** the synchronized one-minute solo deadline applies
- **AND** single-player finite refresh rules do not apply to that seat

### Requirement: Solo infinite peeks and finite steps

When phase is `playing`, **maxSeats ≥ 2**, and exactly one seated player remains eligible to move, the server SHALL set that seat’s **peeks** budget to an **infinite** mode and SHALL set that seat’s **steps** budget to an **infinite** mode, MUST NOT accept or require end-turn to keep playing, and MUST NOT apply further turn grants while that same seat remains current under solo leftover (successful moves do not re-grant and do not spend steps). When another seat’s permanent leave or finish causes this seat to **become** current turn as the sole eligible seat, infinite steps and peeks MUST apply without a finite +1 step grant. When the seat was **already** current and merely becomes the sole eligible seat, steps and peeks MUST become infinite (any prior finite remainder is superseded; no finite +1). The one-minute solo deadline remains as the sole solo soft-lock besides finishing all pieces (`game/finish`). The server MUST **not** mark the seat time-expired because steps reached 0 or because no legal peek remains. Finishing all pieces follows `game/finish`. The solo leftover client MUST be informed via a modal that they are alone, that moves and peeks are unlimited, and that they have one minute. Other clients MUST NOT see that seat’s budget values. This requirement MUST **not** apply when maxSeats is 1 (see single-player requirements).

#### Scenario [SC-MOVE-40]: Becoming solo while already current carries steps

- **GIVEN** a tourist room with maxSeats ≥ 2 in phase `playing` where it is non-finished seated player A’s turn with a finite steps budget greater than 0, and every other seated player either receives a finish place or permanently leaves, leaving A as the only eligible seat **without** changing current turn away from A
- **WHEN** that condition becomes true
- **THEN** A’s peeks are infinite and A’s steps are infinite
- **AND** that client is shown the solo unlimited-budgets modal
- **AND** the synchronized one-minute solo deadline applies as specified

#### Scenario [SC-MOVE-41]: Solo needs no end-turn

- **GIVEN** a non-finished seated player under solo leftover infinite peeks and infinite steps (maxSeats ≥ 2)
- **WHEN** that player views Game controls for ending a turn
- **THEN** the end-turn control is not offered
- **AND** successful moves do not pass the turn to another seat
- **AND** successful moves do not grant an additional +1 step solely because the seat remains solo current

#### Scenario [SC-MOVE-48]: Solo steps exhaustion without a live task tile time-locks

- **GIVEN** a tourist room in phase `playing` with maxSeats ≥ 2 and exactly one eligible seated player under solo infinite peeks and infinite steps, and no own free unfinished piece on a still-present task cell
- **WHEN** the server evaluates solo end conditions
- **THEN** that seat does **not** become time-expired solely because of missing peeks or step count
- **AND** the synchronized one-minute solo deadline keeps running until timer expiry or finish

#### Scenario [SC-MOVE-97]: Last-step ordinary center finish with start and jail remainder time-locks

- **GIVEN** a tourist room in phase `playing` with maxSeats ≥ 2 and exactly one eligible seated player under solo infinite steps and peeks, one own free unfinished piece adjacent to a free center cell, at least one other own unfinished free piece on a start cell, and one own unfinished trapped piece (grille), with finish place still 0
- **WHEN** that player successfully moves the free piece onto the center cell (ordinary move, not a trap-pipeline fling), finishing that piece without assigning a finish place
- **THEN** that seat’s steps remain infinite
- **AND** that seat does **not** become time-expired
- **AND** the synchronized solo turn deadline keeps ticking
- **AND** further moves and peeks from that seat remain allowed

#### Scenario [SC-MOVE-98]: Trapped piece on live task does not block steps exhaustion

- **GIVEN** a tourist room in phase `playing` with maxSeats ≥ 2 and exactly one eligible seated player under solo infinite steps and peeks, and every own unfinished piece is either trapped or not on a still-present task cell
- **WHEN** the server evaluates solo end conditions
- **THEN** that seat does **not** become time-expired solely for that reason
- **AND** the synchronized solo turn deadline is not cleared by that evaluation

### Requirement: Solo endgame five-minute budget

When phase is `playing`, **maxSeats ≥ 2**, and exactly one seated player remains eligible to move (all other seats either hold a finish place or have permanently left), the server SHALL give that seat a synchronized turn deadline of exactly **one minute** (60 seconds) from the moment that condition becomes true, replacing any prior multiplayer turn deadline, and SHALL apply solo leftover infinite peeks and infinite steps as specified above. While that solo budget runs, the seat remains current turn after its own successful moves without resetting the remaining solo budget to a new one minute on each move and without +1/+1 turn grants; if the player finishes all four pieces before the deadline, normal finish rules apply (`game/finish`) and the turn deadline MUST clear. Spectators and other clients MUST observe the same synchronized solo deadline. When maxSeats is 1, this requirement MUST NOT apply (no turn deadline — see single-player requirements).

#### Scenario [SC-MOVE-29]: Solo remaining player gets five minutes

- **GIVEN** a tourist room with maxSeats ≥ 2 in phase `playing` where every other seated player has either a finish place or has permanently left, leaving exactly one non-finished seated player
- **WHEN** that condition becomes true
- **THEN** that seat’s synchronized turn deadline is exactly one minute (60 seconds) from that moment
- **AND** every client observes that one-minute deadline

#### Scenario [SC-MOVE-30]: Successful solo finish clears the deadline

- **GIVEN** a tourist room in phase `playing` with maxSeats ≥ 2 and exactly one non-finished seated player under the one-minute solo budget
- **WHEN** that player finishes their last piece and receives a finish place
- **THEN** the turn deadline for that seat is cleared
- **AND** the seat follows finished-seat rules without a time-expired lock

#### Scenario [SC-MOVE-45]: Solo timer expiration time-locks with timer copy

- **GIVEN** a tourist room in phase `playing` with maxSeats ≥ 2 and exactly one non-finished seated player under the one-minute solo budget with infinite peeks and infinite steps
- **WHEN** the one-minute deadline elapses without finishing all pieces
- **THEN** that seat becomes time-expired and further moves and peeks are rejected
- **AND** the client is shown the timer-expired end copy

### Requirement: Player end-turn and auto end-turn when no actions remain

While phase is `playing`, at least two eligible seats remain, and it is a non-finished non-time-expired seat’s turn, that client MUST be able to submit an end-turn action. On acceptance the server MUST advance the turn to the next eligible seat in join order and grant that seat +1 step and +1 peek per the budget requirement. The server MUST **not** auto-advance while the current seat still has peeks ≥ 1 **and** at least one own unfinished piece stands on a still-present task cell (a legal peek remains available), even if steps are 0. The server MUST also treat a legal own rescue, a legal own return (when finish place is 0), or a legal own push as available actions. When the current seat has **no available actions** — cannot legally move (steps 0 or no legal destinations; removed task cells **are** legal destinations when otherwise playable) **and** does not have (peeks ≥ 1 and an own unfinished piece on a still-present task cell) **and** has no legal rescue, return, or push — the server MUST auto-advance the turn the same way without waiting for the deadline. Solo leftover mode (maxSeats ≥ 2, one eligible) MUST NOT offer or require end-turn advancement among seats. Single-player (maxSeats === 1) MUST NOT offer end-turn; when no available actions remain it MUST refresh +1/+1 per the single-player refresh requirement instead of advancing turn.

#### Scenario [SC-MOVE-37]: End-turn advances and grants the next seat

- **GIVEN** a tourist room in phase `playing` with two eligible seated players A then B, it is A’s turn, and A still has steps or peeks remaining
- **WHEN** A submits end-turn
- **THEN** the current turn becomes B
- **AND** B’s private budgets increase by 1 step and 1 peek
- **AND** no piece positions change solely because of end-turn

#### Scenario [SC-MOVE-38]: Auto end-turn when no actions remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0 and peeks 0 (or peeks remain but no own unfinished piece stands on a still-present task cell), so no legal move and no available peek remain
- **WHEN** the server evaluates available actions
- **THEN** the turn advances to the next eligible seat without waiting for the deadline

#### Scenario [SC-MOVE-47]: Auto end-turn does not fire while peeks and a live task tile remain

- **GIVEN** it is a seated player’s multiplayer turn with steps 0, peeks ≥ 1, and an own unfinished piece on a still-present task cell
- **WHEN** the server evaluates available actions
- **THEN** the turn does not auto-advance solely because steps are 0
