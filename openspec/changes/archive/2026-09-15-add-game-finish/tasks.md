## 1. Server — schema и finish на move

- [x] 1.1 Прочитать `design.md`, delta `specs/game/finish`, `specs/game/move`, skills `server/work-with-schema`, `server/work-with-game`, `server/work-with-rooms` и текущие `MyRoomState` / `MyRoom` / `touristMove` — зафиксировать поля `finished` / `finishPlace` / `nextFinishPlace` перед правками
- [x] 1.2 Добавить в schema `Piece.finished`, `Seat.finishPlace`, `MyRoomState.nextFinishPlace` (defaults: false / 0 / 1) — проверить типами/сборкой schema
- [x] 1.3 В apply move: occupancy только `!finished`; ход на center → `finished=true`, клетка свободна; при 4 finished на seat выдать `finishPlace` из `nextFinishPlace++` — mocha: SC-FINISH-01…04, SC-MOVE-08
- [x] 1.4 `advanceTurn` / current turn: skip seats с `finishPlace > 0`; finished offline не держит turn; reject move от finished seat / finished piece — mocha: SC-MOVE-21…23, SC-FINISH-06 (move reject)
- [x] 1.5 Finished seat остаётся в `seats` (ёмкость); dispose только при `seats.size === 0`; grace для finished как у active — mocha: SC-FINISH-07/08/11, SC-PIECE-21
- [x] 1.6 Из sibling `happy-tourist-server`: `npm test` — затронутые SC-FINISH / SC-MOVE / SC-PIECE зелёные

## 2. Client — store и board/strip finish

- [x] 2.1 Прочитать delta `specs/game/finish`, `specs/game/pieces`, `specs/game/move`, skills `client/work-with-stores`, `client/work-with-game-board`, `client/colyseus-client` и текущие `game` store / `GamePage`
- [x] 2.2 Зеркалить `piece.finished`, `seat.finishPlace` в Pinia `game`; хелперы для unfinished board pieces и finished strip slots
- [x] 2.3 Board: не рендерить finished на поле; после хода на center — короткий disappear (без улёта к strip); strip: inactive + иконка finish справа сверху — SC-FINISH-01/09/10, SC-PIECE-09, SC-MOVE-24
- [x] 2.4 Из sibling `happy-tourist.github.io`: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 3. Client — place modal, presence, leave

- [x] 3.1 Прочитать delta `specs/game/finish`, `specs/game/presence`, `specs/game/leave`, skills `client/work-with-pages`, `client/work-with-localization`, `client/work-with-game-board`
- [x] 3.2 i18n: модалка «Вы N-й…» (1/2/3…); a11y для finish/place badges
- [x] 3.3 При переходе собственного `finishPlace` 0→N показать `q-dialog`; после закрытия остаться в room; presence badge с номером места на finished seats — SC-FINISH-03…05, SC-PRESENCE-06/07
- [x] 3.4 `needsLeaveConfirm = seated ∧ playing ∧ !finishPlace`; finished → сразу consented leave — SC-LEAVE-02/05/06
- [x] 3.5 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам
