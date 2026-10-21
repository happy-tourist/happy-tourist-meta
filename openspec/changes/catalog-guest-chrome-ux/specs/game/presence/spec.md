# game/presence — delta: translucent sticky bottom game HUD

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-22 | covered — sticky pin unchanged; surface fill revised by SC-PRESENCE-34 |
| SC-PRESENCE-34 | covered — `AppHeaderChrome.test.ts` source-match `.game-hud` translucent rgba 0.78 |

Related: `ui/branding` / `content/packs` SC-PACK-285 (same alpha product sense for underlays over page art). Top presence row (`.presence-row--top`) out of scope — no opaque panel chrome there today.

## MODIFIED Requirements

### Requirement: Sticky bottom game HUD panel

While the user is on the Game screen as a **seated** viewer, the bottom HUD panel that holds the own presence marker, budgets beside the avatar, and personal strip (when present) MUST remain pinned to the bottom of the viewport so it stays visible while the board area above may scroll or resize. Opponent markers MUST use the row above the board, not this sticky panel — laid out tightly against the board (no large reserved empty band under the markers for say chrome). For a **spectator**, there MUST NOT be a bottom presence HUD row; markers stay above the board. The sticky bottom HUD surface MUST use a **translucent** fill so the shell / page artwork remains visible through it — product sense the same alpha as pack card-grid underlay (light `rgba(255, 255, 255, 0.78)`; dark `rgba(18, 18, 18, 0.78)`). The panel MUST NOT use an opaque solid `#fff` / `#121212` (or equivalent fully opaque theme canvas) that fully hides the art behind the HUD.

#### Scenario [SC-PRESENCE-22]: Bottom HUD stays pinned while board scrolls

- **GIVEN** the seated user is on the Game screen with the bottom HUD panel visible
- **WHEN** the board content area above the panel is scrolled or the board height changes
- **THEN** the bottom HUD panel remains pinned to the bottom of the viewport
- **AND** the user’s own marker and strip remain inside that panel
- **AND** opponent markers remain above the board

#### Scenario [SC-PRESENCE-34]: Bottom HUD underlay is translucent

- **GIVEN** a seated user on the Game screen over the shell page background
- **WHEN** the sticky bottom `.game-hud` (or equivalent seated HUD panel) renders
- **THEN** the HUD background is translucent (artwork visible through it) at approximately three-quarters opacity matching SC-PACK-285 underlay alphas
- **AND** MUST NOT present as a fully opaque white/near-black bar that completely hides the page art behind the panel
