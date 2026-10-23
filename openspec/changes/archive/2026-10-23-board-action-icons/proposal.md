## Why

На Game affordances рисовались Material `q-icon` в цветных кругах — нужен единый illustrated стиль. После подключения PNG иконки увеличены и стоят на окружности presence; hit перекрывает голову туриста, say-picker обрезается слева, focus на 9 o’clock мешает HUD, а finish/catapult→center (в т.ч. клик по клетке финиша) выглядит как мгновенное исчезновение. Всё это — один change `board-action-icons`.

## What Changes

**Применено (итог):**
- Illustrated PNG вместо Material для peek / rescue / push / return / flag / focus / end-turn / say; без цветного circular chrome; PNG подрезаны по opaque bbox.
- Glyph/hit **36×36** (glyph fills hit) для восьми слотов; **flag** размер без изменений.
- Board/strip affordances (peek / rescue / push / return): hit целиком **над** спрайтом (~8 px gap).
- Presence: **say @12** и **end-turn @3** на окружности; **focus под end-turn** на угол квадрата аватара (`calc(50% + 36px)`); не в budgets и не @9.
- Say-picker смещён **вправо** — левый край пресетов не обрезается HUD/overflow.
- Finish travel (ход / push / catapult→center, в т.ч. клик по center): ~550 ms slide + ~300 ms fade; без race drop без travel.
- `peekClose.reason` (`wrong` | `timeout`); match-status flash 10 с («Правильно!» / «Ошибка!» / «Не успел!»).

## Scope

- **Capabilities:** `game/board`, `game/move`, `game/finish`, `game/presence`, `game/say`
- **Пакеты:** client (GamePage chrome / finish anim / status flash / say-picker / assets); server — `peekClose.reason`
- **Страница:** Game

## Out of scope

- Перерисовка PNG; budgets HUD icons (walk/visibility Material) / ready; peek modal chrome
- Удлинение catapult **overlay** (~1500 ms)
- Deploy pipeline

## Capabilities

### New Capabilities

(нет)

### Modified Capabilities

- `game/board`: illustrated peek; 36×36; clearance над спрайтом; `peekClose.reason`; catapult→center finish timing / no drop without travel
- `game/move`: illustrated rescue/push; 36×36; clearance; finish travel parity
- `game/finish`: illustrated return/flag; return над спрайтом; enter-center / catapult→center ~550/300 без race-drop
- `game/presence`: illustrated focus/end-turn; focus **under end-turn** at avatar square corner; say 12 + end-turn 3; peek outcome flash
- `game/say`: illustrated say at 12; say-picker shifted right (left edge fully visible)

## Impact

- Client: GamePage affordances, focus under end-turn, say-picker position, finish/catapult timers, App status flash, i18n, cropped board PNGs.
- Server: `peekClose.reason`.
- Skills notes: board / focus / finish / presence / say / messages.

## References

- Explore follow-up → wire → scale/flash → clearance / focus / picker / finish travel → crop + focus under end-turn; **одна** папка `board-action-icons`
- Sibling AGENTS + `work-with-game-board`, `work-with-styles/board.md`
