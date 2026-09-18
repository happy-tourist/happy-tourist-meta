# game/say Delta

Related: presence top/bottom — `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-11 | implemented (client bubbles toward board from top markers) |
| SC-SAY-12 | implemented (client bubbles toward board from bottom marker) |

## MODIFIED Requirements

### Requirement: Bubble stack orientation by presence slot

Speech bubbles for a seat MUST stack against that seat’s presence marker **toward the board**:
- markers in the row **above** the board → bubbles stack **below** the avatar (down toward the board);
- the seated user’s marker in the **bottom** HUD → bubbles stack **above** the avatar (up toward the board).

Newer bubbles MUST appear closer to the avatar than older ones. Horizontal spacing between neighboring presence markers MUST keep concurrent bubbles of adjacent seats from overlapping. Bubbles MUST NOT use viewport toast notifications as their primary presentation. Left/right side-slot stack rules MUST NOT apply.

#### Scenario [SC-SAY-11]: Side slots stack newer below

- **GIVEN** a sender whose presence marker is visible above the board for the viewing client and that sender already has one live bubble
- **WHEN** a second say from that sender is accepted
- **THEN** both bubbles appear below that avatar toward the board
- **AND** the newer bubble is closer to the avatar than the older bubble
- **AND** no bubble stack appears above that top-row avatar away from the board

#### Scenario [SC-SAY-12]: Home and opposite slots keep newer closer to avatar

- **GIVEN** the seated user’s own presence marker in the bottom HUD with multiple live bubbles
- **WHEN** any client views that marker
- **THEN** the bubbles appear above the avatar toward the board
- **AND** the newest bubble is visually closer to the avatar than older bubbles
- **AND** bubbles are attached beside that presence marker rather than as screen-edge toasts
