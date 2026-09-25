# game/leave — delta: leave control on the right; logo also leave on Game

Базовый канон: `openspec/specs/game/leave/spec.md`. Follow-up: явный leave справа на Game; logo на Game тоже leave; session logout на Game нет. Название пака по центру — out of scope.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-09 | covered (vitest) |
| SC-LEAVE-10 | covered (vitest) |
| SC-LEAVE-11 | covered (vitest) |
| SC-LEAVE-12 | covered (vitest) |

Related: `ui/branding` (header chrome; section nav hidden on Game).

## ADDED Requirements

### Requirement: Explicit leave control on the right of Game header

While the user is on the Game screen of a tourist room, the shared header MUST show an explicit leave-from-room control on the **right**. Activating it MUST follow the same confirmation and consented-leave rules as the existing Game leave capability. This control is leave-from-room, NOT session logout.

#### Scenario [SC-LEAVE-09]: Leave control on the right on Game

- **GIVEN** the user is on the Game screen of a tourist room
- **WHEN** the shared header renders
- **THEN** an explicit leave-from-room control appears on the right
- **AND** activating it follows the confirm / consented-leave rules of this capability

### Requirement: Brand logo on Game also leaves

While the user is on the Game screen, activating the brand logo MUST perform the same leave-from-room flow as the right-side leave control (including confirmation when required).

#### Scenario [SC-LEAVE-10]: Logo leave matches right leave

- **GIVEN** the user is on the Game screen
- **WHEN** the user activates the brand logo
- **THEN** the same leave-from-room flow runs as for the right-side leave control

### Requirement: No session logout on Game

While on the Game screen, the shared header MUST NOT show session logout. Leave-from-room controls (right-side and logo) remain available per this capability.

#### Scenario [SC-LEAVE-11]: No session logout on Game

- **GIVEN** the user is on the Game screen
- **WHEN** the shared header renders
- **THEN** a session logout control is not shown

### Requirement: No section navigation on Game

While on the Game screen, Packs / Maps / Support / staff moderation header links MUST NOT appear (see `ui/branding`).

#### Scenario [SC-LEAVE-12]: No section links on Game header

- **GIVEN** the user is on the Game screen
- **WHEN** the shared header renders
- **THEN** Packs, Maps, Support, and staff «Модерация» header links are not shown

## MODIFIED Requirements

### Requirement: Exit control only while on Game

The shared application header MUST apply leave-to-lobby semantics (confirmation when required, then consented leave and navigation to the lobby) on the Game screen via the **right-side leave control** and via the **brand logo**. A session-logout control MUST NOT be used as the leave-from-room control. On Lobby, auth, and other non-Game screens, brand-logo click behavior remains as in `ui/branding` (navigate toward lobby, lobby no-op, or auth→lobby), not as consented leave. Theme toggle availability MUST remain unchanged (`ui/theme`).

#### Scenario [SC-LEAVE-08]: No exit control on Lobby

- **GIVEN** the user is on the Lobby screen (not Game)
- **WHEN** the shared header renders
- **THEN** the Game leave-from-room control is not shown as a leave-from-room action
- **AND** session logout MAY still appear per `ui/branding`
