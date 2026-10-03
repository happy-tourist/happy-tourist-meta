# lobby/rooms Specification

## Purpose

Live-список доступных игровых комнат на экране лобби: авторизованный пользователь (включая гостя) видит актуальные комнаты `tourist` в реальном времени и может создать или присоединиться к партии.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-01 | partial (client subscribe on mount; full check → task 3.2 manual) |
| SC-LOBBY-02 | covered (server mocha, task 1.5) |
| SC-LOBBY-03 | covered (server mocha, task 1.5) |
| SC-LOBBY-04 | covered (registration + tests/loadtest `tourist`, tasks 1.2/1.4) |
| SC-LOBBY-05 | partial (client `_enterRoom` unsubscribe; full check → task 3.2) |
| SC-LOBBY-06 | pending (manual, task 3.2) |
| SC-LOBBY-07 | covered (client — initial subscribe fail still surfaces) |
| SC-LOBBY-08 | covered (client — lobby drop / reservation noise quiet) |
| SC-LOBBY-09 | removed (superseded by map ceiling + seats picker) |
| SC-LOBBY-10 | removed (superseded by map ceiling + seats picker) |
| SC-LOBBY-11 | covered (client UX) |
| SC-LOBBY-12 | covered (client UX) |
| SC-LOBBY-13 | covered (client UX + server parse) |
| SC-LOBBY-14 | covered (server mocha — many → 35%) |
| SC-LOBBY-15 | covered (client UX default medium) |
| SC-LOBBY-16 | covered (client UX + server parse) |
| SC-LOBBY-17 | covered (server mocha create options — many → 35% catapults) |
| SC-LOBBY-18 | covered (client UX default medium for catapults) |
| SC-LOBBY-19 | covered (client LobbyPage join busy-lock) |
| SC-LOBBY-20 | covered (client game store clear rooms on leave/resubscribe) |
| SC-LOBBY-21 | covered (client — map required; capacity in options) |
| SC-LOBBY-22 | covered (revised — chosen maxSeats ≤ map.players) |
| SC-LOBBY-23 | covered (client + server) |
| SC-LOBBY-24 | covered (client vitest LobbyCreateWire — no set author; `#{n}` ordinal) |
| SC-LOBBY-25 | covered (revised — card chrome: preview + seats/tourists metric rows + pack; maxSeats) |
| SC-LOBBY-26 | covered (revised — per-set rows with taskCount; no author; ordinal labels) |
| SC-LOBBY-27 | covered (server mocha create reject) |
| SC-LOBBY-28 | covered (client — seats picker 1…map.players) |
| SC-LOBBY-29 | covered (client — no duplicate capacity caption) |
| SC-LOBBY-30 | covered (client — auto-check single published set) |
| SC-LOBBY-31 | covered (client vitest LobbyCreateWire selected mini) |
| SC-LOBBY-32 | covered (lobby listing metadata includes taskCount per selected set) |
| SC-LOBBY-33 | covered (card chrome ~180; outline Войти; pack-card tokens) |
| SC-LOBBY-34 | covered (phase status overlay centered; short ОЖИДАНИЕ / ИГРА) |
| SC-LOBBY-35 | covered (≤4 set rows + overflow «ещё {k}») |
| SC-LOBBY-36 | covered (seats/tourists rows: label left + count right; same lead column as sets) |

Related: map snapshot / task deck — `game/board`; seating capacity — `game/pieces` / `game/start`. Grille/catapult density unchanged (SC-LOBBY-13…18). Pack catalog set-preview overflow SC-PACK-249; shared card chrome SC-PACK-255 / SC-MAP-55/66/67.

## Requirements

### Requirement: Live lobby listing subscription

The system SHALL provide an authorized user on the lobby screen with a live listing of available `tourist` game rooms over a realtime Colyseus connection to the lobby listing room, without periodic HTTP polling of the room list while the lobby screen is active.

#### Scenario [SC-LOBBY-01]: Subscribe on lobby entry

