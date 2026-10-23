# game/leave — delta: right leave control icon-only

Related unchanged: confirmation / consented leave rules (client confirm + room leave / server `onLeave` protocol as-is — no BE delta); brand logo as leave control on Game; no session logout on Game; no section links on Game. Brand mark theme assets: `ui/branding`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-13 | covered (`AppHeaderChrome` — Game right leave icon-only + `game.leave` aria) |
| SC-LEAVE-09 | MODIFIED — right leave remains; visible text label removed |

## MODIFIED Requirements

### Requirement: Explicit leave control on the right of Game header

While the user is on the Game screen of a tourist room, the shared header MUST show an explicit leave-from-room control on the **right**. The control MUST be **icon-only** (no visible text label such as «Выход из игры» beside the icon) and MUST expose an accessible name equivalent to leaving the game (product key `game.leave`, Russian «Выход из игры», localized per client locale files). Activating it MUST follow the same confirmation and consented-leave rules as the existing Game leave capability. This control is leave-from-room, NOT session logout.

#### Scenario [SC-LEAVE-09]: Leave control on the right on Game

- **GIVEN** the user is on the Game screen of a tourist room
- **WHEN** the shared header renders
- **THEN** an explicit leave-from-room control appears on the right
- **AND** activating it follows the confirm / consented-leave rules of this capability

#### Scenario [SC-LEAVE-13]: Right leave control is icon-only

- **GIVEN** the user is on the Game screen of a tourist room
- **WHEN** the right-side leave-from-room control is observed
- **THEN** the control shows an icon and no visible text label beside the icon
- **AND** the control exposes an accessible name for leaving the game (`game.leave`)
