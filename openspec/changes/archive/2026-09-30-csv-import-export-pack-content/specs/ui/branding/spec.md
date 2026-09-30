## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-20 | covered (`AppHeaderChrome` Lobby crumb → lobby) |

Related: breadcrumbs under header — main `ui/branding` (SC-BRAND-16…18). This change requires the Lobby crumb to navigate to the lobby route.

## ADDED Requirements

### Requirement: Lobby breadcrumb navigates to lobby

Wherever content packs, maps, or staff/author moderation breadcrumbs include a **Lobby** crumb, activating that crumb MUST navigate the user to the lobby screen. The Lobby crumb MUST NOT be a non-interactive label when other crumbs in the same trail are navigable.

#### Scenario [SC-BRAND-20]: Lobby crumb opens lobby from a content route

- **GIVEN** an authenticated user on a packs, maps, or moderation route that shows breadcrumbs including Lobby
- **WHEN** the user activates the Lobby crumb
- **THEN** the client navigates to the lobby screen
