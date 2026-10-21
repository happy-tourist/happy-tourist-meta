# ui/branding — delta: catalog list page chrome (packs + maps) + create/underlay parity

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-25 | covered (ContentUnifiedList / ContentMaps / AppHeaderChrome — title+create band; logo without wordmark; map capacity rows) |
| SC-BRAND-26 | covered (ContentUnifiedList / ContentMaps — guest create on title band; no filter row) |
| SC-BRAND-27 | covered (ContentUnifiedList / ContentMaps / LobbyCreateWire — catalog create matches lobby Quasar primary) |
| SC-BRAND-28 | covered (AppHeaderChrome / ContentUnifiedList — catalog grid keeps SC-PACK-285 translucent underlay; no solid cream/navy override) |
| SC-BRAND-01 | covered — unchanged (logo without wordmark) |
| SC-BRAND-16…23 | covered — breadcrumbs + page-bg remain |
| SC-PACK-285 | related — translucent `.pack-card-grid` underlay MUST remain (see content/packs main) |

Related: `content/packs`, `content/maps`, `game/presence`. Title/subtitle geometry may follow design Visual Spec; **create button sizing tokens from mock are superseded** by lobby primary parity (SC-BRAND-27). Solid `page.canvas` cream/navy under the card grid is **out** — underlay stays SC-PACK-285 rgba.

## ADDED Requirements

### Requirement: Catalog list pages use hero title chrome

On the **packs catalog list** and the **maps list**, the client MUST present page chrome that reads as one composition with the shell illustration background: breadcrumbs (existing trail), a prominent page title, a short subtitle under the title, and the primary **create** affordance on the **same horizontal band as the title** (right side on wide viewports). Title and subtitle copy MUST remain the product i18n for those lists (packs: catalog title/subtitle; maps: maps title/subtitle). The shared header MUST continue to show the brand **image mark without** a visible wordmark text label (SC-BRAND-01 unchanged). Map list cards MUST keep the existing two capacity rows (not a mock-only «2-4» players pill as the sole capacity chrome). The create control MUST use the **same Quasar primary filled button chrome** as Lobby «Создать игру» (default elevated/`color="primary"` + `icon="add"` product sense) — MUST NOT use a separate custom height/padding/weight class that makes catalog create look like a different control family. The shared pack card grid on those lists MUST keep the **translucent** underlay from SC-PACK-285 (light `rgba(255,255,255,0.78)` / dark `rgba(18,18,18,0.78)`); the client MUST NOT paint an opaque cream/navy solid behind that grid that hides the shell illustration.

#### Scenario [SC-BRAND-25]: Packs and maps lists show title chrome with create

- **GIVEN** an authenticated user on the packs catalog list or the maps list
- **WHEN** the page renders
- **THEN** breadcrumbs, a prominent page title, and a subtitle are visible above the list
- **AND** the create affordance is on the same row/band as the title (not below the filter row alone)
- **AND** the shared header logo remains an image mark without a «СЧАСТЛИВЫЙ ТУРИСТ» wordmark label

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
- **WHEN** the `.pack-card-grid` (or equivalent shared pack card grid) renders
- **THEN** the grid underlay is translucent per SC-PACK-285 (artwork still visible through it)
- **AND** MUST NOT use an opaque solid cream/navy page-canvas fill that fully hides the illustration under the card block
