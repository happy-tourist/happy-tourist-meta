## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LEAVE-01 | client UX (brand logo as leave control on Game) |
| SC-LEAVE-02 | covered-by-reuse (confirm rules unchanged) |
| SC-LEAVE-03 | covered-by-reuse |
| SC-LEAVE-04 | covered-by-reuse |
| SC-LEAVE-05 | covered-by-reuse |
| SC-LEAVE-06 | covered-by-reuse |
| SC-LEAVE-07 | covered-by-reuse |
| SC-LEAVE-08 | client UX (no Material logout; leave only via logo on Game) |

Related brand chrome: `ui/branding`. Confirm copy and consent path unchanged from prior leave behavior.

## MODIFIED Requirements

### Requirement: Exit control label

On the game screen the primary control that returns the user to the lobby SHALL be the shared-header **Happy Tourist brand logo** (image mark, no visible text label such as «Выход из игры» or «В лобби»), placed at the **left** side of the shared application header (theme toggle remains on the right). The control MUST expose an accessible name equivalent to leaving the game (product Russian accessible name «Выход из игры», localized per client locale files). The control MUST NOT use the Material Icons glyph `logout`. Confirm / immediate-leave rules for activating the control are unchanged by this requirement. Brand-logo presence on non-Game screens is specified in `ui/branding`.

#### Scenario [SC-LEAVE-01]: Exit control wording

- **GIVEN** the user is on the game screen of a tourist room
- **WHEN** the primary leave-to-lobby control is shown
- **THEN** the control is the brand logo without a visible text label
- **AND** it is positioned at the left of the shared application header
- **AND** it does not use the Material `logout` icon
- **AND** its accessible name is «Выход из игры» (localized per client locale files)

### Requirement: Exit control only while on Game

The shared application header MUST apply leave-to-lobby semantics (confirmation when required, then consented leave and navigation to the lobby) to the brand logo **only** while the user is on the Game screen of a tourist room. The Material Icons `logout` leave control MUST NOT appear in the shared header. On Lobby, auth, and other non-Game screens brand-logo click behavior is specified in `ui/branding` (navigate toward lobby, lobby no-op, or auth→lobby with possible guard bounce), not as a consented leave. The brand logo remains an interactive control on those screens for layout stability (`ui/branding` SC-BRAND-09). Theme toggle availability MUST remain unchanged (`ui/theme`).

#### Scenario [SC-LEAVE-08]: No exit control on Lobby

- **GIVEN** the user is on the Lobby screen
- **WHEN** the shared application header is shown
- **THEN** the Material `logout` leave control is not shown
- **AND** activating the brand logo does not perform a consented leave of a tourist room
- **AND** the theme toggle remains available