- **GIVEN** the user is authenticated (registered or anonymous guest) and navigates to the lobby screen
- **WHEN** the lobby screen becomes active
- **THEN** the client establishes a realtime subscription to the lobby listing filtered to room name `tourist`
- **AND** the UI receives an initial snapshot of available `tourist` rooms
- **AND** the client does not rely on a repeating HTTP poll interval to refresh that list while the subscription is active

#### Scenario [SC-LOBBY-02]: Room appears in listing

- **GIVEN** at least one client is subscribed to the lobby listing for `tourist`
- **WHEN** a new `tourist` room becomes available for listing
- **THEN** every subscribed lobby client SHALL receive an update that adds that room to the list without requiring a manual refresh

#### Scenario [SC-LOBBY-03]: Room leaves listing

- **GIVEN** a `tourist` room is visible in the live lobby listing
- **WHEN** that room is disposed or otherwise removed from the public listing
- **THEN** every subscribed lobby client SHALL receive an update that removes that room from the list

### Requirement: Canonical tourist room name

The server SHALL register the playable game room under the name `tourist`, matching the client contract for create, join, joinOrCreate, and lobby filter.

#### Scenario [SC-LOBBY-04]: Create and join by tourist name

- **GIVEN** the server is running with the game room registered as `tourist`
- **WHEN** an authenticated client creates or joins a game via the lobby actions (create / joinOrCreate / join by id of a listed `tourist` room)
- **THEN** the operation targets the `tourist` room type
- **AND** newly created rooms are eligible for the live lobby listing of `tourist`

### Requirement: Leave lobby listing when entering a game

While the user is on the game screen in an active `tourist` session, the client SHALL NOT keep an active lobby listing subscription. The lobby listing subscription MUST be closed before or when entering a game room, and MAY be re-established when the user returns to the lobby screen.

#### Scenario [SC-LOBBY-05]: Unsubscribe before or on game enter

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **WHEN** the user successfully enters a `tourist` game (create / join by listed room)
- **THEN** the lobby listing subscription is closed
- **AND** the client navigates to the game screen with an active game room connection

#### Scenario [SC-LOBBY-06]: Resubscribe on return to lobby

- **GIVEN** the user left the lobby listing subscription when entering a game
- **WHEN** the user returns to the lobby screen without an active need for the game session listing
- **THEN** the client establishes a new lobby listing subscription and shows the current `tourist` room list

### Requirement: Lobby listing errors

If establishing the lobby listing subscription fails on lobby entry (or the listing remains unavailable after a quiet resubscribe attempt), the system SHALL surface an error on the lobby screen so the user can understand that the room list is unavailable, without blocking unrelated navigation such as logout. Transient disconnects of an already-established lobby listing subscription, automatic reconnect failures, and matchmaking messages such as seat reservation expired that arise from lobby reconnect attempts MUST NOT be treated as that user-facing listing error; the client MAY quietly clear the subscription and resubscribe while the lobby screen is still active.

#### Scenario [SC-LOBBY-07]: Failed lobby subscribe

- **GIVEN** the user is on the lobby screen
- **WHEN** the lobby listing subscription fails to connect on entry (or remains unavailable after a quiet resubscribe attempt)
- **THEN** the lobby UI shows an error indication for the listing
- **AND** the user can still leave the lobby (for example log out)

### Requirement: Lobby subscription has no reconnect hold

The lobby listing room MUST NOT hold a disconnected subscriber via reconnection grace. The client MUST NOT persist a lobby reconnection token in localStorage or sessionStorage for page reload. A dropped lobby listing connection while the user remains on the lobby screen MAY be followed by a new listing subscription without showing seat-reservation or reconnect failure text to the user.

#### Scenario [SC-LOBBY-08]: Lobby drop does not surface reservation expired

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **WHEN** that subscription drops unexpectedly or a lobby reconnect/reservation attempt fails
- **THEN** the UI does not show a seat reservation expired (or equivalent reconnect) error for that listing drop
- **AND** the client MAY establish a fresh lobby listing subscription while the lobby screen stays active
- **AND** room list updates resume after a successful resubscribe

