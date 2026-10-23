## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-17 | modified (end-turn = illustrated icon; no Material `skip_next` / solid colored chrome) |
| SC-PRESENCE-18 | unchanged (carried) |
| SC-PRESENCE-25 | unchanged (carried) |
| SC-PRESENCE-30 | modified (focus = illustrated icon; no solid colored circular chrome) |
| SC-PRESENCE-31 | unchanged (carried) |
| SC-PRESENCE-32 | unchanged (carried) |
| SC-PRESENCE-33 | modified (hit areas stay ~circular; icons illustrated without solid fill chrome) |

## MODIFIED Requirements

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show an **icon-only** end-turn control on the **right** edge of that user’s own presence avatar, **vertically centered** on that edge (mirror of a say/dialog affordance centered on the **top** edge). The control MUST use the **illustrated board-action end-turn icon** (not a Material `skip_next` glyph) and MUST NOT paint a solid colored circular chrome behind the icon (no solid fill and no medallion-like `box-shadow`); the hit target remains ~36 CSS pixels. Activating it MUST submit end-turn per `game/move` **immediately** — no confirmation dialog. The control MUST NOT show a visible text label (tooltips later are out of scope). The control MUST NOT appear in the budgets stack beside the avatar. The control MUST NOT appear as a labeled dock above the sticky bottom HUD. The control MUST NOT appear for spectators, for seats that are not current turn, during solo play, or for finished / time-expired seats.

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views Game chrome
- **THEN** an icon-only end-turn control with the illustrated end-turn icon is shown on the right edge of the own avatar, vertically centered
- **AND** the control has no solid colored circular chrome behind the icon (no solid fill / medallion-like `box-shadow`)
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

### Requirement: Focus actionable tourist control

While the local user is a seated current-turn player in phase `playing` and at least one of their unfinished free pieces can currently take an action (legal move with steps remaining, fresh peek, flipped peek, rescue, push, or return-from-finish when applicable), the Game presence chrome MUST show an **icon-only** focus control placed on the own presence avatar at the **left-center** edge (product clock sense: **9 o’clock**). The control MUST use the **illustrated board-action focus icon** (not a Material `center_focus_strong` glyph) and MUST NOT paint a solid colored circular chrome behind the icon (no solid fill and no medallion-like `box-shadow`); the hit target remains ~36 CSS pixels. Activating it MUST select / focus the **nearest** (or otherwise first) own unfinished free piece that has at least one such available action, without requiring a confirm dialog. When no own piece has an available action, the control MUST be hidden or disabled. Spectators and non-current seats MUST NOT see an enabled control for themselves. End-turn remains on the **right-center** edge (~**3 o’clock**) per the end-turn requirement.

#### Scenario [SC-PRESENCE-30]: Focus control sits between say and end-turn

- **GIVEN** it is the local seated user’s multiplayer turn with at least one actionable own tourist
- **WHEN** the own presence chrome is shown
- **THEN** a focus control with the illustrated focus icon is present at the left-center (9 o’clock) of the own avatar
- **AND** the control has no solid colored circular chrome behind the icon (no solid fill / medallion-like `box-shadow`)
- **AND** say (when visible) is at top-center (12) and end-turn (when visible) at right-center (3)

#### Scenario [SC-PRESENCE-31]: Focus selects nearest actionable tourist

- **GIVEN** it is the local user’s turn with two own unfinished free pieces where only piece B can peek or move and piece A cannot
- **WHEN** the user activates the focus control
- **THEN** piece B becomes selected / focused for board actions
- **AND** piece A is not selected

#### Scenario [SC-PRESENCE-32]: Focus hidden when nothing actionable

- **GIVEN** it is the local user’s turn and no own unfinished free piece has a legal move, peek, rescue, push, or return
- **WHEN** the own presence chrome is shown
- **THEN** the focus control is not available as an enabled action

### Requirement: Say affordance sits above the focus control

On the local seated user’s own presence avatar, when say, focus, and/or end-turn controls are visible together, they MUST sit at distinct clock positions — say at **12** (top-center), focus at **9** (left-center), end-turn at **3** (right-center) — so their hit areas do not overlap. Say remains higher on the avatar than focus. This does not change say whitelist or `game/say` message rules, nor end-turn submission rules. The three hit targets remain large enough for touch (~36 CSS pixels) even when the illustrated icons are not perfect circles.

#### Scenario [SC-PRESENCE-33]: Say is above focus without overlap

- **GIVEN** it is the local seated user’s multiplayer turn with say available, at least one actionable own tourist (focus visible), and end-turn available
- **WHEN** the own presence chrome is shown
- **THEN** the say affordance is at top-center (12), the focus control at left-center (9), and end-turn at right-center (3)
- **AND** the three hit areas do not overlap
