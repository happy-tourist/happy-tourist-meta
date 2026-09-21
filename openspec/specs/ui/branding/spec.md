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

Related leave-on-Game: `game/leave`. Theme toggle: `ui/theme`.

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
