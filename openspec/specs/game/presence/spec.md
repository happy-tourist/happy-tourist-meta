# game/presence Specification

## Purpose

На экране Game показывает занятые места за столом кружками туристов: раскладка относительно игрока или зрителя, офлайн-состояние с круговым countdown на время reconnect grace, значок места финиша для finished-seat (`game/finish`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-01 | covered (client Game presence chrome) |
| SC-PRESENCE-02 | covered (client seated relative layout) |
| SC-PRESENCE-03 | covered (client spectator layout) |
| SC-PRESENCE-04 | covered (client offline circular countdown) |
| SC-PRESENCE-05 | covered (client only occupied seats) |
| SC-PRESENCE-06 | covered (client UX) |
| SC-PRESENCE-07 | covered (client UX) |

## Requirements

### Requirement: Occupied seat presence on Game

While the user is on the Game screen in a tourist room, the system SHALL show a presence marker for every currently seated player (including seats held offline during reconnect grace). Empty seat slots MUST NOT be shown. Spectators MUST see the same set of occupied markers. Presence markers MUST use each seat’s tourist kind image.

#### Scenario [SC-PRESENCE-01]: Seated and spectators see occupied markers

- **GIVEN** a tourist room with two or more seated players
- **WHEN** a seated client and a spectator client view the Game screen
- **THEN** both see one presence marker per occupied seat
- **AND** each marker uses that seat’s tourist kind image

#### Scenario [SC-PRESENCE-05]: No empty seat placeholders

- **GIVEN** a tourist room with fewer than four seated players
- **WHEN** any client views the Game screen
- **THEN** only occupied seats have presence markers
- **AND** vacant places are not rendered as empty markers

### Requirement: Relative layout for seated players

For a seated viewer, that viewer’s own marker MUST appear at the home/north position (bottom of the presence layout). Other seated players MUST be placed relative to join order among opponents: the first other seated player opposite (top), the second to the left, the third to the right.

#### Scenario [SC-PRESENCE-02]: Seated viewer is always at home position

- **GIVEN** the user is seated and three other seated players exist, ordered by earlier join time among those others
- **WHEN** the Game presence layout is shown
- **THEN** the user’s marker is at the home/north (bottom) position
- **AND** the earliest other seated player is at the top
- **AND** the next other is at the left
- **AND** the last other is at the right

### Requirement: Spectator presence layout

For a spectator viewer, occupied seats MUST be laid out by join order among seated players: first seated at the top, second at the bottom, third at the left, fourth at the right. Missing seats simply omit those positions.

#### Scenario [SC-PRESENCE-03]: Spectator order top, bottom, left, right

- **GIVEN** a tourist room with four seated players in known join order and the user is a spectator
- **WHEN** the Game presence layout is shown
- **THEN** the first seated player’s marker is at the top
- **AND** the second is at the bottom
- **AND** the third is at the left
- **AND** the fourth is at the right

### Requirement: Offline reconnect countdown on presence

While a seat is offline within reconnect grace, its presence marker MUST show a determinate circular progress reflecting remaining grace time (full at disconnect, empty at deadline) around the tourist image. While the seat is connected, the marker MUST NOT show that reconnect countdown ring.

#### Scenario [SC-PRESENCE-04]: Offline seat shows circular countdown

- **GIVEN** a seated player’s seat is marked offline with a reconnect deadline in synced state
- **WHEN** any client views that seat’s presence marker on Game
- **THEN** a circular countdown progress is shown for the remaining grace time
- **AND** the tourist kind image remains visible in the marker
- **AND** when the seat becomes connected again, the countdown ring is not shown

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
