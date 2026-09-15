## 1. Server — schema, deferred pieces, turn timer

- [x] 1.1 Прочитать `design.md`, delta `specs/game/{move,pieces,start}`, skills `server/work-with-schema`, `server/work-with-game`, `server/work-with-rooms`, `server/server-work-with-test` и текущие `MyRoomState` / `MyRoom` — зафиксировать поля и точки spawn/timer перед правками
- [x] 1.2 Добавить в schema `turnUntil`, `turnBudgetSeconds` (room) и `Seat.timeExpired` (defaults 0 / 0 / false) — проверить типами/сборкой schema
- [x] 1.3 Deferred pieces: join в `waiting`/`countdown` → seat+kind без pieces; на переход в `playing` — materialize four pieces всем seats без pieces; join в `playing` — сразу pieces — mocha: SC-PIECE-01/05/18/19, SC-START-02/13
- [x] 1.4 Turn deadline в `playing`: 60s при ≥2 eligible; schedule/clear на assign current; timeout → `advanceTurn` без хода фигуры; тик во время reconnect grace; до `playing` таймер не ставится — mocha: SC-MOVE-25…28
- [x] 1.5 Solo: при ровно одном eligible — `turnBudgetSeconds=300`, свежие 5:00; finish всех pieces → clear deadline; expiry → `timeExpired`, reject move, room живёт — mocha: SC-MOVE-29/30 (+ reject path для SC-MOVE-31 server half)
- [x] 1.6 Из sibling `happy-tourist-server`: `npm test` — новые/обновлённые SC-MOVE / SC-PIECE / SC-START зелёные (бюджеты в тестах ускорить через export констант / короткий timeout)

## 2. Client — store mirror и presence rings

- [x] 2.1 Прочитать delta `specs/game/{presence,move}`, `design.md` D5–D7, skills `client/work-with-stores`, `client/work-with-game-board`, `client/colyseus-client` и текущие `game` store / `GamePage` presence
- [x] 2.2 Зеркалить `turnUntil`, `turnBudgetSeconds`, `timeExpired` в Pinia `game`; хелперы remaining turn / isSoloBudget / isTimeExpired
- [x] 2.3 Presence: убрать static `--turn` outline; reserved outer chrome; outer turn ring (blue 60s / red solo); inner reconnect warning; оба видны вместе — SC-PRESENCE-04/08…11
- [x] 2.4 Из sibling `happy-tourist.github.io`: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 3. Client — solo modal, lock UX, leave, empty pre-start board

- [x] 3.1 Прочитать delta `specs/game/{move,leave,pieces}`, skills `client/work-with-pages`, `client/work-with-localization`, `client/work-with-game-board`
- [x] 3.2 i18n: модалка timeout (смысл «не успели довести туристов») + OK; watch `timeExpired` → показать только своему seat; clear selection/hints; блок board/strip move — SC-MOVE-31/32
- [x] 3.3 До `playing`: не показывать pieces на доске / moveable strip для seats без pieces; после materialize — как сейчас — SC-PIECE-17
- [x] 3.4 `needsLeaveConfirm = seated ∧ playing ∧ !finishPlace ∧ !timeExpired` — SC-LEAVE-05/07
- [x] 3.5 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам
