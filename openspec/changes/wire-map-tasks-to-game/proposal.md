## Why

Партии до сих пор играют на захардкоженной раскладке и stub-peek («Правильно»/«Неправильно» без контента), хотя UGC-карты и наборы заданий уже в каталоге. Нужно при создании игры выбирать карту и опубликованные наборы заданий и разыгрывать реальные вопросы с общей модалкой и серверной проверкой.

**Follow-up (после первой реализации):** create должен позволять выбрать число мест **не больше** ёмкости карты; spawn своих туристов — с минимальной дистанцией, но рандомнее (не детерминированные «углы»); мелкий UX create/HUD (без дубля ёмкости под селектом, авточек единственного set, say выше focus).

## What Changes

- **BREAKING:** create `tourist` больше не использует фиксированный layout и bag наград 28/14/6; обязательны опубликованная карта и ≥1 опубликованный набор заданий одного pack’а.
- Create: выбор карты; **число мест `1…map.players`** (default `min(2, map.players)`); pack + галочки task set(ов); grille/catapult density; один опубликованный set — авточек; без дублирующей подписи ёмкости под селектом карты.
- Runtime board/pcs с snapshot карты; без сторон света N/E/S/W; свои туристы на free starts с **min-distance floor + random** (не жёсткий max-pair greedy).
- Peek: колода заданий → bind на клетку при первом открытии; публичная цифра сложности; shared Q&A-модалка; сервер сверяет порядок слотов; flipped peek без траты peeks (в т.ч. при 0).
- Lobby listing: превью карты; ёмкость комнаты = **выбранный** `maxSeats` × tourists; какой набор разыгрывается.
- HUD: круглая кнопка фокуса на actionable tourist; **say** визуально выше, чтобы не пересекаться с focus.

## Scope

- **Capability ID:** `lobby/rooms`, `game/board`, `game/pieces`, `game/move`, `game/presence`, `content/maps`, `content/packs` (delta); при необходимости согласованные правки `game/start` / `game/finish` под ёмкость карты и pcs без сторон
- **Пакеты:** client (`happy-tourist.github.io`) + server (`happy-tourist-server`)
- **Контракт:** room `tourist` create options (`mapId`, `packId`, `taskSetIds`, **`maxSeats`**, densities) + synced board/tasks/peek; HTTP content только как источник snapshot при create
- **Экраны:** Lobby (create + listing), Game (board, peek modal, HUD focus/say)

## Out of scope

- Покупка тайла на дыру / shop
- Отдельная «тема» у task set (используется title pack’а + автор set)
- Выбор заданий по одной штуке вне set; смешение set’ов из разных pack’ов
- Изменение правил модерации maps/packs (кроме снятия «не wired to rooms» / stub-only)
- Новые плотности grille/catapult или привязка traps к карте как контенту
- Blocked packs в create (отклонять / не показывать — как для unpublished)
- Выбор `touristsPerPlayer` отдельно от карты (только с snapshot карты)

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create options карта + **выбираемый maxSeats ≤ map.players** + pack/task sets; listing meta (превью, выбранная ёмкость, наборы); UX auto-check / без дубля caption
- `game/board`: layout из map snapshot; колода/bind/flipped digit; shared peek Q&A; награда = difficulty
- `game/pieces`: pcs = touristsPerPlayer; spawn **min-distance floor + random**; без side identity
- `game/move`: peeks economy для fresh vs flipped; согласование с новым peek
- `game/presence`: круглая кнопка фокуса; say выше focus
- `content/maps`: снять запрет wiring в tourist-room; create использует published maps
- `content/packs`: снять «peek stub only»; published task sets / answers питают room

## Impact

- Server: `onCreate` snapshot map+tasks; validate `maxSeats` ≤ map.players; schema board/tasks/flipped; peek messages; materialize/all-jail с floor+random
- Client: create modal (seats + map/pack), lobby rows, динамический board, shared peek UI, strip N слотов, focus + say spacing
- Content HTTP: read live map/pack при create (без новой CMS-модели)

## References

- Explore D1–D22 + follow-up (seats ≤ map; spawn A; listing = chosen max; auto-check; say up)
- Main: `openspec/specs/lobby/rooms`, `game/board`, `game/pieces`, `game/move`, `game/presence`, `content/maps`, `content/packs`
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`; `docs/projects-map.md`
