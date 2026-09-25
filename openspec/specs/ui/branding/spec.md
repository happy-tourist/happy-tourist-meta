# ui/branding Specification

## Purpose

Брендинг chrome клиента Happy Tourist: логотип в общей шапке как единый переход в лобби, стабильный interactive control без скачка layout, document title и favicon вкладки, без размазанных page-level «В лобби» и без Quasar scaffold brand assets.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-01 | client UX (shared header logo) |
| SC-BRAND-02 | client UX (navigate to lobby) |
| SC-BRAND-03 | client UX (lobby noop) |
| SC-BRAND-04 | client UX (auth → lobby) |
| SC-BRAND-05 | client UX (no page back-to-lobby) |
| SC-BRAND-06 | client UX (document title) |
| SC-BRAND-07 | client UX (favicon) |
| SC-BRAND-08 | client UX (scaffold assets removed) |
| SC-BRAND-09 | client UX (stable interactive control) |
| SC-BRAND-10 | client UX (logo height ≥ 60px) |
| SC-BRAND-11 | covered (vitest) |
| SC-BRAND-12 | covered (vitest) |
| SC-BRAND-13 | covered (vitest) — wide / toolbar cluster |
| SC-BRAND-14 | covered (vitest) — revise: burger right + Acc/Theme/Logout in menu |
| SC-BRAND-15 | covered (vitest) |
| SC-BRAND-16 | covered (vitest) |
| SC-BRAND-17 | covered (vitest) — strengthen: page-container offset zone |
| SC-BRAND-18 | covered (vitest) |
| SC-BRAND-19 | covered (vitest) |

Related leave-on-Game: `game/leave`. Theme toggle: `ui/theme`. Related content chrome: `content/packs`, `content/maps`.

## Requirements

### Requirement: Brand logo in shared header

The shared application header MUST show the Happy Tourist brand logo on the **left** side on all application screens that use the shared header (including Login, Lobby, Account, Support, and Game). The logo MUST be an image mark without a visible text label such as «В лобби». Theme toggle availability on the right MUST remain unchanged (`ui/theme`). Match status centering on Game MUST remain unchanged (`game/presence`).

#### Scenario [SC-BRAND-01]: Logo left in shared header

- **GIVEN** the user is on any screen that uses the shared application header
- **WHEN** the header is shown
- **THEN** the Happy Tourist brand logo appears on the left
- **AND** the theme toggle remains available on the right

### Requirement: Logo navigates to lobby outside Game

When the user activates the brand logo on a non-Game screen other than Lobby (including Account, Support, staff/admin support surfaces, and auth screens Login / forgot-password / confirm-email / reset-password), the client SHALL navigate toward the lobby screen. If the user is already on the Lobby screen, activating the logo MUST NOT navigate away and MUST NOT perform session logout. Existing route guards MAY redirect an unauthenticated guest away from lobby (for example back to login); that bounce MUST NOT be treated as a brand-logo failure.

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
- **AND** if the user is not authenticated, existing auth guards may redirect them away from lobby (for example back to login)

### Requirement: No page-level back-to-lobby controls

Authenticated screens that previously exposed a dedicated «В лобби» (or equivalent) control to return to the lobby MUST NOT show that control. Returning to the lobby from those screens MUST be available via the shared header brand logo instead. Session logout on Lobby and nested «back to support» navigation MUST remain available where already productized.

#### Scenario [SC-BRAND-05]: Account and Support have no «В лобби» button

- **GIVEN** an authenticated user on the Account or Support screen
- **WHEN** the page chrome is shown
- **THEN** no dedicated «В лобби» button is shown on the page
- **AND** the shared header brand logo remains the way to reach the lobby

### Requirement: Browser tab title is Happy Tourist

The document title shown in the browser tab MUST be exactly `Happy Tourist` (product English brand string). It MUST NOT remain the Quasar scaffold product name such as `happy-tourist-client`.

#### Scenario [SC-BRAND-06]: Tab title

- **GIVEN** the client application is loaded in a browser
- **WHEN** the document title is read
- **THEN** the title is `Happy Tourist`

### Requirement: Favicon uses product ico

The browser tab favicon MUST use the product `.ico` brand icon. The client MUST NOT rely on Quasar scaffold multi-size PNG favicon links as the primary favicon set.

#### Scenario [SC-BRAND-07]: Favicon ico

- **GIVEN** the client application is loaded in a browser
- **WHEN** the favicon is requested
- **THEN** the product `.ico` favicon is used

### Requirement: Scaffold brand assets removed

Unused Quasar scaffold brand assets that are not the product logo or product favicon MUST be removed from the client package so they are not shipped or referenced (including the vertical Quasar logo asset and the scaffold PNG favicon size variants under the public icons folder). Dead scaffold pages that exist only to display the Quasar logo MAY be removed with those assets.

#### Scenario [SC-BRAND-08]: No Quasar scaffold logo or PNG favicons

- **GIVEN** the client package after this change
- **WHEN** brand and favicon assets are inspected
- **THEN** the Quasar vertical logo scaffold asset is absent
- **AND** the scaffold PNG favicon size files are absent
- **AND** the product header logo and product `.ico` favicon remain present

### Requirement: Stable interactive brand control

On every screen that uses the shared application header, the brand logo MUST be presented as the same kind of interactive control (not alternating between a non-interactive image-only presentation and a separate button presentation). Activating the control follows the navigate / no-op / leave rules of this capability and `game/leave`. The purpose is that the logo’s layout position MUST NOT jump when the user navigates between routes that previously used different presentation modes.

