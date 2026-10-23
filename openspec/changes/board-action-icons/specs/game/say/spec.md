## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-07 | modified (say = illustrated board-action icon; no Material speech-bubble / solid colored chrome) |
| SC-SAY-08 | unchanged (carried) |
| SC-SAY-15 | modified (same; affordance remains activatable) |

## MODIFIED Requirements

### Requirement: Own-marker affordance and preset picker

While the user is seated and connected on the Game screen, the system SHALL show a say send affordance only at the **top-center** of that user’s own presence marker (product clock sense: **12 o’clock**). The affordance MUST use the **illustrated board-action say icon** (not a Material speech-bubble glyph) and MUST NOT paint a solid colored circular chrome behind the icon (no solid fill and no medallion-like `box-shadow`). Activating the affordance MUST reveal exactly two preset choices with the display texts «Всем привет» and «Удачи». Choosing a preset MUST close the picker immediately and submit that preset’s identifier. Spectators MUST NOT see a send affordance. Other players’ markers MUST NOT show a send affordance for the local user. The affordance and its open picker MUST remain pointer- and touch-activatable: presence row overflow MUST NOT clip them out of hit-testing or hide the open picker, and the affordance hit target MUST be large enough for touch (~36 CSS pixels on the presence clock, at least about 32 CSS pixels on each side).

#### Scenario [SC-SAY-07]: Seated player opens picker on own marker

- **GIVEN** the user is seated and connected on the Game screen
- **WHEN** the user activates the say affordance (illustrated say icon) at the top-center of their own presence marker
- **THEN** two preset choices «Всем привет» and «Удачи» are shown
- **AND** the affordance has no solid colored circular chrome behind the icon (no solid fill / medallion-like `box-shadow`)
- **AND** choosing one closes the picker immediately

#### Scenario [SC-SAY-08]: Spectator has no send affordance

- **GIVEN** the user is a spectator on the Game screen
- **WHEN** the Game presence layout is shown
- **THEN** no say send affordance is available to that user

#### Scenario [SC-SAY-15]: Affordance and picker stay activatable

- **GIVEN** the user is seated and connected with their own presence marker visible (top or bottom row)
- **WHEN** the user taps or clicks the say affordance
- **THEN** the preset picker becomes visible without being clipped away by the presence row
- **AND** the affordance itself receives the activation (is not blocked by rings, avatar, or row overflow)
- **AND** the user can choose a preset from the picker
