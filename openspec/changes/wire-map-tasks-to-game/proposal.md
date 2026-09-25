## Why

Партии до сих пор играют на захардкоженной раскладке и stub-peek («Правильно»/«Неправильно» без контента), хотя UGC-карты и наборы заданий уже в каталоге. Нужно при создании игры выбирать карту и опубликованные наборы заданий и разыгрывать реальные вопросы с общей модалкой и серверной проверкой.

## What Changes

- **BREAKING:** create `tourist` больше не использует фиксированный layout и bag наград 28/14/6; обязательны опубликованная карта и ≥1 опубликованный набор заданий одного pack’а.
- Create: выбор карты (ёмкость с карты, в т.ч. 1 игрок), pack + галочки task set(ов) из него, grille/catapult density как сейчас.
- Runtime board/pcs с snapshot карты; без сторон света N/E/S/W; свои туристы максимально далеко друг от друга.
- Peek: колода заданий → bind на клетку при первом открытии; публичная цифра сложности; shared Q&A-модалка; сервер сверяет порядок слотов; flipped peek без траты peeks (в т.ч. при 0).
- Lobby listing: превью карты, игроки×туристы, какой набор разыгрывается.
- HUD: круглая кнопка фокуса на actionable tourist (между say и end-turn).

## Scope

- **Capability ID:** `lobby/rooms`, `game/board`, `game/pieces`, `game/move`, `game/presence`, `content/maps`, `content/packs` (delta); при необходимости согласованные правки `game/start` / `game/finish` под ёмкость карты и pcs без сторон
- **Пакеты:** client (`happy-tourist.github.io`) + server (`happy-tourist-server`)
- **Контракт:** room `tourist` create options + synced board/tasks/peek; HTTP content только как источник snapshot при create (live catalog maps/packs)
- **Экраны:** Lobby (create + listing), Game (board, peek modal, HUD focus)

## Out of scope

- Покупка тайла на дыру / shop
- Отдельная «тема» у task set (используется title pack’а + автор set)
- Выбор заданий по одной штуке вне set; смешение set’ов из разных pack’ов
- Изменение правил модерации maps/packs (кроме снятия «не wired to rooms» / stub-only)
- Новые плотности grille/catapult или привязка traps к карте как контенту
- Blocked packs в create (отклонять / не показывать — как для unpublished)

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create options карта + pack/task sets; listing meta (превью, ёмкость, наборы)
- `game/board`: layout из map snapshot; колода/bind/flipped digit; shared peek Q&A; награда = difficulty
- `game/pieces`: pcs = touristsPerPlayer; spawn distance; без side identity
- `game/move`: peeks economy для fresh vs flipped; согласование с новым peek
- `game/presence`: круглая кнопка фокуса actionable tourist
- `content/maps`: снять запрет wiring в tourist-room; create использует published maps
- `content/packs`: снять «peek stub only»; published task sets / answers питают room

## Impact

- Server: `onCreate` snapshot map+tasks; schema board/tasks/flipped; peek messages и валидация слотов; materialize без START_CELLS по сторонам
- Client: create modal, lobby rows, динамический board, shared peek UI, strip N слотов, focus control
- Content HTTP: read live map/pack при create (без новой CMS-модели)

## References

- Explore D1–D22 (ёмкость карты; set’ы одного pack; колода→bind; flipped публично; free re-peek; focus nearest actionable)
- Main: `openspec/specs/lobby/rooms`, `game/board`, `game/pieces`, `game/move`, `game/presence`, `content/maps`, `content/packs`
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`; `docs/projects-map.md`
