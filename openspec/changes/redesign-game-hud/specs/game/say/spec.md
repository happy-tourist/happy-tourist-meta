# game/say Delta

Related: presence bottom HUD — `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-11 | pending (client bubbles above all markers toward board) |
| SC-SAY-12 | pending (client newer closer above avatar) |

## MODIFIED Requirements

### Requirement: Bubble stack orientation by presence slot

Speech bubbles for a seat MUST stack against that seat’s presence marker **above** the avatar (toward the board), because all presence markers live in the bottom HUD panel (`game/presence`). Newer bubbles MUST appear closer to the avatar than older ones. Horizontal spacing between neighboring presence markers MUST keep concurrent bubbles of adjacent seats from overlapping. Bubbles MUST NOT use viewport toast notifications as their primary presentation. Bubbles MUST NOT stack below the avatar. Left/right side-slot stack rules MUST NOT apply.

#### Scenario [SC-SAY-11]: Side slots stack newer below

- **GIVEN** a sender whose presence marker is visible in the bottom HUD panel for the viewing client and that sender already has one live bubble
- **WHEN** a second say from that sender is accepted
- **THEN** both bubbles appear above that avatar toward the board
- **AND** the newer bubble is closer to the avatar than the older bubble
- **AND** no bubble stack appears below the avatar
- **AND** no left/right side-column presence slot is used for bubble orientation

#### Scenario [SC-SAY-12]: Home and opposite slots keep newer closer to avatar

- **GIVEN** any occupied presence marker in the bottom HUD panel with multiple live bubbles for that seat
- **WHEN** any client views that marker
- **THEN** the bubbles appear above the avatar toward the board
- **AND** the newest bubble is visually closer to the avatar than older bubbles
- **AND** bubbles are attached beside that presence marker rather than as screen-edge toasts
