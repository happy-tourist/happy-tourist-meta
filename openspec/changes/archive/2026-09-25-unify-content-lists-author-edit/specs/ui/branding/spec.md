# ui/branding — delta: shared header sections, staff moderation, breadcrumbs

Базовый канон: `openspec/specs/ui/branding/spec.md`. Follow-up: секции и staff «Модерация» в шапке; бургер; крошки; убрать секции с Lobby. Game pack title по центру — out of scope (`game/leave` для leave).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-11 | covered (vitest) |
| SC-BRAND-12 | covered (vitest) |
| SC-BRAND-13 | covered (vitest) — wide / toolbar cluster |
| SC-BRAND-14 | covered (vitest) — revise: burger right + Acc/Theme/Logout in menu |
| SC-BRAND-15 | covered (vitest) |
| SC-BRAND-16 | covered (vitest) |
| SC-BRAND-17 | covered (vitest) — strengthen: page-container offset zone |
| SC-BRAND-18 | covered (vitest) |
| SC-BRAND-19 | covered (vitest) |

Related: `content/packs`, `content/maps`, `game/leave`, `ui/theme`.

## ADDED Requirements

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
