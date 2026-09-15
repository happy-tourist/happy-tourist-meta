## Why

Старт партии уже есть (`waiting` → `countdown` → `playing`), но нет цели и завершения: туристы могут стоять в жёлтом центре без эффекта, победитель не определяется, finished-игрок не отличается от активного. Нужен канонический финиш: провести всех четырёх туристов в центр, зафиксировать место, оставить игрока за столом без ходов и закрыть комнату только когда все seats вышли.

## What Changes

- Вход фишки на любую клетку центра → фишка завершена: исчезает с поля (короткое исчезновение), клетка сразу свободна для других.
- Личная полоса: прошедший слот неактивен + иконка финиша; после 4/4 — модалка с местом (1-й, 2-й, …) и значок места на presence-аватарке.
- Finished-seat остаётся игроком (seat + strip + say + presence), без ходов; очередь хода всегда пропускает finished; leave без confirm.
- Finished-seat занимает слот `maxSeats` до consented leave / grace timeout; mid-game join с нуля только при свободном слоте (как сейчас по ёмкости).
- Комната не авто-kick при «все finished»; dispose когда seats = 0 (гости не держат room).

## Scope

- **Пакеты:** client + server (согласованный контракт room `tourist`).
- **Capability ID:**
  - `game/finish` (новый) — финиш фишки/игрока, места, модалка, finished-seat семантика, конец room только по выходу seats;
  - `game/move` — landing на center завершает фишку; finished не ходит; turn skip finished;
  - `game/pieces` — strip chrome для finished-слотов; finished занимает seat; mid-game seat только при свободном слоте;
  - `game/leave` — без confirm для finished-seat (и по-прежнему для spectator / pre-playing);
  - `game/presence` — значок места на маркере полностью финишировавшего игрока.
- **Client UX:** Game (доска, strip, presence, модалка места, leave).
- **Server:** room `tourist` — synced finished piece/seat/place, apply finish на move в center, turn skip, dispose без авто-kick по финишу.

## Out of scope

- Отдельная фаза room `finished` / авто-dispose при полном финише всех seats.
- Анимация «улёт» фишки к strip (только исчезновение на клетке).
- Свободный текст чата; новые say-пресеты под финиш.
- Таблица рекордов / история партий / награды вне комнаты.
- Запрет mid-game join при свободном слоте (кроме занятости finished-seats).
- Смена правил центра (нужно пройти все 4 клетки / отдельное действие «зайти»).
- Lobby metadata `status: finished` как обязательный продукт этого change (может остаться `playing`, пока есть seats).

## Capabilities

### New Capabilities

- `game/finish`: завершение фишки при входе в центр; полный финиш игрока с местом; finished-seat (say без move); UI strip/модалка/place; room живёт до выхода всех seats.

### Modified Capabilities

- `game/move`: ход на center-клетку завершает фишку; отказ/skip хода для finished; turn order пропускает finished.
- `game/pieces`: статусы слотов strip (finished/inactive); finished-seat держит ёмкость до leave.
- `game/leave`: confirm не требуется для finished-seat.
- `game/presence`: отображение места на presence-маркере финишёра.

## Impact

- **Server:** schema seat/piece (finished + place), логика move→finish, `advanceTurn` skip, тесты mocha.
- **Client:** Game board/strip/presence/модалка/i18n; leave UX; зеркало synced state в game store.
- **Контракт:** room name `tourist` без новых HTTP routes; поведение через synced state + существующий `move` (side-effect finish).
- **Docs/skills:** канон в OpenSpec meta; runtime в siblings per `docs/projects-map.md`.

## References

- Explore-решения D1–D10 (finish center, place order, finished-seat, no auto-kick, seat occupancy).
- Main specs: `openspec/specs/game/{start,move,pieces,leave,presence,board}/spec.md`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
- Карта путей: `docs/projects-map.md`.
