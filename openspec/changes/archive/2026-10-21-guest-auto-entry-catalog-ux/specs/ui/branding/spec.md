# ui/branding — delta: guest login icon; no phantom scroll with breadcrumbs

Базовый канон: `openspec/specs/ui/branding/spec.md`. Change: у anonymous вместо session logout — иконка входа; page shell с breadcrumbs без ложного вертикального скролла при коротком контенте.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-13 | covered (client vitest — `AppHeaderChrome` registered logout) |
| SC-BRAND-19 | covered (client vitest — burger Acc/Theme/Logout for registered) |
| SC-BRAND-22 | covered (client vitest — anonymous header login icon + auth omit) |
| SC-BRAND-23 | covered (client vitest — `--with-crumbs` class + app.scss flex/min-height) |
| SC-BRAND-24 | covered (client vitest — burger login for guest) |

Related: `auth/login` (Login upgrade path); packs/maps lists that show breadcrumbs.

## MODIFIED Requirements

### Requirement: Header account, theme, and session logout cluster

On authenticated non-Game screens with a **wide** viewport (section links visible in the toolbar), the shared header MUST place the applicable account-cluster controls to the left of the rightmost session control, with that session control rightmost among that toolbar cluster. For a **registered** (non-anonymous) user the cluster MUST include account and theme, and the session control MUST be session logout. For an **anonymous guest** the cluster MUST include theme only (account MUST remain omitted — cabinet stays registered-only) and the session control MUST be a login affordance (icon-only is sufficient; same control density as today’s logout) that opens the Login screen for upgrade — NOT session logout. On **narrow** viewports where the section burger applies, those same cluster controls MUST appear **inside** the burger menu (not as separate toolbar icons); within the menu, the session control MUST be last among that account cluster. On Game, neither logout nor the guest login affordance MUST appear (leave-from-room is specified in `game/leave`). On auth screens in the shared `AUTH_ROUTE_NAMES` set (login / forgot-password / confirm-email / reset-password), neither logout nor the guest login affordance MUST appear; theme MAY remain in the toolbar on auth screens without section navigation.

#### Scenario [SC-BRAND-13]: Logout rightmost off-Game (wide)

- **GIVEN** an authenticated registered user on Lobby on a wide viewport
- **WHEN** the shared header renders
- **THEN** session logout is rightmost relative to account and theme in the header toolbar

#### Scenario [SC-BRAND-22]: Guest sees login icon instead of logout (wide)

- **GIVEN** an authenticated anonymous guest on Lobby on a wide viewport
- **WHEN** the shared header renders
- **THEN** a login affordance is rightmost relative to theme in the header toolbar
- **AND** the account control is not shown for that guest
- **AND** session logout is not shown for that guest
- **AND** activating the login affordance opens the Login screen with a return target for the current path (for example `redirect` query)
- **AND** the login affordance MUST NOT appear on auth screens in today’s `AUTH_ROUTE_NAMES` set (login / forgot-password / confirm-email / reset-password) — same omission surface as session logout

### Requirement: Mobile burger for section navigation

On narrow viewports for authenticated non-Game screens that use shared section navigation, Packs / Maps / Support / staff «Модерация» (when applicable) MUST be reachable via a burger or equivalent menu. That burger MUST be the **rightmost** control in the header toolbar. The same account-cluster rules as the wide toolbar MUST apply inside the burger: registered → account + theme + logout; anonymous → theme + login affordance (no account); session control last among that cluster. Those controls MUST NOT remain as separate toolbar icons. Game screens MUST NOT use this section-burger fold (Leave and related chrome stay as specified in `game/leave` / existing Game header). Auth/login screens without section navigation MUST NOT gain this burger solely for theme.

#### Scenario [SC-BRAND-14]: Burger exposes sections on narrow viewport

- **GIVEN** an authenticated user on a narrow viewport non-Game screen
- **WHEN** the user opens the header burger/menu
- **THEN** Packs, Maps, and Support are available
- **AND** if the user is staff, «Модерация» is available there too

#### Scenario [SC-BRAND-19]: Narrow burger is rightmost and holds Acc/Theme/Logout

- **GIVEN** an authenticated registered user on Lobby on a narrow viewport
- **WHEN** the shared header renders
- **THEN** the burger control is rightmost in the toolbar
- **AND** account, theme, and session logout are not separate toolbar controls
- **AND** opening the burger exposes account, theme, and session logout (logout last among that cluster)

#### Scenario [SC-BRAND-24]: Narrow burger holds login for guest

- **GIVEN** an authenticated anonymous guest on Lobby on a narrow viewport
- **WHEN** the user opens the header burger/menu
- **THEN** theme and the login affordance are available in the menu
- **AND** the account control is not shown for that guest
- **AND** session logout is not shown for that guest
- **AND** the login affordance is last among that account cluster

## ADDED Requirements

### Requirement: No phantom page scroll when breadcrumbs and short content

On screens that show shared breadcrumbs above the page content (for example packs list and maps list), the application shell MUST NOT introduce a vertical page scrollbar solely because breadcrumbs occupy space above a page that still uses a near-viewport minimum height. When the visible content (breadcrumbs + page body) fits within the viewport below the header, the document/page MUST NOT scroll. Scrolling MUST still be available when content genuinely overflows the viewport.

#### Scenario [SC-BRAND-23]: Short packs list with crumbs does not force scroll

- **GIVEN** an authenticated user on the packs list with breadcrumbs visible and list content that fits below the header within the viewport
- **WHEN** the screen renders
- **THEN** the page does not require vertical scrolling solely due to breadcrumbs plus page minimum height
- **AND** the same rule applies to the maps list under the same conditions
