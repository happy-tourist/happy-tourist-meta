## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-04 | covered-by-reuse (client Game presence chrome — dual with turn) |
| SC-PRESENCE-08 | covered (client GamePage turn ring) |
| SC-PRESENCE-09 | covered (client GamePage solo red ring) |
| SC-PRESENCE-10 | covered (client GamePage dual sibling rings) |
| SC-PRESENCE-11 | covered (client reserved 52px outer chrome) |
| SC-PRESENCE-12 | covered (client sibling avatar img + rings) |

Related: turn deadlines — `game/move`; reconnect grace — `game/pieces`.

## ADDED Requirements

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

Every occupied presence marker on the Game screen SHALL reserve stable space for the outer turn-ring chrome whether or not a turn or reconnect countdown is currently active, so that showing or hiding active countdown progress does not change the marker’s layout size or shift neighboring UI.

#### Scenario [SC-PRESENCE-11]: Marker size stable when countdown appears

- **GIVEN** a connected seated player’s presence marker without an active reconnect countdown
- **WHEN** that seat becomes current turn and the turn countdown becomes active (or later becomes offline with reconnect countdown)
- **THEN** the marker’s reserved layout size does not jump
- **AND** surrounding presence layout does not reflow solely because the countdown appeared

## MODIFIED Requirements

### Requirement: Offline reconnect countdown on presence

While a seat is offline within reconnect grace, its presence marker MUST show a determinate circular progress reflecting remaining grace time (full at disconnect, empty at deadline) around the tourist image as the inner ring when a turn ring is also present, or as the sole active countdown ring when no turn deadline ring is active. While the seat is connected, the marker MUST NOT show that reconnect countdown as an active progress. Layout reservation for ring chrome remains per the reserved-marker requirement even when reconnect progress is inactive.

#### Scenario [SC-PRESENCE-04]: Offline seat shows circular countdown

- **GIVEN** a seated player’s seat is marked offline with a reconnect deadline in synced state
- **WHEN** any client views that seat’s presence marker on Game
- **THEN** a circular countdown progress is shown for the remaining grace time
- **AND** the tourist kind image remains visible in the marker
- **AND** when the seat becomes connected again, the reconnect countdown progress is not shown as active
- **AND** if that seat also holds an active turn deadline, the turn countdown remains visible together with the reconnect countdown
