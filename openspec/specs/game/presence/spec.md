# game/presence Specification

## Purpose

На экране Game показывает занятые места за столом кружками туристов: для seated — свой маркер внизу под доской, оппоненты в одном ряду сверху; для spectator — все маркеры сверху. Офлайн — круговой reconnect countdown; текущий ход в `playing` — countdown хода (синий / соло красный), при необходимости оба кольца сразу. Значок места финиша, ready и say — угловые affordances на маркере (`game/finish`, `game/start`, `game/say`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-01 | covered (client Game presence chrome) |
| SC-PRESENCE-02 | covered (client GamePage row layout — self bottom, opponents top) |
| SC-PRESENCE-03 | covered (client GamePage spectator all-top) |
| SC-PRESENCE-04 | covered-by-reuse (client Game presence chrome — dual with turn) |
| SC-PRESENCE-05 | covered (client only occupied seats) |
| SC-PRESENCE-06 | covered (client UX) |
| SC-PRESENCE-07 | covered (client UX) |
| SC-PRESENCE-08 | covered (client GamePage turn ring) |
| SC-PRESENCE-09 | covered (client GamePage solo red ring) |
| SC-PRESENCE-10 | covered (client GamePage dual sibling rings) |
| SC-PRESENCE-11 | covered (client reserved outer chrome 96px around 72px avatar) |
| SC-PRESENCE-12 | covered (client sibling avatar img + rings) |
| SC-PRESENCE-13 | covered (client avatar size matches strip tourist) |
| SC-PRESENCE-14 | covered (client finish/ready/say corner affordances) |

Related: turn deadlines — `game/move`; reconnect grace — `game/pieces`; bubbles — `game/say`; board width — `game/board`.

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

For a seated viewer, that viewer’s own marker MUST appear in a bottom presence row under the board. All other occupied seated players MUST appear in a single top presence row above the board, ordered left-to-right by earlier join time among those opponents. Presence markers MUST NOT be placed in left or right columns beside the board.

#### Scenario [SC-PRESENCE-02]: Seated viewer is always at home position

- **GIVEN** the user is seated and three other seated players exist, ordered by earlier join time among those others
- **WHEN** the Game presence layout is shown
- **THEN** the user’s marker is alone in the bottom row under the board
- **AND** the three opponents appear in one top row above the board left-to-right in join order among those others
- **AND** no presence marker is laid out in a left or right column beside the board

### Requirement: Spectator presence layout

For a spectator viewer, all occupied seats MUST be laid out in a single top presence row above the board, ordered left-to-right by join order among seated players. Missing seats simply omit markers. Presence markers MUST NOT be placed in left, right, or bottom columns for spectators.

#### Scenario [SC-PRESENCE-03]: Spectator order top, bottom, left, right

- **GIVEN** a tourist room with four seated players in known join order and the user is a spectator
- **WHEN** the Game presence layout is shown
- **THEN** all four seated players’ markers appear in one top row above the board left-to-right in that join order
- **AND** no presence marker is shown in a bottom, left, or right column beside the board

### Requirement: Offline reconnect countdown on presence

While a seat is offline within reconnect grace, its presence marker MUST show a determinate circular progress reflecting remaining grace time (full at disconnect, empty at deadline) around the tourist image as the inner ring when a turn ring is also present, or as the sole active countdown ring when no turn deadline ring is active. While the seat is connected, the marker MUST NOT show that reconnect countdown as an active progress. Layout reservation for ring chrome remains per the reserved-marker requirement even when reconnect progress is inactive.

#### Scenario [SC-PRESENCE-04]: Offline seat shows circular countdown

- **GIVEN** a seated player’s seat is marked offline with a reconnect deadline in synced state
- **WHEN** any client views that seat’s presence marker on Game
- **THEN** a circular countdown progress is shown for the remaining grace time
- **AND** the tourist kind image remains visible in the marker
- **AND** when the seat becomes connected again, the reconnect countdown progress is not shown as active
- **AND** if that seat also holds an active turn deadline, the turn countdown remains visible together with the reconnect countdown

### Requirement: Tourist avatar always visible on occupied presence

Every occupied presence marker on the Game screen SHALL keep the seat’s tourist kind image visible in the marker at all times (connected or offline-in-grace, with or without active turn/reconnect rings). Rings MUST surround that image without replacing or omitting it. Implementations MUST NOT rely on placing the image only inside a Quasar `q-circular-progress` default slot without `show-value` (that slot is not rendered), and MUST NOT nest circular-progress components such that the image never appears in the DOM.

#### Scenario [SC-PRESENCE-12]: Occupied marker always shows tourist image

- **GIVEN** any occupied seated player’s presence marker on Game (with or without active turn deadline and/or reconnect grace countdown)
- **WHEN** any client views that marker
- **THEN** the tourist kind image for that seat is visible in the marker
- **AND** any active turn and/or reconnect countdown rings appear around that image without hiding it

### Requirement: Turn countdown ring on presence

While phase is `playing` and a seated player’s seat holds the synchronized current turn with an active turn deadline, every client that shows that seat’s presence marker (including spectators) SHALL show a determinate circular progress for remaining turn time around the tourist image: full at the start of the deadline budget, empty at the deadline. While two or more eligible seats remain, that ring MUST use the same blue accent previously used for the static current-turn outline. While the solo five-minute budget of `game/move` applies, that ring MUST use a red accent instead. The static blue outline / box-shadow that previously marked the current-turn seat MUST NOT be used. When the seat is not current turn, or has no active turn deadline (including time-expired or finished with cleared deadline), the turn countdown ring MUST NOT show progress as an active turn timer (track chrome may remain for layout stability per the reserved-marker requirement).

#### Scenario [SC-PRESENCE-08]: Current turn shows blue countdown instead of outline

- **GIVEN** a tourist room in phase `playing` with at least two non-finished seated players and a current-turn seat with a 60-second deadline
- **WHEN** any client views that seat’s presence marker
- **THEN** a blue circular countdown for remaining turn time is shown
- **AND** the tourist kind image remains visible in the marker
- **AND** no static blue outline-only turn indicator is shown on that marker

#### Scenario [SC-PRESENCE-09]: Solo budget shows red countdown

- **GIVEN** a tourist room in phase `playing` with exactly one non-finished seated player under the five-minute solo budget
- **WHEN** any client (including a spectator) views that seat’s presence marker
- **THEN** a red circular countdown for the remaining solo budget is shown
- **AND** the tourist kind image remains visible in the marker

### Requirement: Dual turn and reconnect countdowns

When a seat simultaneously has an active turn deadline ring and an offline reconnect grace countdown, the presence marker MUST show **both** determinate circular progresses at once so neither hides the other. The turn countdown MUST be the outer ring and the reconnect grace countdown the inner ring (warning accent as today). Spectators and seated viewers MUST see the same dual display.

#### Scenario [SC-PRESENCE-10]: Offline current-turn seat shows both rings

- **GIVEN** a current-turn seat that is offline within reconnect grace with remaining turn deadline and remaining reconnect grace
- **WHEN** any client views that seat’s presence marker
- **THEN** both the turn countdown ring and the reconnect grace countdown ring are visible together
- **AND** the turn ring is outside the reconnect ring
- **AND** the tourist kind image remains visible in the marker

### Requirement: Reserved presence marker chrome size

Every occupied presence marker on the Game screen SHALL reserve stable space for the outer turn-ring chrome whether or not a turn or reconnect countdown is currently active, so that showing or hiding active countdown progress does not change the marker’s layout size or shift the board or neighboring markers.

#### Scenario [SC-PRESENCE-11]: Marker size stable when countdown appears

- **GIVEN** a connected seated player’s presence marker without an active reconnect countdown
- **WHEN** that seat becomes current turn and the turn countdown becomes active (or later becomes offline with reconnect countdown)
- **THEN** the marker’s reserved layout size does not jump
- **AND** the board layout does not reflow solely because the countdown appeared
- **AND** neighboring presence markers do not shift solely because the countdown appeared

### Requirement: Presence avatar matches strip tourist size

The tourist kind image inside every occupied presence marker SHALL use the same display size as one tourist image in the seated player’s personal strip on Game. Turn and reconnect rings MUST surround that image (outer ring larger than the avatar).

#### Scenario [SC-PRESENCE-13]: Presence avatar matches strip image size

- **GIVEN** the user is seated with a personal tourist strip visible and at least one occupied presence marker on Game
- **WHEN** any client compares the presence avatar image box to one strip tourist image box
- **THEN** those image boxes match in width and height
- **AND** any active countdown rings appear around the presence avatar without shrinking it below that size

### Requirement: Presence affordance and badge corners

When a seat has a synchronized finish place, every client SHALL show the finish place indicator at the **top-left** of that seat’s presence marker. While the local user may send a ready intent from the Game screen, the ready affordance MUST appear at the **top-left** of that user’s own presence marker only. While the local user may send a say intent, the say send affordance MUST appear at the **top-right** of that user’s own presence marker only. Other players’ markers MUST NOT show a say send affordance for the local user. Spectators MUST NOT see a say send affordance.

#### Scenario [SC-PRESENCE-14]: Finish, ready, and say corners

- **GIVEN** a seated connected user viewing Game with their own marker and at least one opponent marker that has finish place `1`
- **WHEN** the presence chrome is shown
- **THEN** the opponent’s finish place indicator is at the top-left of that opponent marker
- **AND** if the ready affordance is available for the local user, it is at the top-left of the local user’s marker
- **AND** the say send affordance is at the top-right of the local user’s marker only
- **AND** opponent markers do not show a say send affordance for the local user

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
