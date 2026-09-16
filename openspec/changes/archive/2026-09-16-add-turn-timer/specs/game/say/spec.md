## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-07 | covered (client say affordance top-right on own marker) |
| SC-SAY-11 | covered (client bubbles toward board — replaces side-slot stack) |
| SC-SAY-12 | covered (client bubbles toward board; newer closer to avatar) |
| SC-SAY-15 | covered (client say affordance/picker activatable — no overflow clip) |

Related: presence row layout and affordance corners — `game/presence`.

## MODIFIED Requirements

### Requirement: Own-marker affordance and preset picker

While the user is seated and connected on the Game screen, the system SHALL show a speech-bubble affordance only at the **top-right** of that user’s own presence marker. Activating the affordance MUST reveal exactly two preset choices with the display texts «Всем привет» and «Удачи». Choosing a preset MUST close the picker immediately and submit that preset’s identifier. Spectators MUST NOT see a send affordance. Other players’ markers MUST NOT show a send affordance for the local user. The affordance and its open picker MUST remain pointer- and touch-activatable: presence row overflow MUST NOT clip them out of hit-testing or hide the open picker, and the affordance hit target MUST be large enough for touch (at least about 32 CSS pixels on each side).

#### Scenario [SC-SAY-07]: Seated player opens picker on own marker

- **GIVEN** the user is seated and connected on the Game screen
- **WHEN** the user activates the speech-bubble affordance at the top-right of their own presence marker
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

### Requirement: Bubble stack orientation by presence slot

Speech bubbles for a seat MUST stack against that seat’s presence marker on the side toward the board: for markers in the top presence row, bubbles appear below the avatar; for the seated viewer’s own marker in the bottom row, bubbles appear above the avatar. Newer bubbles MUST appear closer to the avatar than older ones. Horizontal spacing between neighboring presence markers MUST keep concurrent bubbles of adjacent seats from overlapping. Bubbles MUST NOT use viewport toast notifications as their primary presentation. Left/right side-slot stack rules MUST NOT apply (those slots are removed by `game/presence`).

#### Scenario [SC-SAY-11]: Side slots stack newer below

- **GIVEN** a sender whose presence marker is in the top row for the viewing client and that sender already has one live bubble
- **WHEN** a second say from that sender is accepted
- **THEN** both bubbles appear below that avatar toward the board
- **AND** the newer bubble is closer to the avatar than the older bubble
- **AND** no left/right side-column presence slot is used for bubble orientation

#### Scenario [SC-SAY-12]: Home and opposite slots keep newer closer to avatar

- **GIVEN** a seated viewer whose own presence marker is in the bottom row and multiple live bubbles exist for that seat
- **WHEN** any client views that marker
- **THEN** the bubbles appear above the avatar toward the board
- **AND** the newest bubble is visually closer to the avatar than older bubbles
- **AND** bubbles are attached beside that presence marker rather than as screen-edge toasts
