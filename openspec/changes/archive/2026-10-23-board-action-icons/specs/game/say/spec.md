## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-07 | modified (say 36×36; hit-center on avatar circumference at 12 o’clock) |
| SC-SAY-15 | modified (enlarged hit + picker fully visible; picker shifted right so left edge not clipped) |

## MODIFIED Requirements

### Requirement: Own-marker affordance and preset picker

While the user is seated and connected on the Game screen, the system SHALL show a say send affordance only with its **hit-center on the circumference** of that user’s own presence marker at the **top-center** (product clock sense: **12 o’clock**). The affordance MUST use the **illustrated board-action say icon** (not a Material speech-bubble glyph) and MUST NOT paint a solid colored circular chrome behind the icon. The illustrated glyph MUST render at **36×36 CSS pixels** for both the illustrated glyph and the hit target (glyph fills the hit). Activating the affordance MUST reveal exactly two preset choices with the display texts «Всем привет» and «Удачи». The open preset picker MUST be positioned so its **left edge is fully visible** (product default: shift the picker **to the right** enough that HUD / presence-row overflow does not crop the left side). Choosing a preset MUST close the picker immediately and submit that preset’s identifier. Spectators MUST NOT see a send affordance. Other players’ markers MUST NOT show a send affordance for the local user. The affordance and its open picker MUST remain pointer- and touch-activatable: presence row overflow MUST NOT clip them out of hit-testing or hide the open picker.

#### Scenario [SC-SAY-07]: Seated player opens picker on own marker

- **GIVEN** the user is seated and connected on the Game screen
- **WHEN** the user activates the say affordance (illustrated say icon) at the top-center circumference of their own presence marker
- **THEN** two preset choices «Всем привет» and «Удачи» are shown
- **AND** the affordance uses the 36×36 glyph-fills-hit family
- **AND** the affordance has no solid colored circular chrome behind the icon
- **AND** choosing one closes the picker immediately

#### Scenario [SC-SAY-08]: Spectator has no send affordance

- **GIVEN** the user is a spectator on the Game screen
- **WHEN** the Game presence layout is shown
- **THEN** no say send affordance is available to that user

#### Scenario [SC-SAY-15]: Affordance and picker stay activatable

- **GIVEN** the user is seated and connected with their own presence marker visible (top or bottom row)
- **WHEN** the user taps or clicks the say affordance
- **THEN** the preset picker becomes visible without being clipped away by the presence row
- **AND** the picker’s left edge is fully visible (not cropped by HUD overflow)
- **AND** the affordance itself receives the activation (is not blocked by rings, avatar, or row overflow)
- **AND** the user can choose a preset from the picker
