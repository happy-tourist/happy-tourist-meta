## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-285 | covered — `AppHeaderChrome.test.ts` (`app.scss` source-match underlay rgba/pad/radius on `.pack-card-grid` only, not `.peek-chrome`) |

App-wide page background: see delta `ui/branding` (SC-BRAND-21).

## ADDED Requirements

### Requirement: Pack card grids sit on a translucent underlay

Wherever the client shows a wrapping row of pack-related cards via the shared pack card grid chrome (catalog packs, answer cards, task-set summaries, tasks — and the same shared grid used on maps list and lobby room lists), that card block MUST sit on a translucent underlay between the page background and the cards. The underlay MUST soften the artwork enough that card labels, badges, and actions remain readable in both light and dark themes (product sense roughly three-quarters opaque: light `rgba(255, 255, 255, 0.78)`; dark `rgba(18, 18, 18, 0.78)`; modest pad ~`0.75rem` and radius ~`8px` on the grid wrapper). Implementation sense: paint underlay CSS on the **layout** class `.pack-card-grid` only (already separated from the shared token selector `.pack-card-grid, .peek-chrome`). The underlay MUST apply to that shared pack card wrapping grid only — MUST NOT paint the same underlay on peek answer-pool / peek-slot / `.peek-chrome` that shares pack-card resting tokens but is not a list card grid. The underlay MUST NOT replace shared pack-card resting surface tokens on the tiles themselves (`--pack-card-bg` and siblings stay on tiles). Empty states without a card grid MAY omit the underlay.

#### Scenario [SC-PACK-285]: Card grid underlay keeps tiles readable over the background

- **GIVEN** an authenticated user on a surface with at least one visible shared pack card grid and an active app page background
- **WHEN** the user views the grid in light theme
- **THEN** a translucent underlay appears behind the card block (between artwork and cards)
- **AND** tile text and controls remain readable against the underlay
- **AND WHEN** the user switches to dark theme
- **THEN** the underlay uses a dark-theme translucent surface and tiles remain readable
- **AND** individual card resting chrome continues to use the shared pack-card surface tokens
