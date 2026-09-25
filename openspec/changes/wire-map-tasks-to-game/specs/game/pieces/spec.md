## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PIECE-01 | modified (variable pcs, no sides) |
| SC-PIECE-02 | modified (kinds unique; pcs count) |
| SC-PIECE-03 | modified (map starts, distance) |
| SC-PIECE-04 | modified |
| SC-PIECE-09 | modified (N-slot strip) |
| SC-PIECE-18 | modified |
| SC-PIECE-25 | modified (all-jail without sides) |
| SC-PIECE-50 | pending (server mocha — distance spawn) |
| SC-PIECE-51 | pending (server mocha — touristsPerPlayer count) |
| SC-PIECE-52 | pending (client UX — strip slot count) |

Related: map capacity — `lobby/rooms`; layout — `game/board`; strip HUD — `game/presence`.

## MODIFIED Requirements

### Requirement: Four pieces per seated player

While the start phase is `waiting` and fewer seated players exist than maxSeats, when an authenticated client joins, the server SHALL assign that client a unique tourist kind from `1`…`4` not used by any current seat and MUST NOT place pieces yet; pieces for those seats are created when playing begins per the materialize requirement. The number of pieces per seated player MUST equal the room map snapshot’s **touristsPerPlayer** (integer 1…4). All pieces of the same player MUST use that player’s tourist kind. Pieces MUST NOT use board side identities `N`/`E`/`S`/`W`. The assignment MUST be synchronized to all clients in the room. New seats MUST NOT be assigned while the phase is `countdown` or `playing` (see spectators requirement).

#### Scenario [SC-PIECE-01]: First join receives four pieces on all sides

- **GIVEN** a tourist room in phase `waiting` that has no seated players, free seat capacity, and map touristsPerPlayer=2
- **WHEN** an authenticated client joins the room
- **THEN** the client is assigned exactly one tourist kind from `1`…`4`
- **AND** the client has a seat with no pieces yet in synced state
- **AND** when the phase later becomes `playing`, that seat has exactly two pieces on distinct free start cells of the map snapshot
- **AND** every client in the room observes those pieces after materialize
- **AND** those pieces MUST NOT carry side identities N/E/S/W

#### Scenario [SC-PIECE-02]: Tourist kinds stay unique among players

- **GIVEN** a tourist room in phase `waiting` that already has one or more seated players and still has free seat capacity
- **WHEN** another authenticated client joins and receives a seat
- **THEN** the new tourist kind is not equal to any currently seated player’s kind
- **AND** when pieces exist for that seat, all of the new player’s pieces use that same new kind

#### Scenario [SC-PIECE-03]: Start cells lie on the assigned side and stay free

- **GIVEN** a tourist room that is materializing pieces at playing start for seated players
- **WHEN** seats are assigned pieces on map start cells
- **THEN** each piece’s row and column match a start cell of the map snapshot
- **AND** no two pieces in the room share the same row and column

#### Scenario [SC-PIECE-04]: Second player uses remaining cells on each side

- **GIVEN** a tourist room that entered `playing` with two seated players who received seats during `waiting`
- **WHEN** pieces are materialized for both seats
- **THEN** none of the second player’s cells equal any cell occupied by the first player

### Requirement: Materialize pieces when playing begins

When the start phase transitions from `countdown` to `playing`, the server SHALL place exactly `touristsPerPlayer` pieces for every seated player that does not yet have pieces, choosing free **start** cells from the map snapshot. Placement MUST maximize distance among that seat’s own pieces as far as free start cells allow (greedy nearest-first among remaining starts is acceptable). Opponent pieces MAY occupy any remaining free starts; own-piece separation is the primary rule. No two pieces in the room MAY share the same cell. Every client MUST observe those pieces in synced state as phase becomes `playing`.

#### Scenario [SC-PIECE-18]: Waiting seats receive pieces at playing

- **GIVEN** a tourist room that completed countdown with two or more seated players who had tourist kinds but no pieces during `waiting`/`countdown`
- **WHEN** the synchronized phase becomes `playing`
- **THEN** each of those seats has exactly touristsPerPlayer pieces on free map start cells
- **AND** no two pieces in the room share the same cell
- **AND** every client observes those pieces

### Requirement: Board pieces and personal four-slot strip

All clients in the room SHALL see all current unfinished pieces on the Game board at their synced cells, using the tourist image for each piece’s kind. While a seated player has no pieces yet (phase `waiting` or `countdown`), no pieces for that seat MUST appear on the board. Finished pieces MUST NOT be rendered on the board (see `game/finish`). A seated client SHALL see a personal strip of exactly `touristsPerPlayer` slots once their pieces exist, corresponding one-to-one to that client’s pieces (same kind image). That strip MUST appear inside the bottom Game HUD panel (`game/presence`) beside the seated user’s own presence/budgets cluster and MUST NOT use a separate caption label such as «Мои туристы».

