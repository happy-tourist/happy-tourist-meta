## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-11 | covered / reinforced (seats `occupied/maxSeats` on card) |
| SC-LOBBY-19 | covered / reinforced (join busy-lock; card + button) |
| SC-LOBBY-25 | revised (card chrome: preview + seats + tourists without plus + pack title; maxSeats capacity; metric rows) |
| SC-LOBBY-26 | revised (per-set rows with taskCount; no author; ordinal labels) |
| SC-LOBBY-32 | covered (lobby listing metadata includes taskCount per selected set) |
| SC-LOBBY-33 | covered (card chrome ~180; outline Войти; pack-card tokens) |
| SC-LOBBY-34 | covered (phase status overlay centered; short ОЖИДАНИЕ / ИГРА) |
| SC-LOBBY-35 | covered (≤4 set rows + overflow «ещё {k}») |
| SC-LOBBY-36 | covered (seats/tourists rows: label left + count right; same lead column as sets; declined labels) |

Related: pack catalog set-preview overflow SC-PACK-249; shared card chrome SC-PACK-255 / SC-MAP-55/66/67. Create-modal SC-LOBBY-21…31 unchanged except listing presentation.

## MODIFIED Requirements

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

## ADDED Requirements

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
