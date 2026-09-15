## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-06 | covered (client UX) |
| SC-PRESENCE-07 | covered (client UX) |

Related: finish place assignment — `game/finish`.

## ADDED Requirements

### Requirement: Finish place badge on presence marker

When a seated player has a synchronized finish place, every client that shows that seat’s presence marker on the Game screen SHALL display a finish indicator on that marker that includes the place number (1, 2, …). Seats without a finish place MUST NOT show that place badge. The badge MUST remain while the finished seat remains occupied (including offline within reconnect grace).

#### Scenario [SC-PRESENCE-06]: Finished seat shows place on presence

- **GIVEN** a seated player has finish place `2` and is connected
- **WHEN** any client views that seat’s presence marker on Game
- **THEN** the marker shows a finish indicator that includes the place `2`
- **AND** the tourist kind image remains visible in the marker

#### Scenario [SC-PRESENCE-07]: Non-finished seat has no place badge

- **GIVEN** a seated player who has unfinished pieces and no finish place
- **WHEN** any client views that seat’s presence marker
- **THEN** no finish-place badge is shown on that marker
