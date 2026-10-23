## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-07 | modified (say affordance top-center / 12 o’clock) |
| SC-SAY-08 | unchanged (carried in MODIFIED block) |
| SC-SAY-15 | unchanged (carried; overflow / hit size still apply) |

## MODIFIED Requirements

### Requirement: Own-marker affordance and preset picker

While the user is seated and connected on the Game screen, the system SHALL show a speech-bubble affordance only at the **top-center** of that user’s own presence marker (product clock sense: **12 o’clock**). Activating the affordance MUST reveal exactly two preset choices with the display texts «Всем привет» and «Удачи». Choosing a preset MUST close the picker immediately and submit that preset’s identifier. Spectators MUST NOT see a send affordance. Other players’ markers MUST NOT show a send affordance for the local user. The affordance and its open picker MUST remain pointer- and touch-activatable: presence row overflow MUST NOT clip them out of hit-testing or hide the open picker, and the affordance hit target MUST be large enough for touch (at least about 32 CSS pixels on each side).

#### Scenario [SC-SAY-07]: Seated player opens picker on own marker

- **GIVEN** the user is seated and connected on the Game screen
- **WHEN** the user activates the speech-bubble affordance at the top-center of their own presence marker
- **THEN** two preset choices «Всем привет» and «Удачи» are shown
- **AND** choosing one closes the picker immediately

#### Scenario [SC-SAY-08]: Spectator has no send affordance

- **GIVEN** the user is a spectator on the Game screen
- **WHEN** the Game presence layout is shown
- **THEN** no say send affordance is available to that user

#### Scenario [SC-SAY-15]: Affordance and picker stay activatable

- **GIVEN** the user is seated and connected with their own presence marker visible (top or bottom row)
- **WHEN** the user taps or clicks the speech-bubble affordance
- **THEN** the preset picker becomes visible without being clipped away by the presence row
- **AND** the affordance itself receives the activation (is not blocked by rings, avatar, or row overflow)
- **AND** the user can choose a preset from the picker
