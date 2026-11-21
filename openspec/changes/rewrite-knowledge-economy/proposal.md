## Why

Текущая экономика (шаги/просмотры, peek по иконке, таймер хода, соло-остаток, место 1/2/3) размывает цель партии. Нужна модель «один ход → открытие клетки → очки знаний на туристе → донести до финиша», с фиксированными ловушками и обрезанием последнего игрока.

## What Changes

- **BREAKING:** убрать budgets steps/peeks, peek-аффорданс, таймер хода, end-turn, push, выбор density в create, соло-остаток (∞ / 1 мин / time-expired по соло), модалку места 1/2/3 как итог.
- Один ход по карте за очередь (move **или** return); после приземления клетка открывается (задание / решётка / катапульта); задание — модалка до закрытия, затем очередь дальше.
- Очки знаний = сложность задания (1|2|3) на туристе; при финише туриста — на аватар (+N анимация); недонесённые сгорают при обрезании последнего.
- Спасение своего рядом — бесплатно и без лимита **до** хода по карте; толкания нет.
- Сид: решётки 12%, катапульты 15% от task-клеток, **взаимоисключающе** с заданием; после ловушки/угаданного — пустая клетка.
- Решётка: счётчик 3…1 (тик при приходе своего хода); при приходе с 1 — снять и сразу ходить; all-jail → старт и ход дальше.
- Предпоследний довёл всех туристов → партия кончается и для последнего; модалка = число донесённых знаний (без рейтинга). `maxSeats === 1` — без правила предпоследнего.

## Scope

- **Capability ID:** `lobby/rooms`, `game/board`, `game/move`, `game/presence`, `game/finish` (по касанию `game/pieces`, `game/start` — только если требования меняются из-за экономики/таймера)
- **Server:** room `tourist` — сид ловушек, очередь хода без deadline, open-on-land / fling-цепочка, knowledge на piece/seat, grille timer, all-jail → advance, penultimate cut, убрать push и peeks/steps budgets.
- **Client:** Lobby create без density; Game — knowledge HUD, счётчик решётки, focus вместо end-turn, без peek-иконки / push / turn ring / solo leftover UX / place-badge / «Вы N-й!»; одна модалка знаний (`delivered`) при полном финише и обрезании; локальный gate после своего move|return до смены хода.

## Out of scope

- Контент packs/maps (редактор, модерация), auth, leave confirm, say whitelist.
- Новый рейтинг/лидерборд между игроками (только показ своих донесённых знаний).
- Смена геометрии карт / `touristsPerPlayer` / `maxSeats` выбора в lobby.
- Визуальный редизайн тайлов сверх необходимых индикаторов (knowledge / grille counter / убрать устаревшие chrome).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: убрать выбор grille/catapult density; create без density options; сид фиксирован на сервере.
- `game/board`: клетка — ровно один тип (задание | решётка | катапульта); open-on-land вместо peek-иконки; после wrong — открытая клетка с числом знаний; после clear ловушки/correct — пустая клетка; счётчик решётки UI.
- `game/move`: один ход (move|return); без steps/peeks/таймера/end-turn/push; rescue бесплатно до хода; open-on-land и после fling; grille timer 3; all-jail → старт + advance; без соло-остатка ∞.
- `game/presence`: knowledge у аватара вместо steps/peeks; focus на месте end-turn; убрать turn deadline ring / solo unlimited modal / end-turn.
- `game/finish`: transfer knowledge турист→аватар; модалка знаний вместо place 1/2/3; penultimate full finish обрезает последнего (кроме `maxSeats === 1`).

## Impact

- Контракт room `tourist`: private `budgets` steps/peeks → private `knowledge` `{ delivered, pieces }` (own only); open-on-land auto-opens shared peek session (`peekPlace`/`peekSubmit`/`peekClose`; client `peek` reject); `push`/`endTurn` retired; create без density (server ignores if sent); public `Piece.remainingTurns`; exclusive trap cells (delta REMOVED density/overlap seed in `game/move`); без `turnUntil` / solo infinite; `finishPlace=1` как finished/cut marker **без** place UX (нет badge «N» / модалки места; только knowledge modal).
- Client + server тесты: переписать сценарии move/board/presence/finish/lobby под новую экономику; удалить/адаптировать push и solo leftover coverage.

## References

- Explore-решения D1–D17 (knowledge economy rewrite), чат 2026-11-21.
- Main specs: `openspec/specs/lobby/rooms`, `game/board`, `game/move`, `game/presence`, `game/finish`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
