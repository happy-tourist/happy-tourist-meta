# game/presence Delta

Related: bubbles — `game/say`; strip — `game/pieces`; exit — `game/leave`; budgets/end-turn — `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-02 | implemented (client seated: opponents above board, own in bottom HUD) |
| SC-PRESENCE-03 | implemented (client spectator: all markers above board) |
| SC-PRESENCE-15 | implemented (client budgets beside own avatar) |
| SC-PRESENCE-16 | covered-by-reuse (solo infinity peeks — layout only moves) |
| SC-PRESENCE-17 | implemented (client end-turn above panel right) |
| SC-PRESENCE-18 | covered-by-reuse (solo hides end-turn) |
| SC-PRESENCE-20 | covered-by-reuse (+N animation unchanged) |
| SC-PRESENCE-22 | implemented (client sticky bottom HUD — own+strip when seated) |
| SC-PRESENCE-23 | covered-by-reuse (match status in shared header — phase 1) |
| SC-PRESENCE-24 | covered-by-reuse (no room id — phase 1) |
| SC-PRESENCE-25 | implemented (client end-turn not in budgets row) |

## MODIFIED Requirements

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

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show a control whose visible label is exactly **«Завершить ход»** **above** the sticky bottom HUD panel, aligned toward the **right** edge of the board/HUD content width (lower-right of the board region). Activating it MUST submit end-turn per `game/move`. The control MUST NOT appear in the budgets stack beside the avatar. The control MUST NOT appear for spectators, for seats that are not current turn, during solo play, or for finished / time-expired seats.

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views Game chrome
- **THEN** a control labeled «Завершить ход» is shown above the bottom HUD toward the right
- **AND** activating it submits end-turn

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is the sole eligible seated player in solo play
- **WHEN** the user views Game chrome
- **THEN** the «Завершить ход» control is not shown

#### Scenario [SC-PRESENCE-25]: End-turn not inline with budgets

- **GIVEN** it is the local seated user’s turn in multi-seat play and end-turn is available
- **WHEN** the bottom HUD budgets stack beside the own avatar is shown
- **THEN** steps and peeks appear in that stack without the end-turn control inline
- **AND** the end-turn control remains above the bottom panel toward the right

## ADDED Requirements

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