#### Scenario [SC-BRAND-09]: Same interactive control on auth and Game

- **GIVEN** the user can move between an auth screen and the Game screen (or other shared-header screens)
- **WHEN** the shared header brand logo is shown on each of those screens
- **THEN** the logo is an interactive control on each screen
- **AND** its horizontal position in the header does not jump solely because the route changed between decorative and clickable presentation modes

### Requirement: Logo height

The brand logo image in the shared header MUST be displayed at a height of at least 60 CSS pixels, preserving aspect ratio (`object-fit` contain / equivalent). The shared header MAY grow taller than the default Quasar toolbar height to accommodate the logo.

#### Scenario [SC-BRAND-10]: Logo at least 60px tall

- **GIVEN** the shared application header with the brand logo is shown
- **WHEN** the logo image size is observed
- **THEN** the logo is rendered at least 60 CSS pixels tall
- **AND** its aspect ratio is preserved

### Requirement: Shared header section navigation

On authenticated non-Game screens that use the shared header, the header MUST expose section links to the right of the brand logo for Packs, Maps, and Support. Moderator and admin MUST also see a **Модерация** control that opens the staff content moderation queue. On Game screens, section links and «Модерация» MUST NOT appear in the header.

#### Scenario [SC-BRAND-11]: Section links right of logo

- **GIVEN** an authenticated non-staff user on Lobby or Packs
- **WHEN** the shared header renders
- **THEN** Packs, Maps, and Support controls appear to the right of the brand logo

#### Scenario [SC-BRAND-12]: Staff moderation in header

- **GIVEN** an authenticated moderator or admin on a non-Game screen
- **WHEN** the shared header renders
- **THEN** a «Модерация» control is available in the header section cluster
- **AND** activating it opens the staff moderation queue

### Requirement: Header account, theme, and session logout cluster

On authenticated non-Game screens with a **wide** viewport (section links visible in the toolbar), the shared header MUST place account and theme controls to the left of session logout, with session logout rightmost among that toolbar cluster. On **narrow** viewports where the section burger applies, account / theme / session logout MUST appear **inside** the burger menu (not as separate toolbar icons); within the menu, session logout MUST be last among that account cluster. On Game, session logout MUST NOT appear (leave-from-room is specified in `game/leave`). On auth/login screens without section navigation, theme MAY remain in the toolbar.

#### Scenario [SC-BRAND-13]: Logout rightmost off-Game (wide)

- **GIVEN** an authenticated registered user on Lobby on a wide viewport
- **WHEN** the shared header renders
- **THEN** session logout is rightmost relative to account and theme in the header toolbar

### Requirement: Mobile burger for section navigation

On narrow viewports for authenticated non-Game screens that use shared section navigation, Packs / Maps / Support / staff «Модерация» (when applicable) MUST be reachable via a burger or equivalent menu. That burger MUST be the **rightmost** control in the header toolbar. Account, theme, and session logout (off-Game) MUST be reachable from the **same** burger menu on those screens and MUST NOT remain as separate toolbar controls. Game screens MUST NOT use this section-burger fold (Leave and related chrome stay as specified in `game/leave` / existing Game header). Auth/login screens without section navigation MUST NOT gain this burger solely for theme.

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

### Requirement: Lobby is not the sole section entry

The Lobby screen MUST NOT be the only place that exposes Packs / Maps / Support / staff moderation. Duplicate Lobby-only section toolbar controls that merely mirror the header MUST NOT remain as the primary product pattern once the header provides those links.

#### Scenario [SC-BRAND-15]: Lobby does not require a separate section toolbar

- **GIVEN** an authenticated user on Lobby with header section links available
- **WHEN** the Lobby page renders
- **THEN** a duplicate page-local section toolbar for Packs/Maps/Support is not required

### Requirement: Breadcrumbs under shared header

On content packs, maps, and staff/author moderation routes (list, detail, editor, tasks, queue as applicable), the client MUST show breadcrumbs **below** the shared elevated header chrome in the **page/layout zone that receives header height offset** (so fixed elevated header does not cover the crumbs). Breadcrumbs MUST include Lobby and the section root (Packs, Maps, or Модерация) and the current entity when applicable. Breadcrumbs MUST NOT render as a second row **inside** the elevated header bar where link colors blend into the header background. Breadcrumbs MUST NOT sit in document flow above the page container in a position that the fixed header overlays.

#### Scenario [SC-BRAND-16]: Breadcrumbs under header on pack route

- **GIVEN** an authenticated user on a pack detail or tasks route
- **WHEN** the page renders
- **THEN** breadcrumbs appear under the shared header with a path including Lobby and Packs

#### Scenario [SC-BRAND-17]: Breadcrumbs outside elevated header bar

- **GIVEN** an authenticated user on a packs or maps content route with breadcrumbs
- **WHEN** the page renders
- **THEN** breadcrumbs are not inside the elevated shared header bar
- **AND** breadcrumbs sit below that header chrome in the page/layout zone that is offset for the header (not covered by the fixed header)

#### Scenario [SC-BRAND-18]: Breadcrumbs on staff moderation queue

- **GIVEN** an authenticated moderator or admin on the staff content moderation queue
- **WHEN** the page renders
- **THEN** breadcrumbs include a path toward Lobby and Модерация (or equivalent staff moderation root)
