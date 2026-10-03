# Lobby room listing cards (SC-LOBBY-25/26/32…36)

Read this topic when changing **card chrome** for the live lobby room list
(`LobbyRoomCardTile`, `.pack-card-grid`, status overlay, set rows / overflow,
outline join). Subscribe / create / join lifecycle stays in [SKILL.md](SKILL.md).
Shared tokens: [../work-with-styles/pack-cards.md](../work-with-styles/pack-cards.md).

## Pieces

| Layer | Path | Role |
|-------|------|------|
| Page | `src/pages/LobbyPage.vue` | `.pack-card-grid` + `LobbyRoomCardTile` per room; `#status` badge; `#actions` outline «Войти»; join busy-lock SC-LOBBY-19 |
| Tile | `src/components/LobbyRoomCardTile.vue` | ~180 card; preview + status slot; seats/tourists metric rows; pack title; set rows ≤4 + overflow; aliases `--pack-card-*` → `--lobby-room-*` |
| Asset | `src/assets/content/lobby-room-card-seats.svg` | Seats lead mask (quoted `` `url("${…}")` `` — SC-MAP-68); tourists/set reuse map/task-set SVGs |
| Store types | `src/stores/game.ts` | `GameRoomMeta.taskSetLabels?: [{ taskSetId, authorDisplayName?, taskCount? }]` |
| i18n | `src/i18n/en-US/index.ts` → `lobby.*` | `capacity` (count cell), `roomCardSeats*`, `roomCardTourists*`, `join`, `roomCardTaskSetLabel`, `roomCardStatusWaiting` / `Playing` |
| Test ids | seats/tourists rows | Root `lobby-room-seats` / `lobby-room-tourists`; cells `…-label` + `…-count` (also `data-test-id`); set rows `lobby-room-set-row` |
| Tests | `LobbyRoomCardTile.test.ts` + `LobbyCreateWire` | SC-LOBBY-25/26/33/34/35/36 (+ busy-lock SC-LOBBY-19 on page) |

## Metadata → card

`RoomAvailable<GameRoomMeta>` listing fields used by the card:

- `metadata.seats` / `metadata.maxSeats` → seats **metric row**: label plural by **maxSeats** (`lobby.roomCardSeatsOne|Few|Many` = Место/Места/Мест); count = `lobby.capacity` `{seats} / {maxSeats}` (not `clients`/`maxClients`; not map `players`)
- `metadata.touristsPerPlayer` → tourists **metric row**: label plural by **touristsPerPlayer** (`lobby.roomCardTouristsOne|Few|Many` = Турист/Туриста/Туристов); count = bare `n` **without** leading `+` (no `mapCapacityCaption` / `players×tourists` on card)
- `metadata.packTitle` → uppercase pack title
- `metadata.taskSetLabels[]` → short `lobby.roomCardTaskSetLabel` «Набор #{n}» + right-aligned **`taskCount`** (SC-LOBBY-26/32); **ignore** `authorDisplayName` on UI; missing count → `—`
- `metadata.status` `waiting` \| `playing` → centered short badge ОЖИДАНИЕ / ИГРА (SC-LOBBY-34); tone `--muted` / `--pending`
- `metadata.mapGrid` → `MapGridPreview` ~156 inside tile

**Metric rows = set-row rhythm (SC-LOBBY-36):** seats and tourists use the same full-width grid as set rows — `lead (22) | label left | count right` (`grid-template-columns: var(--lobby-room-lead-w) minmax(0, 1fr) auto`). Lead icons share one vertical with set icons. **Not** a centered `icon+text` / `max-content` cluster.

Set rows: show at most **4**; overflow `content.packCardSetsOverflow` «ещё {k}» (SC-LOBBY-35).

## Chrome rules

| Do | Don't |
|----|--------|
| Render rooms in `.pack-card-grid` with `LobbyRoomCardTile` | Revive dense `q-list` / `q-item` as listing chrome |
| Outline join (`lobby.join` «Войти») in `#actions`; body click also joins | Primary-filled join as sole chrome |
| Alias `--pack-card-*` via `--lobby-room-*` (same host tokens as pack/map cards) | Invent a parallel lobby color system |
| Seats + tourists as **full-width** metric rows (same columns as sets); plural by maxSeats / touristsPerPlayer | Centered icon+combined-text block; combined «N ТУРИСТ…» copy |
| Quoted Vite mask `url("${…}")` for SVG leads | Unquoted `url(...)` that breaks CSS masks |
| Tolerate missing `taskCount` on older rooms | Require author strings on the card |

## Related

- Lifecycle / create modal: [SKILL.md](SKILL.md)
- Server `refreshMetadata` + `taskCount`: `.agents/skills/server/work-with-rooms/SKILL.md`
- Vitest patterns: `../work-with-test/stores.md`
