## 1. Server — schema, deferred pieces, turn timer



- [x] 1.1 Прочитать `design.md`, delta `specs/game/{move,pieces,start}`, skills `server/work-with-schema`, `server/work-with-game`, `server/work-with-rooms`, `server/server-work-with-test` и текущие `MyRoomState` / `MyRoom` — зафиксировать поля и точки spawn/timer перед правками

- [x] 1.2 Добавить в schema `turnUntil`, `turnBudgetSeconds` (room) и `Seat.timeExpired` (defaults 0 / 0 / false) — проверить типами/сборкой schema

- [x] 1.3 Deferred pieces: join только в `waiting` → seat+kind без pieces; на переход в `playing` — materialize four pieces всем seats; join в `countdown`/`playing` — spectator (см. блок 7) — mocha: SC-PIECE-01/05/18/19, SC-START-02/13

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



## 4. Client — restore presence avatar (Quasar slot / sibling)



- [x] 4.1 Presence: вынести tourist `<img>` sibling поверх sibling turn/reconnect rings (как pre-timer solo img); не класть avatar только в default slot `q-circular-progress` без `show-value`; не nested progress — SC-PRESENCE-04/12

- [x] 4.2 Обновить `client/work-with-game-board` (канон sibling img + rings) и `client-align-code` Axis C: hard defect если img в slot без `show-value` или nested `q-circular-progress` прячет avatar

- [x] 4.3 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам



## 5. Client — presence row layout + affordances + bubbles



- [x] 5.1 Прочитать `design.md` D9–D11, delta `specs/game/{presence,say}`, skills `client/work-with-game-board`, `client/work-with-styles` и текущий `GamePage` presence-frame / slots

- [x] 5.2 Seated: self bottom row; opponents one top row L→R join order; spectator: all seats top row L→R; remove left/right presence columns — SC-PRESENCE-02/03

- [x] 5.3 Avatar size = strip tourist; scale outer/inner rings around avatar; reserved marker box stable when rings activate — SC-PRESENCE-11/13

- [x] 5.4 Finish badge top-left (all); ready affordance top-left (own only); say affordance top-right (own only) — SC-PRESENCE-14 / SC-SAY-07

- [x] 5.5 Say bubbles toward board (top row below avatar; self above avatar); gap so neighbor bubbles do not overlap — SC-SAY-11/12

- [x] 5.6 Обновить `work-with-game-board` / `work-with-styles` (+ AGENTS index rows if needed) под row layout / corners / bubble direction

- [x] 5.7 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам



## 6. Client — board tile chrome + full width



- [x] 6.1 Прочитать delta `specs/game/board`, `design.md` D12, skill `client/work-with-game-board` (tile chrome канон)

- [x] 6.2 `--gap: 2px`, `--radius: 2px`; пересчитать max-width формул доски под gap 2; board full content width без side presence gutters — SC-BOARD-02/03

- [x] 6.3 Обновить skill/docs канон gap/radius 2 (work-with-game-board / AGENTS при упоминании 6/12)

- [x] 6.4 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 7. Server — seating only while waiting

- [x] 7.1 Прочитать `design.md` D13, delta `specs/game/{pieces,finish,move,start}`, skills `server/work-with-rooms`, `server/work-with-game`, `server/server-work-with-test` и текущий `MyRoom.onJoin`
- [x] 7.2 `onJoin`: новый seat только при `phase === 'waiting'` и `seats.size < maxSeats`; иначе spectator; reconnect path без изменений — SC-PIECE-19 (+ countdown join spectator)
- [x] 7.3 Leave/grace в `countdown`/`playing` не открывают seating для последующих join — SC-PIECE-08/14/21, SC-FINISH-07; SC-MOVE-19 turn order unchanged
- [x] 7.4 Обновить server skills / AGENTS при упоминании mid-game seat (work-with-game / work-with-rooms)
- [x] 7.5 Из sibling `happy-tourist-server`: `npm test` — обновлённые SC-PIECE / SC-FINISH / SC-MOVE seating зелёные

