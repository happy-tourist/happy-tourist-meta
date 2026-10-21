# ui/branding — delta: catalog list page chrome (packs + maps) vs Visual Spec

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BRAND-25 | covered (ContentUnifiedList / ContentMaps / AppHeaderChrome — title+create band; logo without wordmark; map capacity rows) |
| SC-BRAND-26 | covered (ContentUnifiedList / ContentMaps — guest create on title band; no filter row) |
| SC-BRAND-01 | covered — unchanged (logo without wordmark) |
| SC-BRAND-16…23 | covered — breadcrumbs + page-bg remain; tighten presentation per design Visual Spec |

Related: `content/packs`, `content/maps`. Measurable tokens: change `design.md` → Visual Spec (from mock).

## ADDED Requirements

### Requirement: Catalog list pages use hero title chrome

On the **packs catalog list** and the **maps list**, the client MUST present page chrome that reads as one composition with the shell illustration background: breadcrumbs (existing trail), a prominent page title, a short subtitle under the title, and the primary **create** affordance on the **same horizontal band as the title** (right side on wide viewports). Title and subtitle copy MUST remain the product i18n for those lists (packs: catalog title/subtitle; maps: maps title/subtitle). The shared header MUST continue to show the brand **image mark without** a visible wordmark text label (SC-BRAND-01 unchanged). Map list cards MUST keep the existing two capacity rows (not a mock-only «2-4» players pill as the sole capacity chrome).

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
