# Delta: game/presence

Связано с main `openspec/specs/game/presence/spec.md` и `game/move` (одиночная vs соло-остаток). Этот change: HUD/chrome для **одиночной** (`maxSeats === 1`) — числа, без таймер-кольца, без end-turn, без unlimited-модала. Соло-остаток UX без изменений.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-09 | modified — red ring only solo leftover (maxSeats≥2), not single-player |
| SC-PRESENCE-16 | covered-by-reuse (solo leftover ∞) |
| SC-PRESENCE-18 | covered-by-reuse (hide end-turn when sole eligible) |
| SC-PRESENCE-19 | modified — unlimited modal only solo leftover, not maxSeats=1 |
| SC-PRESENCE-38 | added — client vitest (single-player numeric budgets) |
| SC-PRESENCE-39 | added — client vitest (single-player no turn ring) |
| SC-PRESENCE-40 | added — client vitest (single-player no unlimited modal) |
| SC-PRESENCE-41 | added — client vitest (+N animation on single-player refresh) |
| SC-PRESENCE-42 | added — client vitest (single-player hides end-turn) |

## MODIFIED Requirements

### Requirement: Turn countdown ring on presence

While phase is `playing` and a seated player’s seat holds the synchronized current turn with an active turn deadline, every client that shows that seat’s presence marker (including spectators) SHALL show a determinate circular progress for remaining turn time around the tourist image: full at the start of the deadline budget, empty at the deadline. While two or more eligible seats remain, that ring MUST use the same blue accent previously used for the static current-turn outline. While the solo leftover one-minute budget of `game/move` applies (`maxSeats ≥ 2` and exactly one eligible seat: non-finished and not time-expired), that ring MUST use a red accent instead. Clients MUST NOT distinguish solo leftover vs multiplayer for this accent solely by `turnBudgetSeconds` duration — multiplayer turn and solo leftover budget are both **60 seconds**; use eligible-seat count **and** that maxSeats is at least 2 (or equivalently: an active deadline under solo leftover rules). In a **single-player** room (`maxSeats === 1`) there is no active turn deadline, so the turn countdown ring MUST NOT be shown — clients MUST gate on `maxSeats === 1` (not solely an empty `turnUntil`). The static blue outline / box-shadow that previously marked the current-turn seat MUST NOT be used.

#### Scenario [SC-PRESENCE-08]: Current turn shows blue countdown instead of outline

- **GIVEN** a tourist room in phase `playing` with at least two non-finished seated players and a current-turn seat with a 60-second deadline
- **WHEN** any client views that seat’s presence marker
- **THEN** a blue circular countdown for remaining turn time is shown
- **AND** the tourist kind image remains visible in the marker
- **AND** no static blue outline-only turn indicator is shown on that marker

#### Scenario [SC-PRESENCE-09]: Solo budget shows red countdown

- **GIVEN** a tourist room with maxSeats ≥ 2 in phase `playing` with exactly one eligible seat (non-finished and not time-expired) under the one-minute solo leftover budget, and at least one spectator or finished/time-expired client may also be present
- **WHEN** any client views that seat’s presence marker
- **THEN** a red circular countdown for the remaining solo budget is shown
- **AND** the accent is red even though the budget duration equals the multiplayer 60 seconds

#### Scenario [SC-PRESENCE-39]: Single-player shows no turn countdown ring

- **GIVEN** a tourist room with maxSeats 1 in phase `playing` with one eligible seated player
- **WHEN** any client views that seat’s presence marker
- **THEN** no turn-deadline circular countdown ring is shown for that seat
- **AND** the client treats maxSeats 1 as no turn ring (not solely an empty `turnUntil`)

### Requirement: Own step and peek counters beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s private **steps** and **peeks** counters in a **vertical stack to the right of** that user’s own presence avatar in the bottom HUD. When peeks are infinite (solo leftover mode), the peeks counter MUST display an infinity indication. When steps are infinite (solo leftover mode), the steps counter MUST also display an infinity indication. In multiplayer finite mode and in **single-player** (`maxSeats === 1`, finite budgets) both counters MUST show numeric values. Other seated players’ and spectators’ clients MUST NOT show another seat’s step or peek counters. Spectators MUST NOT see step/peek counters for any seat. When the user’s finite budgets increase, the client SHOULD play a local “+N falls into the counter” animation for steps and peeks grants (turn grant, single-player no-actions refresh, and successful peek rewards) lasting approximately **two seconds**. The end-turn control MUST NOT sit in this budgets stack (`game/presence` end-turn requirement).

#### Scenario [SC-PRESENCE-15]: Seated user sees only own counters

- **GIVEN** two seated players in phase `playing` with different private budgets
- **WHEN** each views Game presence
- **THEN** each sees steps and peeks only on their own marker, stacked vertically to the right of their own avatar
- **AND** neither sees the other’s budget numbers on the opponent marker

#### Scenario [SC-PRESENCE-16]: Solo shows infinity only on peeks

