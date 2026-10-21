## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-21 | covered — `AppHeaderChrome.test.ts` (layout `app-page-bg` + `app.scss` source-match theme×viewport fixed bg + transparent canvas + four JPG) |

Related underlay on card grids: delta `content/packs` (SC-PACK-285). Board holes / page background under tiles: `ui/theme` (SC-THEME-07).

## ADDED Requirements

### Requirement: App shell uses fixed theme and viewport page backgrounds

The shared application shell MUST show a full-bleed atmospheric background image on **all** application screens that use the shared layout (including Login, Lobby, Account, Support, Packs, Maps, staff/moderation, and Game). The image MUST be chosen from exactly four shipped assets: desktop-light, desktop-dark, mobile-light, mobile-dark (files `pack-bg-{desktop|mobile}-{light|dark}.jpg` under the client packs assets folder). The client MUST select **dark** vs **light** from the active Quasar Dark theme. The client MUST select **mobile** vs **desktop** by viewport width with the switch at Quasar **`sm`** (about **600** CSS px): below that width use the mobile asset; at or above use the desktop asset. This breakpoint MUST be independent of the header section burger (Quasar **`md`**). The background MUST remain visually fixed while page content scrolls. Page content surfaces MUST NOT paint an opaque page fill that fully hides the artwork (product sense: transparent page canvas over the shell background — including the main page area, its container, and shell chrome that would otherwise use the page background color, e.g. breadcrumbs). Elevated header / toolbar chrome MAY stay opaque. Tourist board tile fills and other opaque chrome MAY still cover the artwork where they already do. On Game, board holes MUST continue to show the current page background under tiles (compatible with `ui/theme` SC-THEME-07 — atmospheric shell art is the page background).

#### Scenario [SC-BRAND-21]: Shell page background matches theme and viewport and stays fixed

- **GIVEN** a user on any shared-layout screen (for example Lobby or Login) in light theme at a viewport width at or above about 600 CSS px
- **WHEN** the user views the page
- **THEN** the shell shows the desktop-light background as a full-bleed backdrop
- **AND** the main page canvas and breadcrumbs (or equivalent shell chrome using the page background color) do not paint an opaque fill that fully hides that artwork
- **AND WHEN** the user scrolls the page content
- **THEN** the background stays fixed relative to the viewport
- **AND WHEN** the theme is dark at the same wide viewport
- **THEN** the shell shows the desktop-dark background instead
- **AND WHEN** the viewport width is below about 600 CSS px in dark theme
- **THEN** the shell shows the mobile-dark background
- **AND** the header burger breakpoint remains unchanged (still folds at Quasar md / about 1024 CSS px)
