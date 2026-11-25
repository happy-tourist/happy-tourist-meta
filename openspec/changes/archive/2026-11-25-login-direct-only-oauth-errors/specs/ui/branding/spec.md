# ui/branding — delta: без кнопки Login в шапке

Базовый канон: `openspec/specs/ui/branding/spec.md`. У гостя нет login affordance в wide toolbar и в burger; Login только по прямой ссылке (`auth/login`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-13 | covered-by-reuse (registered logout rightmost — unchanged) |
| SC-BRAND-22 | covered (client vitest — guest header has no login control) |
| SC-BRAND-24 | covered (client vitest — burger has no login for guest) |
| SC-BRAND-31 | covered (client vitest — wide logout icon-only when shown; no guest login control) |

## MODIFIED Requirements

### Requirement: Header account, theme, and session logout cluster

On authenticated non-Game screens with a **wide** viewport (section links visible in the toolbar), the shared header MUST place the applicable account-cluster controls with the rightmost session-related control among that toolbar cluster as specified below. For a **registered** (non-anonymous) user the cluster MUST include account and theme, and the session control MUST be session logout (rightmost among that cluster). For an **anonymous guest** the cluster MUST include theme only (account MUST remain omitted — cabinet stays registered-only) and MUST NOT show a login affordance and MUST NOT show session logout. In the **wide toolbar**, session logout (when shown for registered users) MUST be **icon-only** (no visible text label beside the icon) and MUST expose an accessible name (product key `auth.logout`). On **narrow** viewports where the section burger applies, those same cluster controls MUST appear **inside** the burger menu (not as separate toolbar icons); within the menu, session logout (registered only) MUST be last among that account cluster and MAY keep a visible text label as a menu item. On Game, session logout MUST NOT appear (leave-from-room is specified in `game/leave`). On auth screens in the shared `AUTH_ROUTE_NAMES` set (login / forgot-password / confirm-email / reset-password), session logout MUST NOT appear; theme MAY remain in the toolbar on auth screens without section navigation.

#### Scenario [SC-BRAND-13]: Logout rightmost off-Game (wide)

- **GIVEN** an authenticated registered user on Lobby on a wide viewport
- **WHEN** the shared header renders
- **THEN** session logout is rightmost relative to account and theme in the header toolbar

#### Scenario [SC-BRAND-22]: Guest header has theme only, no login (wide)

- **GIVEN** an authenticated anonymous guest on Lobby on a wide viewport
- **WHEN** the shared header renders
- **THEN** theme is shown in the header toolbar account cluster
- **AND** the account control is not shown for that guest
- **AND** session logout is not shown for that guest
- **AND** no login affordance is shown in the header toolbar
- **AND** session logout / login controls MUST NOT appear on auth screens in today’s `AUTH_ROUTE_NAMES` set (login / forgot-password / confirm-email / reset-password)

#### Scenario [SC-BRAND-31]: Wide toolbar session controls are icon-only

- **GIVEN** an authenticated registered user on a non-Game screen on a wide viewport where session logout is shown in the toolbar
- **WHEN** that session logout control is observed
- **THEN** the control shows an icon and no visible text label beside the icon
- **AND** the control still exposes an accessible name for logout

### Requirement: Mobile burger for section navigation

On narrow viewports for authenticated non-Game screens that use shared section navigation, Packs / Maps / Support / staff «Модерация» (when applicable) MUST be reachable via a burger or equivalent menu. That burger MUST be the **rightmost** control in the header toolbar. The same account-cluster rules as the wide toolbar MUST apply inside the burger: registered → account + theme + logout; anonymous → theme only (no account, no login affordance, no logout); session logout last among that cluster when present. Those controls MUST NOT remain as separate toolbar icons. Game screens MUST NOT use this section-burger fold (Leave and related chrome stay as specified in `game/leave` / existing Game header). Auth/login screens without section navigation MUST NOT gain this burger solely for theme.

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

#### Scenario [SC-BRAND-24]: Narrow burger has theme only, no login for guest

- **GIVEN** an authenticated anonymous guest on Lobby on a narrow viewport
- **WHEN** the user opens the header burger/menu
- **THEN** theme is available in the menu
- **AND** the account control is not shown for that guest
- **AND** session logout is not shown for that guest
- **AND** no login affordance is shown in the burger menu

### Requirement: Logo navigates to lobby outside Game

When the user activates the brand logo on a non-Game screen other than Lobby (including Account, Support, staff/admin support surfaces, and auth screens Login / forgot-password / confirm-email / reset-password), the client SHALL navigate toward the lobby screen. If the user is already on the Lobby screen, activating the logo MUST NOT navigate away and MUST NOT perform session logout. Existing route guards MAY keep or establish an anonymous session on lobby; that MUST NOT be treated as a brand-logo failure. Guards MUST NOT send the user to Login solely because of this logo navigation.

#### Scenario [SC-BRAND-02]: Logo opens lobby from Account or Support

- **GIVEN** an authenticated user on the Account or Support screen (or another non-Game authenticated screen other than Lobby)
- **WHEN** the user activates the brand logo
- **THEN** the client navigates the user to the lobby screen

#### Scenario [SC-BRAND-03]: Logo on Lobby is a no-op

- **GIVEN** the user is on the Lobby screen
- **WHEN** the user activates the brand logo
- **THEN** the user remains on the Lobby screen
- **AND** the session is not logged out

#### Scenario [SC-BRAND-04]: Auth screens logo navigates toward lobby

- **GIVEN** the user is on Login, forgot-password, confirm-email, or reset-password
- **WHEN** the user activates the brand logo
- **THEN** the client navigates toward the lobby screen
- **AND** if the user is not authenticated, existing auth guards may establish a guest session on lobby without requiring Login for that navigation
