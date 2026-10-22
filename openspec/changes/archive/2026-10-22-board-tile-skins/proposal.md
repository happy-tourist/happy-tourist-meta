## Why

Клетки поля сейчас — плоские CSS-заливки; в клиенте уже есть PNG-скины (`start` / `tile` / `tile-back` / `finish`). Нужно показать их на всех поверхностях карты и упростить финиш: вместо одного визуального 2×2 — отдельные клетки, как при paint в редакторе.

## What Changes

- Скины клеток на игровой доске, в mini-preview карт и в палитре редактора.
- Обычный таск = `tile`; после flip = `tile-back` + читаемая цифра сложности.
- Финиш = по одной клетке с `finish` (без merge в один `span 2×2`).
- Клик и outline по finish — как у обычных клеток (конкретная клетка).

## Scope

- **Capability IDs:** `game/board`, `game/move`, `content/maps`
- **Пакет:** client only (`happy-tourist.github.io`)
- Ассеты: `src/assets/tiles/*.png` (уже подготовлены: квадрат, без полей)

## Out of scope

- Server / room schema / messages / HTTP / DB.
- Анимация переворота тайла.
- Новые npm-зависимости.
- Перерисовка туристов / решёток / катапульт.
- Смена правил хода, peek, finish-authority (кроме клиентского resolve клика по конкретной finish-клетке).
- Origin анимации return-from-finish (nearest of four centers, SC-FINISH-15) — без изменений.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/board`: PNG-скины клеток; finish по одной клетке; flipped `tile-back` + читаемая цифра.
- `game/move`: клик/подсветка finish как у обычных клеток (вместо nearest по всему 2×2).
- `content/maps`: preview и палитра paint используют те же скины kinds.

## Impact

- Client: игровая доска, `MapGridPreview` (список карт, редактор, lobby room cards), палитра инструментов карты; локальная геометрия тайлов (без merge center).
- **Server / BE:** нет задач и нет правок. Контракт **as-is** (room `tourist`): каждый grid-char `7` — отдельный playable center; client message `move` `{ pieceId, row, col }` валидируется на эту клетку; synced `flippedCells` уже отдаёт difficulty для цифры. Меняется только клиентский chrome (скины, без merge, клик без nearest-remap).

## References

- Client: `happy-tourist.github.io/AGENTS.md`; skills `work-with-game-board`, `work-with-styles` (board), `work-with-pages`.
- Main specs: `openspec/specs/game/board/spec.md`, `openspec/specs/game/move/spec.md`, `openspec/specs/content/maps/spec.md`.
- Explore: скины везде; center = 4 клетки; тёмная цифра на `tile-back`.
