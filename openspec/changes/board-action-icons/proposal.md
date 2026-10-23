## Why

На Game affordances рисовались Material `q-icon` в цветных кругах — нужен единый illustrated стиль. После подключения PNG иконки увеличены (~×2) и стоят на окружности presence; hit перекрывает голову туриста, say-picker обрезается слева, focus на 9 o’clock мешает HUD, а finish/catapult→center (в т.ч. клик по клетке финиша) выглядит как мгновенное исчезновение. Всё это — один change `board-action-icons`.

## What Changes

**Уже применено:**
- Illustrated PNG вместо Material для peek / rescue / push / return / flag / focus / end-turn / say; без цветного circular chrome.
- ~×2 глифы/hit; presence hit-centers на окружности (say 12 / focus 9 / end-turn 3 на момент apply); flag без scale.
- `peekClose.reason` (`wrong` | `timeout`); match-status flash 10 с («Правильно!» / «Ошибка!» / «Не успел!»).

**Осталось:**
- Поднять board/strip affordances (peek / rescue / push / return): hit **не перекрывает** фигурку.
- **Focus** перенести **под счётчик peeks** в budgets у своего аватара; на окружности маркера остаются **say (12)** и **end-turn (3)** — слева (9) ничего.
- **Say-picker** (пресеты «Всем привет» / «Удачи») сместить **вправо**, чтобы левый край не обрезался HUD/overflow.
- Finish travel (ход / push / catapult→center, в т.ч. клик по center): ~550 ms slide + ~300 ms fade; устранить race drop без travel.

## Scope

- **Capabilities:** `game/board`, `game/move`, `game/finish`, `game/presence`, `game/say`
- **Пакеты:** client (GamePage chrome / finish anim / status flash / say-picker); server — `peekClose.reason` (уже applied)
- **Страница:** Game

## Out of scope

- Перерисовка PNG; budgets HUD icons (walk/visibility Material) / ready; peek modal chrome
- Удлинение catapult **overlay** (~1500 ms)
- Deploy pipeline

## Capabilities

### New Capabilities

(нет)

### Modified Capabilities

- `game/board`: illustrated peek; ×2; clearance над спрайтом; `peekClose.reason`; catapult→center finish timing / no drop without travel
- `game/move`: illustrated rescue/push; ×2; clearance; finish travel parity
- `game/finish`: illustrated return/flag; return над спрайтом; enter-center / catapult→center ~550/300 без race-drop
- `game/presence`: illustrated focus/end-turn; focus **under peeks budget**; say 12 + end-turn 3 on circumference; peek outcome flash
- `game/say`: illustrated say at 12; say-picker shifted right (left edge fully visible)

## Impact

- Client: GamePage affordances, budgets stack (focus), say-picker position, finish/catapult timers, App status flash, i18n.
- Server: `peekClose.reason` (already shipped).
- Skills notes: board / focus / finish / presence / say / messages.

## References

- Explore follow-up: clearance; focus under peeks; say-picker → right; finish travel + race; **одна** папка `board-action-icons`
- Sibling AGENTS + `work-with-game-board`, `work-with-styles/board.md`