The strip MUST present interactive slots (not a compact status-only chip and not a picker menu). Each slot MUST reflect status: finish indicator when finished (`game/finish`); grille presentation when the piece is trapped. Until pieces exist, the seated client MUST NOT be shown the strip. The strip MUST remain visible for finished seats. The chrome MUST NOT show other players’ tourists. A spectator MUST NOT be shown that personal strip.

Activating an unfinished non-trapped slot MUST select that piece for move UX while applicable (`game/move`). Finished and trapped pieces MUST NOT be used for move selection (finished return flow is `game/finish`). Board piece click MUST still select without opening any menu. The Game UI MUST NOT use a picker menu opened from a compact chip.

On narrow viewports the strip MAY wrap or shrink slots so the own avatar cluster plus strip fit without relying on horizontal scrolling as the primary UX.

#### Scenario [SC-PIECE-09]: Seated client sees own four-slot strip

- **GIVEN** the user is on the Game screen as a seated player with touristsPerPlayer=3 pieces of one kind
- **WHEN** the game UI is shown
- **THEN** the bottom HUD panel shows three personal tourist slots after the user’s own presence/budgets cluster
- **AND** no compact chip-only chrome and no tourist picker menu are required to select a piece
- **AND** no separate «Мои туристы» caption labels that strip
- **AND** the strip does not display other players’ tourist kinds

#### Scenario [SC-PIECE-10]: Spectator sees pieces but no strip

- **GIVEN** the user is on the Game screen as a spectator while at least one seat exists
- **WHEN** the Game presence layout is shown
- **THEN** seated players’ unfinished pieces are visible on the board at their cells
- **AND** no personal tourist strip is shown for that user

#### Scenario [SC-PIECE-17]: No board pieces before playing

- **GIVEN** a tourist room in phase `waiting` or `countdown` with one or more seated players who have tourist kinds
- **WHEN** any client views the Game board
- **THEN** no tourist pieces for those seats are shown on the board
- **AND** when phase becomes `playing` and pieces are materialized, those pieces appear at their synced cells

#### Scenario [SC-PIECE-32]: Narrow width uses 2×2 strip

- **GIVEN** the seated user views Game at about 320 CSS pixels content width (or narrower about 300) with touristsPerPlayer=4
- **WHEN** the bottom HUD with own avatar and personal strip is shown
- **THEN** the tourist slots are laid out so the own avatar cluster plus strip fit without relying on horizontal scrolling as the primary UX
- **AND** when four slots are present they MAY use a 2×2 grid with slots smaller than the own avatar image

### Requirement: All-jail places one piece per side on free starts

On all-jail reset for a seat, each of that seat’s pieces MUST be placed on a free start cell of the map snapshot, choosing free starts to maximize distance among that seat’s own pieces as at materialize. Pieces MUST be free (not trapped) after placement. Side identities MUST NOT be used.

#### Scenario [SC-PIECE-25]: Reset uses each side’s free start cells

- **GIVEN** a seated player triggers all-jail reset with touristsPerPlayer=2
- **WHEN** the server places the pieces
- **THEN** both pieces occupy distinct free map start cells
- **AND** none share a cell with another unfinished piece
- **AND** none remain trapped

## ADDED Requirements

### Requirement: Own pieces maximize mutual distance at materialize

When materializing multiple pieces for one seat, the server MUST prefer free start-cell pairs/sets that maximize distance among that seat’s own pieces.

#### Scenario [SC-PIECE-50]: Own pieces maximize mutual distance

- **GIVEN** a map snapshot with at least four free start cells and touristsPerPlayer=2 for one seated player at materialize
- **WHEN** the server places that seat’s two pieces
- **THEN** the two cells are a pair of free starts that maximizes distance between them among available free starts at placement time (ties MAY break arbitrarily)

#### Scenario [SC-PIECE-51]: Piece count follows touristsPerPlayer

- **GIVEN** a room whose map snapshot has touristsPerPlayer=3 and two seated players who waited without pieces
- **WHEN** phase becomes `playing`
- **THEN** each of those seats has exactly three pieces
- **AND** no piece carries a side identity N/E/S/W

#### Scenario [SC-PIECE-52]: Strip slot count follows touristsPerPlayer

- **GIVEN** touristsPerPlayer=1 for the local seated user with pieces materialized
- **WHEN** the bottom HUD strip is shown
- **THEN** exactly one personal tourist slot is shown
