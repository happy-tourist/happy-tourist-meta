## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-14 | modified (say at 12 o’clock / top-center; finish/ready top-left unchanged) |
| SC-PRESENCE-17 | unchanged (end-turn right-center ~3 o’clock) |
| SC-PRESENCE-30 | modified (focus at 9 o’clock / left-center; **keep historic scenario title**) |
| SC-PRESENCE-31 | unchanged (carried in Focus MODIFIED block) |
| SC-PRESENCE-32 | unchanged (carried) |
| SC-PRESENCE-33 | modified (say 12 + focus 9 without hit overlap; end-turn stays right; **keep historic requirement title**) |

## MODIFIED Requirements

### Requirement: Presence affordance and badge corners

When a seat has a synchronized finish place, every client SHALL show the finish place indicator at the **top-left** of that seat’s presence marker. While the local user may send a ready intent from the Game screen, the ready affordance MUST appear at the **top-left** of that user’s own presence marker only. While the local user may send a say intent, the say send affordance MUST appear at the **top-center** of that user’s own presence marker only (product clock sense: **12 o’clock** on the avatar; CSS placement per change `design.md` D3 — `top: -40px`, centered). When ready (or the own finish-place badge) and say are both visible on the own marker, their hit boxes MUST NOT overlap (raise say enough that centers stay clear). Other players’ markers MUST NOT show a say send affordance for the local user. Spectators MUST NOT see a say send affordance.

#### Scenario [SC-PRESENCE-14]: Finish, ready, and say corners

- **GIVEN** a seated connected user viewing Game with their own marker and at least one opponent marker that has finish place `1`
- **WHEN** the presence chrome is shown
- **THEN** the opponent’s finish place indicator is at the top-left of that opponent marker
- **AND** if the ready affordance is available for the local user, it is at the top-left of the local user’s marker
- **AND** the say send affordance is at the top-center (12 o’clock) of the local user’s marker only
- **AND** when ready (or own finish-place badge) and say are both visible, their hit boxes do not overlap
- **AND** opponent markers do not show a say send affordance for the local user

### Requirement: Focus actionable tourist control

While the local user is a seated current-turn player in phase `playing` and at least one of their unfinished free pieces can currently take an action (legal move with steps remaining, fresh peek, flipped peek, rescue, push, or return-from-finish when applicable), the Game presence chrome MUST show a **circular** icon-only control placed on the own presence avatar at the **left-center** edge (product clock sense: **9 o’clock**). Activating it MUST select / focus the **nearest** (or otherwise first) own unfinished free piece that has at least one such available action, without requiring a confirm dialog. When no own piece has an available action, the control MUST be hidden or disabled. Spectators and non-current seats MUST NOT see an enabled control for themselves. End-turn remains on the **right-center** edge (~**3 o’clock**) per the end-turn requirement.

#### Scenario [SC-PRESENCE-30]: Focus control sits between say and end-turn

- **GIVEN** it is the local seated user’s multiplayer turn with at least one actionable own tourist
- **WHEN** the own presence chrome is shown
- **THEN** a circular focus control is present at the left-center (9 o’clock) of the own avatar
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

On the local seated user’s own presence avatar, when say, focus, and/or end-turn controls are visible together, they MUST sit at distinct clock positions — say at **12** (top-center), focus at **9** (left-center), end-turn at **3** (right-center) — so their hit areas do not overlap. Say remains higher on the avatar than focus. This does not change say whitelist or `game/say` message rules, nor end-turn submission rules.

#### Scenario [SC-PRESENCE-33]: Say is above focus without overlap

- **GIVEN** it is the local seated user’s multiplayer turn with say available, at least one actionable own tourist (focus visible), and end-turn available
- **WHEN** the own presence chrome is shown
- **THEN** the say affordance is at top-center (12), the focus control at left-center (9), and end-turn at right-center (3)
- **AND** the three circular hit areas do not overlap
