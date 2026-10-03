# Lobby room listing cards (SC-LOBBY-25/26/32…35)

Read this topic when changing **card chrome** for the live lobby room list
(`LobbyRoomCardTile`, `.pack-card-grid`, status overlay, set rows / overflow,
outline join). Subscribe / create / join lifecycle stays in [SKILL.md](SKILL.md).
Shared tokens: [../work-with-styles/pack-cards.md](../work-with-styles/pack-cards.md).

## Pieces

| Layer | Path | Role |
|-------|------|------|
| Page | `src/pages/LobbyPage.vue` | `.pack-card-grid` + `LobbyRoomCardTile` per room; `#status` badge; `#actions` outline «Войти»; join busy-lock SC-LOBBY-19 |
| Tile | `src/components/LobbyRoomCardTile.vue` | ~180 card; preview + status slot; seats/tourists; pack title; set rows ≤4 + overflow; aliases `--pack-card-*` → `--lobby-room-*` |
| Asset | `src/assets/content/lobby-room-card-seats.svg` | Seats lead mask (quoted `` `url("${…}")` `` — SC-MAP-68); tourists/set reuse map/task-set SVGs |
| Store types | `src/stores/game.ts` | `GameRoomMeta.taskSetLabels?: [{ taskSetId, authorDisplayName?, taskCount? }]` |
| i18n | `src/i18n/en-US/index.ts` → `lobby.*` | `capacity`, `join`, `roomCardTourists*`, `roomCardTaskSetLabel`, `roomCardStatusWaiting` / `Playing` |
| Tests | `LobbyRoomCardTile.test.ts` + `LobbyCreateWire` | SC-LOBBY-25/26/33/34/35 (+ busy-lock SC-LOBBY-19 on page) |

## Metadata → card

`RoomAvailable<GameRoomMeta>` listing fields used by the card:

- `metadata.seats` / `metadata.maxSeats` → `lobby.capacity` `{seats} / {maxSeats}` (not `clients`/`maxClients`; not map `players`)
- `metadata.touristsPerPlayer` → plural `lobby.roomCardTouristsOne|Few|Many` **without** leading `+` (no `mapCapacityCaption` / `players×tourists` on card)
- `metadata.packTitle` → uppercase pack title
- `metadata.taskSetLabels[]` → short `lobby.roomCardTaskSetLabel` «Набор #{n}» + right-aligned **`taskCount`** (SC-LOBBY-26/32); **ignore** `authorDisplayName` on UI; missing count → `—`
- `metadata.status` `waiting` \| `playing` → centered short badge ОЖИДАНИЕ / ИГРА (SC-LOBBY-34); tone `--muted` / `--pending`
- `metadata.mapGrid` → `MapGridPreview` ~156 inside tile

Set rows: show at most **4**; overflow `content.packCardSetsOverflow` «ещё {k}» (SC-LOBBY-35).

## Chrome rules

| Do | Don't |
|----|--------|
| Render rooms in `.pack-card-grid` with `LobbyRoomCardTile` | Revive dense `q-list` / `q-item` as listing chrome |
| Outline join (`lobby.join` «Войти») in `#actions`; body click also joins | Primary-filled join as sole chrome |
| Alias `--pack-card-*` via `--lobby-room-*` (same host tokens as pack/map cards) | Invent a parallel lobby color system |
| Seats + tourists as **centered** icon+text block (lead ~22) with pale divider | Full-width left grid like pack-list lead columns |
| Quoted Vite mask `url("${…}")` for SVG leads | Unquoted `url(...)` that breaks CSS masks |
| Tolerate missing `taskCount` on older rooms | Require author strings on the card |

## Related

- Lifecycle / create modal: [SKILL.md](SKILL.md)
- Server `refreshMetadata` + `taskCount`: `.agents/skills/server/work-with-rooms/SKILL.md`
- Vitest patterns: `../work-with-test/stores.md`
