# ui/branding — delta: shared header sections, staff moderation, breadcrumbs

Базовый канон: `openspec/specs/ui/branding/spec.md`. Follow-up: секции и staff «Модерация» в шапке; бургер; крошки; убрать секции с Lobby. Game pack title по центру — out of scope (`game/leave` для leave).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-11 | covered (vitest) |
| SC-BRAND-12 | covered (vitest) |
| SC-BRAND-13 | covered (vitest) |
| SC-BRAND-14 | covered (vitest) |
| SC-BRAND-15 | covered (vitest) |
| SC-BRAND-16 | covered (vitest) |
| SC-BRAND-17 | covered (vitest) |

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

On authenticated non-Game screens, the shared header MUST place account and theme controls to the left of session logout, with session logout rightmost among that cluster. On Game, session logout MUST NOT appear (leave-from-room is specified in `game/leave`).

#### Scenario [SC-BRAND-13]: Logout rightmost off-Game

- **GIVEN** an authenticated registered user on Lobby
- **WHEN** the shared header renders
- **THEN** session logout is rightmost relative to account and theme in the header

### Requirement: Mobile burger for section navigation

On narrow viewports, Packs / Maps / Support / staff «Модерация» (when applicable) MUST be reachable via a burger or equivalent menu. Account, theme, and session logout (off-Game) MAY remain visible in the header without requiring the burger.

#### Scenario [SC-BRAND-14]: Burger exposes sections on narrow viewport

- **GIVEN** an authenticated user on a narrow viewport non-Game screen
- **WHEN** the user opens the header burger/menu
- **THEN** Packs, Maps, and Support are available
- **AND** if the user is staff, «Модерация» is available there too

### Requirement: Lobby is not the sole section entry

The Lobby screen MUST NOT be the only place that exposes Packs / Maps / Support / staff moderation. Duplicate Lobby-only section toolbar controls that merely mirror the header MUST NOT remain as the primary product pattern once the header provides those links.

#### Scenario [SC-BRAND-15]: Lobby does not require a separate section toolbar

- **GIVEN** an authenticated user on Lobby with header section links available
- **WHEN** the Lobby page renders
- **THEN** a duplicate page-local section toolbar for Packs/Maps/Support is not required

### Requirement: Breadcrumbs under shared header

On content packs and maps routes (list, detail, editor, tasks as applicable), the client MUST show breadcrumbs **below** the shared elevated header chrome (page/layout zone). Breadcrumbs MUST include Lobby and the section root (Packs or Maps) and the current entity when applicable. Breadcrumbs MUST NOT render as a second row **inside** the elevated header bar where link colors blend into the header background.

#### Scenario [SC-BRAND-16]: Breadcrumbs under header on pack route

- **GIVEN** an authenticated user on a pack detail or tasks route
- **WHEN** the page renders
- **THEN** breadcrumbs appear under the shared header with a path including Lobby and Packs

#### Scenario [SC-BRAND-17]: Breadcrumbs outside elevated header bar

- **GIVEN** an authenticated user on a packs or maps content route with breadcrumbs
- **WHEN** the page renders
- **THEN** breadcrumbs are not inside the elevated shared header bar
- **AND** breadcrumbs sit below that header chrome in the page/layout zone
