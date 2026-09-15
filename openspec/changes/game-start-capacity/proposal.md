## Why

Сейчас комната `tourist` всегда ждёт ровно четырёх seated-игроков и сразу открывает ходы; нет выбора размера партии, явного старта и отсчёта. Нужен контролируемый старт: создатель задаёт 2/3/4 игрока, стол заполняется или игроки подтверждают готовность, затем общий countdown и только после него — ходы.

## What Changes

- При создании игры — выбор числа игроков (2 / 3 / 4, по умолчанию 2).
- Ёмкость комнаты = выбранное число seats; сверх лимита — гости (как нынешние spectators).
- Автостарт countdown при заполнении seats; при недоборе (≥2 seated) — one-shot «Готов начать» у каждого, бабл через say; когда все текущие seated готовы — тот же countdown.
- Полноэкранный отсчёт 5…1 для всех клиентов в комнате; ходы только после его завершения (**BREAKING** относительно текущего «ходы с первого seated»).
- Лобби показывает `занято/maxSeats` (например `2/4`); кнопка «Играть» убирается из UI (join по списку остаётся).
- Seats остаются открытыми, пока есть свободный слот — в том числе после начала партии (mid-game seat на свободные стартовые клетки).

## Scope

- **Пакеты:** client + server (согласованный контракт room `tourist`).
- **Capability ID:**
  - `game/start` (новый) — фазы waiting / countdown / playing, ready, countdown, gate ходов;
  - `lobby/rooms` — create с ёмкостью, отображение seats/maxSeats, убрать «Играть»;
  - `game/pieces` — порог seats = maxSeats (не жёсткие 4), seats открыты при свободном слоте в любой фазе;
  - `game/say` — whitelist preset «готов начать»;
  - `game/move` — ходы только в фазе playing (после countdown).
- **Client UX:** Lobby (create modal, список), Game (ready у say, overlay countdown, lock поля до playing).
- **Server:** room `tourist` — options create, synced ёмкость/фаза/ready, message ready, авто/ready-triggered countdown, reject move до playing.

## Out of scope

- Конец партии / победа / финальный экран.
- Кнопка «Играть» / matchmaking `joinOrCreate` (удаление из UI; восстановление later).
- Добровольный вход зрителем при свободном seat.
- Сброс ready при leave; отмена countdown при disconnect/leave.
- Смена maxSeats после create.

## Capabilities

### New Capabilities

- `game/start`: фазы старта партии (waiting → countdown → playing), one-shot ready при недоборе, автостарт при полном столе, общий countdown 5…1, блокировка ходов до playing.

### Modified Capabilities

- `lobby/rooms`: create с выбором maxSeats; в листинге `seats/maxSeats`; без действия «Играть».
- `game/pieces`: лимит seats = maxSeats (2|3|4); seat при любом свободном слоте (включая mid-game); гости только при полном столе.
- `game/say`: новый whitelist preset готовности.
- `game/move`: accept move только после перехода в playing.

## Impact

- **BREAKING:** контракт старта и хода в `tourist` — больше не «четвёртый seat = started и ходы сразу»; client и server меняются вместе.
- Лобби metadata / отображение ёмкости; Game UX старта; schema/messages room.
- Skills `work-with-game`, lobby, game-board, messages — обновить после реализации (вне runtime apply, через check-changes / align).
- Тесты server mocha (pieces/start/move/say); client lint/typecheck.

## References

- Explore-решения: D1–D6, Q1–Q5, M1–M2 (этот чат).
- Канон: `docs/projects-map.md`, `openspec/specs/lobby/rooms`, `game/pieces`, `game/say`, `game/move`, `game/presence`.
- Sibling: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