### Requirement: Create game chooses an in-catalog map

When an authenticated user opens create-game from the lobby, the system SHALL require selecting exactly one **in-catalog** content map. Soft-unpublished or never-published maps MUST NOT be selectable. Cancelling MUST NOT create a room. The map’s `players` value is the **ceiling** for seat selection (see seats requirement). Tourists per player and play layout MUST come from that map’s live snapshot at create. The create UI MUST expose each map’s players × tourists in the map **option** list (or equivalent picker chrome) and MUST NOT repeat the same capacity string as a separate caption under the closed map select after selection.

#### Scenario [SC-LOBBY-21]: Create requires map and shows capacity in options

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the user MUST select an in-catalog map before confirm is allowed
- **AND** each map option shows that map’s players × tourists per player

#### Scenario [SC-LOBBY-29]: No duplicate capacity caption under map select

- **GIVEN** the create modal and an in-catalog map is selected
- **WHEN** the map select is closed
- **THEN** the UI MUST NOT show a second caption line under the select that only repeats the same players × tourists text already conveyed by the picker options / selection

### Requirement: Create game chooses seat count up to map players

When an authenticated user has selected an in-catalog map, the create UI SHALL require choosing **maxSeats** as an integer from **1** through that map’s **players** inclusive. The default after map selection MUST be **`min(2, map.players)`**. Confirming create MUST create a `tourist` room whose synced `maxSeats` equals the chosen value (not necessarily the map’s full `players`). The server MUST reject create when `maxSeats` is missing/invalid or greater than the map’s `players` (or less than 1). When `maxSeats` is omitted by a legacy client, the server MAY default to `min(2, map.players)`.

#### Scenario [SC-LOBBY-28]: Seats picker respects map ceiling

- **GIVEN** the create modal has in-catalog map M with players=4 selected
- **WHEN** the seats control is shown
- **THEN** the user may choose 1, 2, 3, or 4
- **AND** the default selection is 2

#### Scenario [SC-LOBBY-22]: Confirm create uses chosen maxSeats

- **GIVEN** the create modal has in-catalog map M with players=3 and touristsPerPlayer=2 selected and the user chose maxSeats=2 with valid task-set selection
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with maxSeats 2
- **AND** each seated player will receive 2 pieces at playing materialize per `game/pieces`

### Requirement: Create game chooses task sets from one pack

