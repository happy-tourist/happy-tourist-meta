## Context

См. `proposal.md`. Контракт реализован: 4 tokens/player (N/E/S/W) + strip×4; racefix attach-before-await сохранён. Пакеты: **server** first, затем **client**, затем meta skills.

## Goals / Non-Goals

**Goals:**

- Seated ≤4: уникальный `touristId`; ровно 4 pieces (N/E/S/W) на свободных стартах стороны.
- Все клиенты рисуют все pieces; seated — strip из 4 своих слотов 1:1 по сторонам (статус UI later).
- Leave снимает все 4; до start освобождает kind и клетки; после start новым не раздаём.
- `started` на 4-м seated (кнопка «готов» — later).

**Non-Goals:**

- Ходы / статус-хром / ready-кнопка; `maxClients=4`; owner chrome кроме разного PNG вида.

## Decisions

### D1 — Schema: seat + four pieces

- Product state:
  - `started: boolean`
  - `seats: Map` keyed by `sessionId` → `{ touristId: 1..4, pieces: Map|list of { side: N|E|S|W, row, col } }` с ровно четырьмя сторонами.
- Alternate (четыре отдельных top-level maps) — отвергнута: владелец и kind удобнее на seat.
- Client только зеркалирует sync.

### D2 — Assign on join

- До `started` и `seats.size < 4`: выбрать свободный `touristId`; для каждой стороны взять random cell из START_CELLS[side], исключая уже занятые `(row,col)` в комнате.
- Геометрия стартов без изменений: N row0 cols3–6; E col9 rows3–6; S row9 cols3–6; W col0 rows3–6.
- На 4-м seat: `started = true`, metadata `playing` optional.
- После `started`: join без записи в seats.

### D3 — Leave

- Удалить seat целиком (все 4 pieces); kind и клетки снова в пуле если `!started`.
- Если `started`: pieces убрать, `started` остаётся, новым seats нет.

### D4 — Client render

- Store: mirror `seats` / `started` / `sessionId`; `_attachRoom` **до** любого await после connect (уже в коде — сохранить).
- `GamePage`: оверлей всех pieces всех seats; strip только если есть свой seat — четыре img того же `touristId`, порядок `N,E,S,W` (слоты = полевые стороны). Статус-хром не рисовать.
- Non-interactive board/pieces.

### D5 — Без maxClients=4

- Как раньше: гости через отсутствие seat, не через hard cap клиентов.

### D6 — Skills

- Обновить board/schema/game/rooms: 4 tokens, strip×4, free cells.

## Risks / Trade-offs

- [До 16 pieces на доске] → OK для CSS overlay; следить за z-index.
- [4 игрока заполняют все 16 стартов] → на 4-м join пул клеток пуст на каждой стороне ровно на одну клетку — OK если учитываем occupied.
- [Одинаковый PNG у 4 слотов strip] → статусы later отличат; сейчас допустимо.

## Migration Plan

1. Server: schema + assign/leave + mocha (новые SC-PIECE-*).
2. Client: store + GamePage ×4 + strip×4; lint/typecheck.
3. Meta skills; Traceability → covered.
4. Rollback: вернуть seats v1 или убрать pieces.

## Technical prerequisites

- PNG `tourist1`…`tourist4` в client — готово.
- Explore D1–D3 / Q1–Q4 закрыты.

## Open Questions

- Нет.
