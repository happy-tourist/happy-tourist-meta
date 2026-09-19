# game/presence Specification

## Purpose

На экране Game показывает занятые места за столом кружками туристов: для seated — свой маркер в sticky нижней HUD-панели вместе с budgets и личной полосой, оппоненты в одном ряду сверху вплотную к доске; для spectator — все маркеры сверху без нижней presence-панели. Офлайн — круговой reconnect countdown; текущий ход в `playing` — countdown хода (синий / соло красный), при необходимости оба кольца сразу. Match status и leave — в общей шапке приложения. Значок места финиша, ready и say — угловые affordances на маркере (`game/finish`, `game/start`, `game/say`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-01 | covered (client Game presence chrome) |
| SC-PRESENCE-02 | covered (client seated — opponents above board, own in bottom HUD) |
| SC-PRESENCE-03 | covered (client spectator — all markers above board) |
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
| SC-PRESENCE-15 | covered (client UX — budgets vertical stack right of avatar) |
| SC-PRESENCE-16 | covered (client UX — solo ∞ peeks only) |
| SC-PRESENCE-17 | covered (client UX — end-turn icon right-center of own avatar) |
| SC-PRESENCE-18 | covered (client UX) |
| SC-PRESENCE-19 | covered (client UX — peeks-unlimited modal) |
| SC-PRESENCE-20 | covered (client UX — +N anim ~2s) |
| SC-PRESENCE-21 | covered (client UX — timer vs steps-loss copy) |
| SC-PRESENCE-22 | covered (client sticky bottom HUD — own+strip when seated) |
| SC-PRESENCE-23 | covered (client match status in shared header) |
| SC-PRESENCE-24 | covered (client no room id in chrome) |
| SC-PRESENCE-25 | covered (client UX — not in budgets stack; not above-panel dock label) |

Related: turn deadlines / budgets / end-turn / solo — `game/move`; peek modal — `game/board`; reconnect grace — `game/pieces`; bubbles — `game/say`; exit chrome — `game/leave`; board width — `game/board`.

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

For a seated viewer, presence markers of **other** occupied seats MUST appear in a row **above** the board, ordered left-to-right by earlier join time among those opponents. The viewer’s **own** presence marker MUST appear in the sticky bottom HUD panel under the board, together with that viewer’s private step/peek counters and the personal tourist strip when it exists (`game/pieces`). Opponent markers MUST NOT appear in the bottom HUD. Own marker MUST NOT appear above the board. Left/right columns beside the board MUST NOT be used for presence.

#### Scenario [SC-PRESENCE-02]: Seated viewer is always at home position

- **GIVEN** the user is seated and three other seated players exist, ordered by earlier join time among those others
- **WHEN** the Game presence layout is shown
- **THEN** the user’s marker is in the bottom HUD panel
- **AND** the three opponents appear above the board left-to-right in join order among those others
- **AND** no opponent marker is laid out in the bottom HUD
- **AND** no presence marker is laid out in a left or right column beside the board

### Requirement: Spectator presence layout

For a spectator viewer, all occupied seats MUST be laid out in a single row **above** the board, ordered left-to-right by join order among seated players. Missing seats simply omit markers. Spectators MUST NOT see an own-marker cluster, personal strip, or bottom presence row. Left/right columns beside the board MUST NOT be used for presence.

#### Scenario [SC-PRESENCE-03]: Spectator order top, bottom, left, right

- **GIVEN** a tourist room with four seated players in known join order and the user is a spectator
- **WHEN** the Game presence layout is shown
- **THEN** all four seated players’ markers appear above the board left-to-right in that join order
- **AND** no presence markers appear in a bottom HUD row for that spectator
- **AND** no own-marker cluster or personal tourist strip is shown for the spectator

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

### Requirement: Own step and peek counters beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s private **steps** and **peeks** counters in a **vertical stack to the right of** that user’s own presence avatar in the bottom HUD. When peeks are infinite (solo mode), the peeks counter MUST display an infinity indication; the steps counter MUST show the finite numeric value. Other seated players’ and spectators’ clients MUST NOT show another seat’s step or peek counters. Spectators MUST NOT see step/peek counters for any seat. When the user’s finite budgets increase, the client SHOULD play a local “+N falls into the counter” animation for steps and peeks grants (turn grant and successful peek rewards) lasting approximately **two seconds**. The end-turn control MUST NOT sit in this budgets stack (`game/presence` end-turn requirement).

#### Scenario [SC-PRESENCE-15]: Seated user sees only own counters

- **GIVEN** two seated players in phase `playing` with different private budgets
- **WHEN** each views Game presence
- **THEN** each sees steps and peeks only on their own marker, stacked vertically to the right of their own avatar
- **AND** neither sees the other’s budget numbers on the opponent marker

#### Scenario [SC-PRESENCE-16]: Solo shows infinity only on peeks

- **GIVEN** the user is the sole non-finished seated player under infinite peeks and finite steps
- **WHEN** the user views their presence marker
- **THEN** the peeks counter shows infinity
- **AND** the steps counter shows the numeric steps value
- **AND** other clients still do not see those budget values

#### Scenario [SC-PRESENCE-20]: Budget grant animation lasts about two seconds

- **GIVEN** the user’s finite steps or peeks budget increases while they view their own presence marker
- **WHEN** the local +N fall animation plays
- **THEN** the animation is visibly slower than a sub-second flash and completes in about two seconds
- **AND** other clients do not see that animation

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show an **icon-only** end-turn control (Material `skip_next` or equivalent) on the **right** edge of that user’s own presence avatar, **vertically centered** on that edge (mirror of a say/dialog affordance centered on the **top** edge). Activating it MUST submit end-turn per `game/move` **immediately** — no confirmation dialog. The control MUST NOT show a visible text label (tooltips later are out of scope). The control MUST NOT appear in the budgets stack beside the avatar. The control MUST NOT appear as a labeled dock above the sticky bottom HUD. The control MUST NOT appear for spectators, for seats that are not current turn, during solo play, or for finished / time-expired seats.

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views Game chrome
- **THEN** an icon-only end-turn control is shown on the right edge of the own avatar, vertically centered
- **AND** activating it submits end-turn without a confirmation dialog
- **AND** no visible «Завершить ход» label is required on the control

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is the sole eligible seated player in solo play
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown

#### Scenario [SC-PRESENCE-25]: End-turn not inline with budgets

- **GIVEN** it is the local seated user’s turn in multi-seat play and end-turn is available
- **WHEN** the bottom HUD budgets stack beside the own avatar is shown
- **THEN** steps and peeks appear in that stack without the end-turn control inline
- **AND** the end-turn control remains on the right edge of the avatar (not a separate above-panel text dock)

### Requirement: Solo peeks-unlimited modal

When the user’s seat enters solo infinite-peeks mode (`game/move`), that user’s client MUST show a modal whose product Russian sense informs them that they are alone, that peeks are unlimited, and that steps remain limited. Closing the modal MUST leave the user in the room. Other clients MUST NOT show that modal.

#### Scenario [SC-PRESENCE-19]: Only the solo player sees the peeks-unlimited modal

- **GIVEN** a seated player becomes the sole non-finished seat and at least one spectator or finished seated client is present
- **WHEN** solo infinite peeks apply
- **THEN** that player’s client shows the peeks-unlimited modal
- **AND** other clients do not show that modal

### Requirement: Distinct solo end copy for timer vs steps exhaustion

When the user’s seat becomes time-expired in solo (`game/move`), the client MUST show an end modal whose product Russian copy depends on the cause: timer expiry versus steps exhaustion with no live task tile under any unfinished piece. Both causes share the same locked interaction (no moves/peeks); the visible text MUST differ so the player understands whether time ran out or steps ran out.

#### Scenario [SC-PRESENCE-21]: Steps-loss and timer-loss show different copy

- **GIVEN** the user is the sole non-finished seated player
- **WHEN** the seat becomes time-expired because steps reached 0 with no unfinished piece on a still-present task cell
- **THEN** the client shows the steps-exhausted end copy
- **AND** when instead the five-minute deadline elapses with steps still available, the client shows the timer-expired end copy
- **AND** neither copy is shown to other clients as that user’s private end modal

### Requirement: Sticky bottom game HUD panel

While the user is on the Game screen as a **seated** viewer, the bottom HUD panel that holds the own presence marker, budgets beside the avatar, and personal strip (when present) MUST remain pinned to the bottom of the viewport so it stays visible while the board area above may scroll or resize. Opponent markers MUST use the row above the board, not this sticky panel — laid out tightly against the board (no large reserved empty band under the markers for say chrome). For a **spectator**, there MUST NOT be a bottom presence HUD row; markers stay above the board.

#### Scenario [SC-PRESENCE-22]: Bottom HUD stays pinned while board scrolls

- **GIVEN** the seated user is on the Game screen with the bottom HUD panel visible
- **WHEN** the board content area above the panel is scrolled or the board height changes
- **THEN** the bottom HUD panel remains pinned to the bottom of the viewport
- **AND** the user’s own marker and strip remain inside that panel
- **AND** opponent markers remain above the board

### Requirement: Match status in shared application header

While the user is on the Game screen, the shared application header MUST show a concise match-status label centered in the header (product Russian senses such as «Ваш ход», «Ход соперника», «Ход игрока», «Ожидание соперника», connection/finished phrases as already used on Game). On Login and Lobby screens that match-status label MUST NOT appear. The status MUST derive from room phase / turn / connection state already mirrored for Game — not from a separate server channel.

#### Scenario [SC-PRESENCE-23]: Status centered in shared header on Game

- **GIVEN** the user is on the Game screen of a tourist room in phase `playing` and it is the local seated user’s turn
- **WHEN** the shared application header is shown
- **THEN** a centered status label conveys that it is the user’s turn
- **AND** the theme control remains available in that header
- **AND** on Lobby the same match-status label is not shown

### Requirement: No room identifier in Game chrome

The Game screen and shared application header MUST NOT display the tourist room identifier (full or truncated) as visible chrome. Routing and reconnect MAY continue to use the room id without showing it to the user.

#### Scenario [SC-PRESENCE-24]: Room id not shown on Game

- **GIVEN** the user is on the Game screen of a tourist room
- **WHEN** the Game chrome and shared application header are shown
- **THEN** no visible room identifier (full or truncated) is presented as page chrome