When an authenticated user opens create-game, the system SHALL require selecting one **in-catalog** content pack and one or more of that pack’s **published** task sets (soft-unpublished sets MUST NOT appear). The picker MUST show each set with the pack title (theme) and a task-set ordinal label **without** the task-set author display name (preferred form «Набор заданий #{n}» or equivalent with a hash before the ordinal). The user MAY select a single set or multiple sets via checkboxes, including selecting all listed published sets of that pack. Task sets from a different pack MUST NOT be combinable in one create. When the selected pack has **exactly one** published task set, the create UI MUST pre-check that set. Confirming create MUST pass the selected pack and task-set identities into the room so the server snapshots questions, difficulties, slots, and answer cards for play. Create MUST be rejected when no published task set is selected.

#### Scenario [SC-LOBBY-23]: Multi-select sets only within one pack

- **GIVEN** the create modal and in-catalog pack P with two published task sets S1 and S2
- **WHEN** the user selects S1 and S2
- **THEN** both remain selected under P
- **AND** the user MUST NOT be able to combine a set from another pack in the same create

#### Scenario [SC-LOBBY-24]: Set picker shows pack theme and author

- **GIVEN** published task set S on pack titled «Математика» authored by a user whose display name is «Иван»
- **WHEN** the create modal lists sets for that pack
- **THEN** the row shows the pack title «Математика» and a task-set ordinal label with a hash before the ordinal and without «Иван» / author text

#### Scenario [SC-LOBBY-30]: Single published set is pre-checked

- **GIVEN** the create modal and in-catalog pack P with exactly one published task set S
- **WHEN** the task-set picker renders
- **THEN** S is pre-checked

#### Scenario [SC-LOBBY-27]: Create rejected without task set

- **GIVEN** the create modal has a valid map selected and no published task set selected
- **WHEN** the user confirms create
- **THEN** create is rejected or blocked until at least one published set is selected

### Requirement: Lobby lists occupied seats over max seats

Each listed `tourist` room in the live lobby listing SHALL show occupied seated count over maxSeats in the form `occupied/maxSeats` (for example `2/4`). Spectators (non-seated clients) MUST NOT increase the occupied numerator. The denominator MUST be that room’s maxSeats.

#### Scenario [SC-LOBBY-11]: Listing shows seats over maxSeats

- **GIVEN** a listed tourist room with maxSeats 4, two seated players, and one spectator
- **WHEN** a lobby subscriber views that room row
- **THEN** the capacity text shows `2/4`
- **AND** does not count the spectator in the numerator

### Requirement: Lobby lists map capacity, preview, and task sets

Each listed `tourist` room in the live lobby listing MUST appear as a **card** (not a dense list-only row) with resting width about **180** CSS pixels (height MAY grow with preview, capacity rows, pack title, set rows, and the join action). The card MUST show, top to bottom: a **mini preview** of the room’s map grid; a **seats metric row** (leading seat-style icon, declined seats noun on the left, **occupied/maxSeats** right-aligned, for example `2/4`); a **tourists metric row** (leading backpack-style icon, declined tourists noun on the left, bare tourists-per-player count right-aligned **without** a leading plus); the **pack title** (uppercase, truncated); then **one row per selected task set** (leading document-style icon, ordinal label with hash product sense «Набор #{n}» or equivalent, **task count** right-aligned) **without** task-set author names. Seats and tourists rows MUST use the same left-to-right metric rhythm as set rows (shared lead column / label edge / count edge) and MUST NOT be a centered icon+combined-text block. Seats noun pluralization MUST follow **maxSeats**; tourists noun pluralization MUST follow **touristsPerPlayer**. Ordinals `{n}` are 1-based among the room’s selected sets in listing order. When more than **four** selected sets exist, the card MUST show at most four rows plus an overflow caption product sense «ещё {k}» where `{k}` is the number of selected sets not shown. Occupied/maxSeats MUST use the room’s chosen `maxSeats` (not the map’s full `players` when those differ). Join MUST remain available via the card body and via a bottom full-width **outline** «Войти» control; join busy-lock (SC-LOBBY-19) MUST still apply.

#### Scenario [SC-LOBBY-25]: Listing shows map preview and room capacity

- **GIVEN** a listed tourist room created from map M with players=4 and touristsPerPlayer=3 where the creator chose maxSeats=2 and two seats are occupied
- **WHEN** a lobby subscriber views that room card
- **THEN** the card shows a mini preview of M’s grid
- **AND** shows a seats row with a declined seats label and occupied/maxSeats using maxSeats 2 on the right (product sense `2/2` when full, or `occupied/2`)
- **AND** shows a tourists row with a declined tourists label and count 3 on the right without a leading plus
- **AND** MUST NOT present capacity as a lone `players×tourists` caption in place of those rows

#### Scenario [SC-LOBBY-36]: Seats and tourists rows match set-row metric layout

- **GIVEN** a listed tourist room with seats, touristsPerPlayer, and at least one selected task set
- **WHEN** a lobby subscriber views that room card
- **THEN** seats and tourists rows each show lead icon, left label, and right count in the same column rhythm as set rows
- **AND** seats/tourists lead icons share the set-row lead column (not a centered capacity cluster)
- **AND** the seats label reflects pluralization by maxSeats and the tourists label by touristsPerPlayer

#### Scenario [SC-LOBBY-26]: Listing shows played task sets

- **GIVEN** a listed tourist room created with pack titled «Математика» and two selected task sets (authors «Мария» and «Иван») with task counts 48 and 32
- **WHEN** the lobby listing renders that room card
- **THEN** the card shows pack title «Математика» (uppercase presentation)
- **AND** shows two set rows with ordinal labels (hash before ordinal) and counts 48 and 32
- **AND** MUST NOT show «Мария» / «Иван» / task-set author as the set identity

#### Scenario [SC-LOBBY-33]: Lobby room card uses pack-card chrome and join outline

- **GIVEN** at least one listed tourist room on the lobby screen
- **WHEN** a subscriber views the listing
- **THEN** each room is a card about 180 CSS px wide with outline «Войти» at the bottom
- **AND** card surface / border / hover / action height follow the same product chrome as pack and map list cards (no primary-filled join button as the sole chrome)

#### Scenario [SC-LOBBY-35]: More than four selected sets show overflow

- **GIVEN** a listed tourist room with six selected task sets
- **WHEN** the lobby listing renders that room card
- **THEN** the card shows at most four set-preview rows with ordinals and counts
- **AND** shows an overflow caption indicating two more sets (product sense «ещё 2»)

### Requirement: Lobby listing metadata includes per-set task counts

For each listed `tourist` room that has a content snapshot, the live lobby listing metadata MUST include, for every selected task set exposed to the listing, a non-negative integer **task count** equal to the number of tasks from that set present in the room’s create-time content snapshot (deck composition at create). The client MUST use those counts on the room card set rows. Task-set author display names MAY remain in metadata for compatibility but MUST NOT be required for the card UI.

#### Scenario [SC-LOBBY-32]: Listing metadata carries taskCount per selected set

- **GIVEN** a `tourist` room created with two selected sets that contributed 48 and 32 tasks to the snapshot
- **WHEN** a lobby subscriber receives that room in the live listing
- **THEN** the room metadata includes those two sets with task counts 48 and 32 (or equivalent fields the client maps to the card)
- **AND** spectators or later play state MUST NOT be required for those create-time counts to appear

### Requirement: Lobby room card shows phase status on the map preview

Each listed `tourist` room card MUST show the room phase status as a short uppercase badge overlaid on the mini map preview, **horizontally centered** (near the top of the preview). Waiting rooms MUST use product sense «ОЖИДАНИЕ»; playing rooms MUST use product sense «ИГРА» (or equivalent i18n). Badge size and soft-pill colors MUST match the product sense of existing pack/task-set/map short status badges (not Quasar solid primary fills). Join affordances remain available per SC-LOBBY-19 / SC-LOBBY-25 regardless of waiting vs playing (spectator join rules unchanged).

#### Scenario [SC-LOBBY-34]: Waiting and playing show centered short status

- **GIVEN** one listed room in waiting phase and one listed room in playing phase
- **WHEN** a lobby subscriber views both cards
- **THEN** the waiting card shows short «ОЖИДАНИЕ» centered on the map preview
- **AND** the playing card shows short «ИГРА» centered on the map preview

### Requirement: Play shortcut removed from lobby

The lobby screen MUST NOT offer a primary «Играть» / joinOrCreate shortcut that enters an arbitrary tourist room without choosing a listed room. Users MUST create a game (with max seats) or join a specific room from the live listing.

#### Scenario [SC-LOBBY-12]: No play shortcut on lobby

- **GIVEN** an authenticated user on the lobby screen
- **WHEN** the lobby actions are shown
- **THEN** there is no «Играть» action that performs joinOrCreate into an arbitrary room
- **AND** create-game and join-by-listed-room actions remain available

### Requirement: Create game chooses grille density

When an authenticated user opens create-game from the lobby, the system SHALL present a choice of grille density with three presets: few, medium, and many. The presets MUST map to **12%**, **22%**, and **35%** of task cells on the tourist layout at playing seed time (rounded to nearest integer, clamped to the task-cell count). Confirming create MUST pass the selected density into the new `tourist` room create options. Cancelling MUST NOT create a room. Product labels remain мало / средне / много without showing the percent numbers.

#### Scenario [SC-LOBBY-13]: Density presets are few medium many

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the selectable grille density options are few, medium, and many
- **AND** the product sense of the labels is мало / средне / много

#### Scenario [SC-LOBBY-14]: Confirm create uses selected density

- **GIVEN** the create modal is open with grille density many selected
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with grille density many (35% of task cells at seed)
- **AND** the user enters that game session

#### Scenario [SC-LOBBY-15]: Default grille density is medium

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** medium (22%) is selected by default for grille density
- **AND** max seats default remains 2 per existing create rules

### Requirement: Create game chooses catapult density

When an authenticated user opens create-game from the lobby, the system SHALL present a choice of catapult density with three presets: few, medium, and many, **independent** of the grille density choice. The presets MUST map to **12%**, **22%**, and **35%** of task cells on the tourist layout at playing seed time (rounded to the nearest integer, clamped to the task-cell count). Confirming create MUST pass the selected catapult density into the new `tourist` room create options alongside grille density. Cancelling MUST NOT create a room. Product labels remain мало / средне / много without showing the percent numbers.

#### Scenario [SC-LOBBY-16]: Catapult density presets are few medium many

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** the selectable catapult density options are few, medium, and many
- **AND** the product sense of the labels is мало / средне / много
- **AND** catapult density is chosen separately from grille density

#### Scenario [SC-LOBBY-17]: Confirm create uses selected catapult density

- **GIVEN** the create modal is open with catapult density many selected
- **WHEN** the user confirms create
- **THEN** a new `tourist` room is created with catapult density many (35% of task cells at seed)
- **AND** the user enters that game session

#### Scenario [SC-LOBBY-18]: Default catapult density is medium

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** medium (22%) is selected by default for catapult density
- **AND** grille density default remains medium per existing create rules

### Requirement: Join action is busy-locked

While a lobby join into a listed `tourist` room is in progress, the client MUST prevent a second join attempt from the same listing (including repeated activation of the same or another listed row). The busy state MUST be visible on the join control. Create-game MAY continue to use its existing modal busy pattern; join MUST NOT leave listing rows fully clickable without a re-entrancy guard.

#### Scenario [SC-LOBBY-19]: Second join click ignored while connecting

- **GIVEN** the user is on the lobby screen with at least one listed room
- **WHEN** the user starts join for a listed room and activates join again before the first attempt finishes
- **THEN** only one join attempt proceeds
- **AND** the join control shows a busy/loading state during the attempt

### Requirement: Lobby listing clears stale rooms on resubscribe

When the client leaves a tourist game and returns to the lobby listing (or otherwise resubscribes to the live lobby list), the client MUST NOT briefly show a stale prior room list as if it were the current live snapshot. The listing MUST either stay in a loading/empty-safe state until a fresh lobby rooms snapshot arrives, or clear the previous rooms collection before presenting rooms again after resubscribe.

#### Scenario [SC-LOBBY-20]: No ghost room flash after leaving last seat

- **GIVEN** the user was the last seated player in a tourist room and returns to the lobby screen
- **WHEN** the lobby listing resubscribes
- **THEN** the user MUST NOT see a flash of that disposed room as an available game between loading and the fresh empty (or updated) list
- **AND** after the fresh lobby snapshot, disposed rooms are absent from the list

### Requirement: Create map select shows mini preview when selected

When an authenticated user opens create-game and selects an in-catalog map, the closed/selected presentation of the map picker MUST show a **mini preview** of that map’s grid together with the map’s identifying label (not label-only text). Option rows in the open list MUST continue to show mini preview and players×tourists per existing create-map picker rules.

#### Scenario [SC-LOBBY-31]: Selected map shows mini preview in create select

- **GIVEN** the create-game modal with an in-catalog map M selected
- **WHEN** the user views the closed map select
- **THEN** the selected presentation includes a mini preview of M’s grid
- **AND** the map’s label remains visible
