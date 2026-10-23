# ui/branding — delta: theme wordmark + icon-only session controls

Related unchanged: navigation/noop/leave modes of the brand control (`game/leave` on Game); section links; burger text labels; favicon; document title; page backgrounds (`ui/theme`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-29 | covered (`AppHeaderChrome` — theme toggles `logo-light`/`logo-dark`; no `logo.png`) |
| SC-BRAND-30 | covered (`AppHeaderChrome` — `.brand-logo` height 28px; toolbar min-height 40) |
| SC-BRAND-31 | covered (`AppHeaderChrome` — toolbar login/logout icon-only + aria) |
| SC-BRAND-32 | covered (`AccountLogout` — Account session logout icon-only + aria) |
| SC-BRAND-08 | unchanged — Quasar scaffold assets (not the product mascot/`logo.png`) |
| SC-BRAND-10 | MODIFIED — 28px with inset in header content band |
| SC-BRAND-13 | strengthen — logout remains rightmost; visible label removed (SC-BRAND-31) |
| SC-BRAND-22 | strengthen — guest login MUST be icon-only in wide toolbar |
| SC-BRAND-25 | MODIFIED — no separate DOM wordmark label; typographic PNG mark allowed |

## MODIFIED Requirements

### Requirement: Brand logo in shared header

The shared application header MUST show the Happy Tourist brand logo on the **left** side on all application screens that use the shared header (including Login, Lobby, Account, Support, and Game). The logo MUST be an image mark. The client MUST NOT show a separate visible text control next to the logo such as «В лобби» or «СЧАСТЛИВЫЙ ТУРИСТ». A typographic wordmark **inside** the logo image asset is allowed. Theme toggle availability on the right MUST remain unchanged (`ui/theme`). Match status centering on Game MUST remain unchanged (`game/presence`).

#### Scenario [SC-BRAND-01]: Logo left in shared header

- **GIVEN** the user is on any screen that uses the shared application header
- **WHEN** the header is shown
- **THEN** the Happy Tourist brand logo appears on the left
- **AND** the theme toggle remains available on the right
- **AND** no separate brand wordmark text control appears beside the logo

### Requirement: Logo height

The brand logo image in the shared header MUST be 28 CSS pixels tall, preserving aspect ratio (`object-fit` contain / equivalent), with clear top/bottom inset inside a shared header content band of about 40 CSS pixels min-height. The shared header MUST NOT keep the previous 60px mark / 64px toolbar sizing.

#### Scenario [SC-BRAND-10]: Logo 28px tall

- **GIVEN** the shared application header with the brand logo is shown
- **WHEN** the logo image size is observed
- **THEN** the logo is rendered 28 CSS pixels tall
- **AND** the header content band is about 40 CSS pixels min-height (inset remains around the mark)
- **AND** its aspect ratio is preserved

### Requirement: Catalog list pages use hero title chrome

On the **packs catalog list** and the **maps list**, the client MUST present page chrome that reads as one composition with the shell illustration background: breadcrumbs (existing trail), a prominent page title, a short subtitle under the title, and the primary **create** affordance on the **same horizontal band as the title** (right side on wide viewports). Title and subtitle copy MUST remain the product i18n for those lists (packs: catalog title/subtitle; maps: maps title/subtitle). The shared header MUST continue to show the brand image mark without a **separate** visible wordmark text label beside it (SC-BRAND-01). Map list cards MUST keep the existing two capacity rows (not a mock-only «2-4» players pill as the sole capacity chrome). The create control MUST use the **same Quasar primary filled button chrome** as Lobby «Создать игру» (default elevated/`color="primary"` + `icon="add"` product sense) — MUST NOT use a separate custom height/padding/weight class that makes catalog create look like a different control family. The shared pack card grid on those lists MUST keep the **translucent** underlay from SC-PACK-285 (light translucent white / dark translucent near-black at about three-quarters opacity); the client MUST NOT paint an opaque cream/navy solid behind that grid that hides the shell illustration.

#### Scenario [SC-BRAND-25]: Packs and maps lists show title chrome with create

- **GIVEN** an authenticated user on the packs catalog list or the maps list
- **WHEN** the page renders
- **THEN** breadcrumbs, a prominent page title, and a subtitle are visible above the list
- **AND** the create affordance is on the same row/band as the title (not below the filter row alone)
- **AND** the shared header has no separate «СЧАСТЛИВЫЙ ТУРИСТ» (or equivalent) text label beside the logo image

#### Scenario [SC-BRAND-26]: Guest catalog lists omit filter row beside create

- **GIVEN** an anonymous guest on the packs catalog list or the maps list
- **WHEN** the page renders
- **THEN** the create affordance remains visible on the title band
- **AND** no «Все» / filter controls appear on that page
- **AND** the filter row chrome itself is absent (not an empty gutter under the title band)
- **AND** breadcrumbs and title/subtitle remain visible over the shell background

#### Scenario [SC-BRAND-27]: Catalog create matches lobby create chrome

- **GIVEN** the packs or maps list create control and the Lobby «Создать игру» control
- **WHEN** both are compared in the same theme
- **THEN** catalog create uses Quasar primary filled chrome equivalent to Lobby create (not a custom unelevated denser/taller tokenized variant)
- **AND** create remains on the title band (SC-BRAND-25)

#### Scenario [SC-BRAND-28]: Catalog card grid keeps translucent underlay

- **GIVEN** the packs catalog list or maps list over the shell illustration background
- **WHEN** the shared pack card grid on that list renders
- **THEN** the grid underlay is translucent per SC-PACK-285 (artwork still visible through it)
- **AND** MUST NOT use an opaque solid cream/navy page-canvas fill that fully hides the illustration under the card block

### Requirement: Header account, theme, and session logout cluster

On authenticated non-Game screens with a **wide** viewport (section links visible in the toolbar), the shared header MUST place the applicable account-cluster controls to the left of the rightmost session control, with that session control rightmost among that toolbar cluster. For a **registered** (non-anonymous) user the cluster MUST include account and theme, and the session control MUST be session logout. For an **anonymous guest** the cluster MUST include theme only (account MUST remain omitted — cabinet stays registered-only) and the session control MUST be a login affordance that opens the Login screen for upgrade — NOT session logout. In the **wide toolbar**, both session logout and the guest login affordance MUST be **icon-only** (no visible text label beside the icon); each MUST expose an accessible name (product keys `auth.logout` / `auth.login`). On **narrow** viewports where the section burger applies, those same cluster controls MUST appear **inside** the burger menu (not as separate toolbar icons); within the menu, the session control MUST be last among that account cluster and MAY keep a visible text label as a menu item. On Game, neither logout nor the guest login affordance MUST appear (leave-from-room is specified in `game/leave`). On auth screens in the shared `AUTH_ROUTE_NAMES` set (login / forgot-password / confirm-email / reset-password), neither logout nor the guest login affordance MUST appear; theme MAY remain in the toolbar on auth screens without section navigation.

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

#### Scenario [SC-BRAND-31]: Wide toolbar session controls are icon-only

- **GIVEN** an authenticated user on a non-Game screen on a wide viewport where the session login or logout control is shown in the toolbar
- **WHEN** that session control is observed
- **THEN** the control shows an icon and no visible text label beside the icon
- **AND** the control still exposes an accessible name for login or logout as applicable

## ADDED Requirements

### Requirement: Theme-aware brand wordmark assets

The shared header brand logo MUST use a **light-theme** wordmark asset when the client is in light theme and a **dark-theme** wordmark asset when the client is in dark theme. Canonical shipped files under the client brand assets folder MUST be `logo-light.png` (dark letterforms for light UI) and `logo-dark.png` (light letterforms for dark UI), each cropped tightly to the letterforms (minimal transparent padding). The previous single mascot header logo asset (`logo.png`) MUST NOT remain in the client package as the header brand mark (and MUST NOT be shipped unused under the brand assets folder for header use). Product favicon rules in this capability are unchanged.

#### Scenario [SC-BRAND-29]: Header mark switches with theme

- **GIVEN** the user is on a screen that uses the shared application header
- **WHEN** the client theme is light
- **THEN** the header shows the light-theme brand wordmark (`logo-light.png`)
- **WHEN** the client theme is dark
- **THEN** the header shows the dark-theme brand wordmark (`logo-dark.png`)
- **AND** the old single mascot header logo asset is not used as the header mark

### Requirement: Account session logout is icon-only

On the Account (cabinet) screen, the session logout control MUST be **icon-only** (no visible text label such as «Выйти»). The control MUST expose an accessible name equivalent to session logout (product key `auth.logout`). Session logout behavior (end session and navigate to login) MUST remain unchanged.

#### Scenario [SC-BRAND-32]: Account logout has no visible label

- **GIVEN** an authenticated registered user on the Account screen
- **WHEN** the session logout control is shown
- **THEN** the control shows an icon and no visible text label
- **AND** the control exposes an accessible name for logout
