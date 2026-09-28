## Context

См. `proposal.md` — Why / follow-up. Первая волна (snapshot map+pack, deck/peek, pieces без sides, lobby create/listing, focus) уже в runtime. Follow-up уточняет create capacity, spawn randomness и мелкий UX.

Пакеты: **server** (`happy-tourist-server`) + **client** (`happy-tourist.github.io`). Контракт room name `tourist` согласованно.

## Goals / Non-Goals

**Goals:**
- Snapshot map + pack answers + selected task sets в room на create
- `maxSeats` выбирает хост в диапазоне `1…map.players`
- Spawn/all-jail: min-distance floor + uniform random (хаос при сохранении разнесённости)
- Lobby create/listing UX; HUD focus; say не перекрывает focus
- Убрать hardcoded layout и stub peek из runtime path

**Non-Goals:**
- Shop / покупка тайла; отдельное theme-поле у task set; смешение packs; CMS redesign
- Отдельный выбор touristsPerPlayer вне карты

## Decisions

### D1. Snapshot at create (не live join к CMS)

Room на `onCreate` читает live map + pack/task sets из GameDatabase / content helpers и кладёт **immutable snapshot** в room memory (+ минимальные synced поля для lobby/UI: grid string, touristsPerPlayer, pack title, set labels, flipped cell→taskId, difficulty digit).

**Альтернатива:** подгружать CMS на каждый peek — отвергнуто (unpublish mid-game, latency).

### D2. Create options shape (revised)

`client.create('tourist', { mapId, packId, taskSetIds: string[], maxSeats, grilleDensity, catapultDensity })`.

- Server: `maxSeats` MUST быть целым `1…map.players` (из snapshot карты); иначе reject. Default на client UI: **`min(2, map.players)`** после выбора карты (на карте с `players=1` — только 1).
- `touristsPerPlayer` и grid — только из map snapshot (не из options).
- Listing metadata `maxSeats` / occupied = **выбранная** ёмкость комнаты (если карта на 4, а create выбрал 2 → в списке максимум 2).

**Альтернатива (отвергнута в первой волне, возвращена частично):** всегда `maxSeats = map.players` — хост не мог играть на меньшем столе.

### D3. Piece identity без side + spawn A (revised)

Убрать `side` из piece schema / strip index; piece id = seat-local index `0..touristsPerPlayer-1`.

**Spawn / all-jail (алгоритм A):** среди free start cells карты:

1. Вычислить `maxPair` = максимальное Chebyshev-расстояние между любыми двумя free starts.
2. `floor = ceil(maxPair / 2)`.
3. Жадно набирать `touristsPerPlayer` клеток: на каждом шаге кандидаты = free starts, у которых min-distance к уже выбранным **своим** ≥ `floor`; взять **uniform random** среди кандидатов; занять клетку.
4. Если кандидатов нет — `floor -= 1` и повторить шаг 3, пока `floor ≥ 0` (при 0 — любой оставшийся free start).

Opponent pieces занимают остальные free starts по тому же правилу для следующего seat. Незанятые старты карты остаются пустыми — ок.

**Альтернативы:** жёсткий greedy max-min (детерминированные углы) — отвергнуто follow-up; чистый random без floor — отвергнуто (свои могут встать рядом).

### D4. Peek protocol (shared)

- Synced: peek session + place/submit; private budgets private.
- Fresh open: −1 peek; flipped open: 0 cost, allowed at peeks=0.
- Submit: server compares ordered answer card ids to task slots.

### D5. Lobby metadata

Map preview grid, **room** capacity (`maxSeats` × touristsPerPlayer или occupied/`maxSeats` + tourists copy), pack/set labels — через Colyseus room metadata.

### D6. Client create UX (revised)

- Map picker: ёмкость карты видна в option caption; **не** дублировать той же строкой под закрытым селектом.
- После выбора карты — seats control `1…map.players`, default `min(2, map.players)`.
- Pack → multi-check published sets; если опубликован ровно **один** set — сразу отметить его.
- Densities без изменений.

### D7. Server layers

- `MyRoom.onCreate`: validate map/pack/sets + `maxSeats`; snapshot; metadata.
- `pickGreedyMaxDistanceStarts` → заменить / расширить на floor+random helper; materialize + all-jail.
- Schema / peek / pieces without side — как в первой волне.

### D8. Say vs focus spacing

Say affordance на own avatar MUST сидеть **выше** по вертикали (дальше от центра / ближе к верхнему краю), чтобы hit-area не пересекалась с focus (между say и end-turn). Pure CSS + при необходимости лёгкий сдвиг focus вниз.

### Prerequisites

Explore follow-up закрыт: seats ≤ map; spawn A; listing = chosen max; auto-check; say up. «Nearest actionable» для focus — без изменений (Manhattan / ties → меньший piece index).

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Floor `ceil(maxPair/2)` слишком жёсткий / мягкий на кривых картах | Понижение floor; после playtest можно подкрутить константу без смены контракта «floor+random» |
| Ломаются тесты на детерминированный max-pair spawn | Переписать SC-PIECE-50 на инварианты (min distance ≥ floor при наличии кандидатов; не соседство при достаточном числе стартов) |
| Client без `maxSeats` в options | Server reject; deploy client+server вместе для follow-up |

## Migration Plan

1. Server: принимать `maxSeats` ≤ map.players; новый spawn; старый client без seats picker может слать отсутствующий maxSeats — либо default `min(2, map.players)` на server, либо reject (предпочтительно **default** `min(2, map.players)` если поле omitted, для мягкого перехода).
2. Client: seats picker + UX polish + say CSS.
3. Rollback: client first.

Чеклист — `tasks.md` (блок 8 = follow-up).
