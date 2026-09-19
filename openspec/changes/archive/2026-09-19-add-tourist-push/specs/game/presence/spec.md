## Purpose

Delta этого change: UX end-turn — иконка справа по центру у своего аватара вместо текстового dock. Wire end-turn — main `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-17 | covered (client UX — end-turn icon right-center of own avatar) |
| SC-PRESENCE-18 | covered-by-reuse (solo still hides end-turn) |
| SC-PRESENCE-25 | covered (client UX — not in budgets stack; not above-panel dock label) |

Related: end-turn rules — `game/move`; say affordance placement analogy — `game/say` / presence marker chrome.

## MODIFIED Requirements

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show an **icon-only** end-turn control (Material `skip_next` or equivalent) on the **right** edge of that user’s own presence avatar, **vertically centered** on that edge (mirror of a say/dialog affordance centered on the **top** edge). Activating it MUST submit end-turn per `game/move` **immediately** — no confirmation dialog. The control MUST NOT show a visible text label (tooltips later are out of scope). The control MUST NOT appear in the budgets stack beside the avatar. The control MUST NOT appear as a labeled dock above the sticky bottom HUD. The control MUST NOT appear for spectators, for seats that are not current turn, during solo play, or for finished / time-expired seats.

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views Game chrome
- **THEN** an icon-only end-turn control is shown on the right edge of the own avatar, vertically centered
- **AND** activating it submits end-turn without a confirmation dialog
- **AND** no visible «Завершить ход» label is required on the control

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is the sole eligible seated player in solo play
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown

#### Scenario [SC-PRESENCE-25]: End-turn not inline with budgets

- **GIVEN** it is the local seated user’s turn in multi-seat play and end-turn is available
- **WHEN** the bottom HUD budgets stack beside the own avatar is shown
- **THEN** steps and peeks appear in that stack without the end-turn control inline
- **AND** the end-turn control remains on the right edge of the avatar (not a separate above-panel text dock)
