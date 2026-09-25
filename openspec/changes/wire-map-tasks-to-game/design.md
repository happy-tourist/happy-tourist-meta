## Context

См. `proposal.md` — Why. Сейчас `tourist` room: hardcoded `TOURIST_LAYOUT` / client `LAYOUT`, create options только `maxSeats` + grille/catapult density, peeks = private stub Correct/Wrong + bag 28/14/6, pcs всегда 4 со side N/E/S/W. Content maps/packs живут в HTTP CMS и явно «не wired». Решения explore D1–D22 зафиксированы в specs delta.

Пакеты: **server** (`happy-tourist-server`) + **client** (`happy-tourist.github.io`). Контракт room name `tourist` согласованно.

## Goals / Non-Goals

**Goals:**
- Snapshot map + pack answers + selected task sets в room на create
- Динамический board/pcs/spawn; task deck → bind; shared peek + server order check
- Lobby create/listing UX; HUD focus control
- Убрать hardcoded layout и stub peek из runtime path

**Non-Goals:**
- Shop / покупка тайла; отдельное theme-поле у task set; смешение packs; CMS redesign

## Decisions

### D1. Snapshot at create (не live join к CMS)

Room на `onCreate` читает live map + pack/task sets из GameDatabase / content helpers и кладёт **immutable snapshot** в room memory (+ минимальные synced поля для lobby/UI: grid string, players, touristsPerPlayer, pack title, set labels, flipped cell→taskId, difficulty digit).

**Альтернатива:** подгружать CMS на каждый peek — отвергнуто (unpublish mid-game, latency).

### D2. Create options shape

`client.create('tourist', { mapId, packId, taskSetIds: string[], grilleDensity, catapultDensity })`. Server выставляет `maxSeats = map.players`. Убрать отдельный maxSeats из UI.

### D3. Piece identity без side

Убрать `side` из piece schema / strip index; piece id = seat-local index `0..touristsPerPlayer-1`. Strip и move/peek/rescue/push/return адресуют piece by id. All-jail и materialize — greedy max-min distance на free starts карты.

**Альтернатива:** сохранять фиктивные стороны — отвергнуто (карты произвольные).

### D4. Peek protocol (shared)

- Synced: `peekSession` (seatId, cell, taskId, slotPlacements[]) или broadcast messages `peekOpen` / `peekPlace` / `peekSubmit` / `peekClose` видны всем.
- Private budgets остаются private.
- Fresh open: −1 peek; flipped open: 0 cost, allowed at peeks=0.
- Submit: server compares ordered answer card ids to task slots.

### D5. Lobby metadata

Metadata room listing: map preview grid (или compact string), capacity, pack/set labels — через Colyseus room metadata / lobby filter fields already used for seats.

### D6. Client layers

- `LobbyPage` create modal: map picker → capacity; pack → multi-check sets; densities.
- `game` store: create options; peek place/submit; focus action helper.
- `GamePage` / board: render snapshot grid, flipped digits, shared modal.
- Presence chrome: circular focus between say and end-turn.
- Maps/content stores: list in-catalog for pickers (reuse existing HTTP).

### D7. Server layers

- `MyRoom.onCreate`: validate map/pack/sets published; snapshot; set maxClients/maxSeats.
- `touristMove` / board helpers: layout from snapshot; deck; bind map; remove fixed bag / START_CELLS sides.
- Schema: grid, touristsPerPlayer, cellBindings, peekSession; pieces without side.
- Messages: extend peek*; add peekPlace.
- Tests: mocha for create reject, bind, order check, flipped free peek, distance spawn.

### Prerequisites (из explore — закрыты)

D1–D22 приняты пользователем; блокеров нет. Assumption: «nearest actionable» = минимальное манхэттенское расстояние от текущего selection (или от центра доски, если нет selection); ties — меньший piece index.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Ломаются тесты/skills, завязанные на 4 pcs + sides | Обновить server/client tests и piece addressing одним PR-порядком: server contract → client |
| Большой pack answers в модалке | Принято (D16); модерация CMS |
| Lobby metadata size (full grid) | Компактная grid string 100 chars уже у maps |
| maxSeats=1 auto countdown | Уже full-table path; проверить solo peeks infinite не включается ошибочно при 1 из 1 until others leave — при одном seated после start это и есть solo rules |

## Migration Plan

1. Deploy server accepting new create options (reject legacy maxSeats-only creates).
2. Deploy client create modal + board/peek.
3. Rollback: revert client first (old client cannot create); server may keep rejecting old options.

Чеклист реализации — `tasks.md`.