- **GIVEN** the user is the sole non-finished seated player under infinite peeks and infinite steps in a maxSeats ≥ 2 room
- **WHEN** the user views their presence marker
- **THEN** the peeks counter shows infinity
- **AND** the steps counter shows infinity
- **AND** other clients still do not see those budget values

#### Scenario [SC-PRESENCE-20]: Budget grant animation lasts about two seconds

- **GIVEN** the user’s finite steps or peeks budget increases while they view their own presence marker
- **WHEN** the local +N fall animation plays
- **THEN** the animation is visibly slower than a sub-second flash and completes in about two seconds
- **AND** other clients do not see that animation

#### Scenario [SC-PRESENCE-38]: Single-player shows numeric budgets

- **GIVEN** the user is the seated player in a maxSeats-1 `playing` room under finite steps and peeks
- **WHEN** the user views their presence marker
- **THEN** the steps and peeks counters show numeric values (not infinity)

#### Scenario [SC-PRESENCE-41]: Single-player refresh plays grant animation

- **GIVEN** the user’s finite steps and peeks increase because the server refreshed +1/+1 after no available actions in a maxSeats-1 room
- **WHEN** the local +N fall animation plays
- **THEN** the animation completes in about two seconds for the increased counters
- **AND** other clients do not see that animation

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show an **icon-only** end-turn control whose **hit-center sits on the circumference** of that user’s own presence marker at the **right-center** edge (product clock sense: **3 o’clock**). The control MUST use the **illustrated board-action end-turn icon** and MUST NOT paint a solid colored circular chrome behind the icon. Glyph and hit MUST both be **36×36 CSS pixels** (glyph fills the hit) while remaining non-overlapping with say when say is visible. Activating it MUST submit end-turn per `game/move` **immediately** — no confirmation dialog. The control MUST NOT show a visible text label (tooltips later are out of scope). The control MUST NOT appear in the budgets stack beside the avatar. The control MUST NOT appear as a labeled dock above the sticky bottom HUD. The control MUST NOT appear for spectators, for seats that are not current turn, during solo leftover play, during **single-player** (`maxSeats === 1`), or for finished / time-expired seats. While a peek modal is open, the control MUST NOT be activatable (board-busy / peek lock as already required).

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room and no peek modal is open
- **WHEN** the user views Game chrome
- **THEN** an icon-only end-turn control with the illustrated end-turn icon is shown with its hit-center on the own avatar circumference at 3 o’clock
- **AND** the glyph / hit use the 36×36 glyph-fills-hit family
- **AND** the control has no solid colored circular chrome behind the icon
- **AND** activating it submits end-turn without a confirmation dialog

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is the sole eligible seated player in solo leftover play (maxSeats ≥ 2)
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown

#### Scenario [SC-PRESENCE-42]: Single-player hides end-turn

- **GIVEN** it is the seated user’s turn in a maxSeats-1 `playing` room under finite budgets (not infinite)
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown
- **AND** the client does not treat sole-eligible alone as sufficient to show end-turn

#### Scenario [SC-PRESENCE-25]: End-turn not inline with budgets

- **GIVEN** it is the local seated user’s turn in multi-seat play and end-turn is available
- **WHEN** the bottom HUD budgets stack beside the own avatar is shown
- **THEN** steps and peeks appear in that stack without the end-turn control inline
- **AND** the end-turn control remains on the right edge of the avatar (not a separate above-panel text dock)
- **AND** when focus is available it sits below end-turn on the marker (not in that budgets stack)

### Requirement: Solo peeks-unlimited modal

When the user’s seat enters solo leftover infinite-budgets mode (`game/move`, maxSeats ≥ 2 and exactly one eligible seat), that user’s client MUST show a modal whose product Russian sense informs them that they are alone, that moves and peeks are unlimited, and that they have one minute. Product Russian default: «Вы один. Ходы и просмотры безлимитны. У вас 1 минута.» Closing the modal MUST leave the user in the room. Other clients MUST NOT show that modal. Entering **single-player** play (`maxSeats === 1`) MUST NOT show that modal — including if `budgetsInfinite` flips true while maxSeats remains 1.

#### Scenario [SC-PRESENCE-19]: Only the solo player sees the peeks-unlimited modal

- **GIVEN** a seated player becomes the sole non-finished seat in a maxSeats ≥ 2 room and at least one spectator or finished seated client is present
- **WHEN** solo leftover infinite steps and peeks apply
- **THEN** that player’s client shows the solo unlimited modal with the one-minute / unlimited-moves-and-peeks sense
- **AND** other clients do not show that modal

#### Scenario [SC-PRESENCE-40]: Single-player does not show unlimited modal

- **GIVEN** a seated player enters phase `playing` in a maxSeats-1 room under finite budgets
- **WHEN** that client observes budgets become available for play
- **THEN** the solo unlimited-budgets modal is not shown
- **AND** a `budgetsInfinite` false→true transition MUST NOT open that modal while maxSeats is 1
