# game/presence Delta

Related: bubbles — `game/say`; strip — `game/pieces`; exit — `game/leave`; board — `game/board`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-02 | pending (client Game bottom HUD seated layout) |
| SC-PRESENCE-03 | pending (client Game bottom HUD spectator center) |
| SC-PRESENCE-22 | pending (client sticky bottom HUD panel) |
| SC-PRESENCE-23 | pending (client match status in shared header) |
| SC-PRESENCE-24 | pending (client no room id on Game chrome) |

## MODIFIED Requirements

### Requirement: Relative layout for seated players

For a seated viewer, the Game screen MUST show a single bottom HUD panel under the board that contains that viewer’s own presence marker, that viewer’s private step/peek counters and end-turn control when applicable (`game/move`), the personal tourist strip when it exists (`game/pieces`), and the presence markers of all other occupied seats. Within that panel the layout MUST be left-to-right: own marker (with own budgets/end-turn beside it as today) → personal tourist strip (when shown) → opponents grouped toward the right edge of the panel, ordered left-to-right by earlier join time among those opponents. Presence markers MUST NOT appear above the board or in left/right columns beside the board.

#### Scenario [SC-PRESENCE-02]: Seated viewer is always at home position

- **GIVEN** the user is seated and three other seated players exist, ordered by earlier join time among those others
- **WHEN** the Game presence layout is shown
- **THEN** the user’s marker is in the bottom HUD panel on the left side of that panel
- **AND** the three opponents appear in the same bottom HUD panel toward the right edge left-to-right in join order among those others
- **AND** no presence marker is laid out above the board or in a left or right column beside the board

### Requirement: Spectator presence layout

For a spectator viewer, all occupied seats MUST be laid out in the single bottom HUD panel under the board, centered as a group within that panel, ordered left-to-right by join order among seated players. Missing seats simply omit markers. Spectators MUST NOT see an own-marker cluster or personal strip. Presence markers MUST NOT appear above the board or in left/right columns beside the board.

#### Scenario [SC-PRESENCE-03]: Spectator order top, bottom, left, right

- **GIVEN** a tourist room with four seated players in known join order and the user is a spectator
- **WHEN** the Game presence layout is shown
- **THEN** all four seated players’ markers appear centered in the bottom HUD panel left-to-right in that join order
- **AND** no presence marker is shown above the board or in a left or right column beside the board
- **AND** no own-marker cluster or personal tourist strip is shown for the spectator

## ADDED Requirements

### Requirement: Sticky bottom game HUD panel

While the user is on the Game screen, the bottom HUD panel that holds presence (and for seated users the personal strip when present) MUST remain pinned to the bottom of the viewport so it stays visible while the board area above may scroll or resize. Markers and strip MUST NOT use a separate floating row above the board.

#### Scenario [SC-PRESENCE-22]: Bottom HUD stays pinned while board scrolls

- **GIVEN** the user is on the Game screen with the bottom HUD panel visible
- **WHEN** the board content area above the panel is scrolled or the board height changes
- **THEN** the bottom HUD panel remains pinned to the bottom of the viewport
- **AND** presence markers remain inside that panel (not above the board)

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
