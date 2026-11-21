# Delta: game/presence

Связано с main `openspec/specs/game/presence/spec.md` и `game/move` (соло 60 с / ∞ budgets). Этот change: оба счётчика ∞ в соло; модал «1 минута / безлимит»; убрать distinct steps-exhausted end copy. **Titles** `### Requirement` / `#### Scenario` совпадают с main (OpenSpec validate); **bodies / THEN** — продуктовый канон (design D5).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-09 | modified — client GamePage (red ring; one-minute solo) |
| SC-PRESENCE-16 | modified — client UX (solo ∞ on steps and peeks) |
| SC-PRESENCE-19 | modified — client UX (solo unlimited modal copy) |
| SC-PRESENCE-21 | modified — client vitest (timer-only end copy; no steps-loss branch) |
| SC-PRESENCE-35 | modified — client vitest (no steps-exhausted copy) |

## MODIFIED Requirements

### Requirement: Turn countdown ring on presence

While phase is `playing` and a seated player’s seat holds the synchronized current turn with an active turn deadline, every client that shows that seat’s presence marker (including spectators) SHALL show a determinate circular progress for remaining turn time around the tourist image: full at the start of the deadline budget, empty at the deadline. While two or more eligible seats remain, that ring MUST use the same blue accent previously used for the static current-turn outline. While the solo one-minute budget of `game/move` applies (exactly one eligible seat: non-finished and not time-expired), that ring MUST use a red accent instead. Clients MUST NOT distinguish solo vs multiplayer for this accent solely by `turnBudgetSeconds` duration — multiplayer turn and solo budget are both **60 seconds**; use eligible-seat count (visible in synced state to seated players and spectators). The static blue outline / box-shadow that previously marked the current-turn seat MUST NOT be used. When the seat is not current turn, or has no active turn deadline (including time-expired or finished with cleared deadline), the turn countdown ring MUST NOT show progress as an active turn timer (track chrome may remain for layout stability per the reserved-marker requirement).

#### Scenario [SC-PRESENCE-08]: Current turn shows blue countdown instead of outline

- **GIVEN** a tourist room in phase `playing` with at least two non-finished seated players and a current-turn seat with a 60-second deadline
- **WHEN** any client views that seat’s presence marker
- **THEN** a blue circular countdown for remaining turn time is shown
- **AND** the tourist kind image remains visible in the marker
- **AND** no static blue outline-only turn indicator is shown on that marker

#### Scenario [SC-PRESENCE-09]: Solo budget shows red countdown

- **GIVEN** a tourist room in phase `playing` with exactly one eligible seat (non-finished and not time-expired) under the one-minute solo budget, and at least one spectator or finished/time-expired client may also be present
- **WHEN** any client (including a spectator) views that eligible seat’s presence marker
- **THEN** a red circular countdown for the remaining solo budget is shown
- **AND** the tourist kind image remains visible in the marker
- **AND** the red accent is derived from eligible-seat count in synced state, not from `turnBudgetSeconds` duration

### Requirement: Own step and peek counters beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s private **steps** and **peeks** counters in a **vertical stack to the right of** that user’s own presence avatar in the bottom HUD. When peeks are infinite (solo mode), the peeks counter MUST display an infinity indication. When steps are infinite (solo mode), the steps counter MUST also display an infinity indication. In multiplayer finite mode both counters MUST show numeric values. Other seated players’ and spectators’ clients MUST NOT show another seat’s step or peek counters. Spectators MUST NOT see step/peek counters for any seat. When the user’s finite budgets increase, the client SHOULD play a local “+N falls into the counter” animation for steps and peeks grants (turn grant and successful peek rewards) lasting approximately **two seconds**. The end-turn control MUST NOT sit in this budgets stack (`game/presence` end-turn requirement).

#### Scenario [SC-PRESENCE-15]: Seated user sees only own counters

- **GIVEN** two seated players in phase `playing` with different private budgets
- **WHEN** each views Game presence
- **THEN** each sees steps and peeks only on their own marker, stacked vertically to the right of their own avatar
- **AND** neither sees the other’s budget numbers on the opponent marker

#### Scenario [SC-PRESENCE-16]: Solo shows infinity only on peeks

- **GIVEN** the user is the sole non-finished seated player under infinite peeks and infinite steps
- **WHEN** the user views their presence marker
- **THEN** the peeks counter shows infinity
- **AND** the steps counter shows infinity
- **AND** other clients still do not see those budget values

#### Scenario [SC-PRESENCE-20]: Budget grant animation lasts about two seconds

- **GIVEN** the user’s finite steps or peeks budget increases while they view their own presence marker
- **WHEN** the local +N fall animation plays
- **THEN** the animation is visibly slower than a sub-second flash and completes in about two seconds
- **AND** other clients do not see that animation

### Requirement: Solo peeks-unlimited modal

When the user’s seat enters solo infinite-budgets mode (`game/move`), that user’s client MUST show a modal whose product Russian sense informs them that they are alone, that moves and peeks are unlimited, and that they have one minute. Product Russian default: «Вы один. Ходы и просмотры безлимитны. У вас 1 минута.» Closing the modal MUST leave the user in the room. Other clients MUST NOT show that modal.

#### Scenario [SC-PRESENCE-19]: Only the solo player sees the peeks-unlimited modal

- **GIVEN** a seated player becomes the sole non-finished seat and at least one spectator or finished seated client is present
- **WHEN** solo infinite steps and peeks apply
- **THEN** that player’s client shows the solo unlimited modal with the one-minute / unlimited-moves-and-peeks sense
- **AND** other clients do not show that modal

### Requirement: Distinct solo end copy for timer vs steps exhaustion

When the user’s seat becomes time-expired in solo (`game/move`), the client MUST show a single end modal for **timer expiry** (product Russian sense: tourists were not finished in time). There MUST NOT be a distinct steps-exhaustion end copy; solo steps are infinite so that branch MUST NOT apply. Closing the modal MUST leave the player in the room. Other clients MUST NOT show that modal.

#### Scenario [SC-PRESENCE-21]: Steps-loss and timer-loss show different copy

- **GIVEN** the user is the sole non-finished seated player
- **WHEN** the one-minute solo deadline elapses
- **THEN** the client shows the timer-expired end copy
- **AND** the client MUST NOT show a steps-exhausted end copy
- **AND** that modal is not shown to other clients

#### Scenario [SC-PRESENCE-35]: Steps-exhausted copy when only jail or start remainder

- **GIVEN** the user is under solo infinite steps with unfinished pieces only trapped and/or on start cells
- **WHEN** the seat is still within the one-minute solo deadline
- **THEN** the client does **not** open a steps-exhausted end modal
- **AND** play continues until timer expiry or finish
