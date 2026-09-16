## 1. Server — budgets, removed tiles, move without advance

- [x] 1.1 Прочитать `design.md`, delta `specs/game/{move,board}`, skills `server/work-with-schema`, `server/work-with-messages`, `server/work-with-game`, `server/server-work-with-test` и текущие `MyRoom` / `MyRoomState` / `touristMove` — зафиксировать точки врезки
- [x] 1.2 Schema: sync removed task cells (keys `r,c`); room-private budgets + reward bag; на `playing` преген 28/14/6 — проверить типами schema
- [x] 1.3 Messages `peek`, `peekAnswer`, `endTurn`; `budgets` private send на grant/change/reconnect; `move` тратит step и **не** вызывает advanceTurn — mocha: SC-MOVE-33…36, SC-MOVE-43/44
- [x] 1.4 Peek flow + remove tile + Correct/Wrong; one peek/turn multi; timeout force wrong — mocha: SC-BOARD-07…10, SC-MOVE-39/42
- [x] 1.5 End-turn / auto-end when no actions; grant +1/+1 next seat; solo infinite + no end-turn — mocha: SC-MOVE-37/38/40/41/45
- [x] 1.6 Playable includes removed `*`; helpers legal move/peek — unit/mocha green
- [x] 1.7 Из sibling `happy-tourist-server`: `npm test` — новые/обновлённые SC-MOVE / SC-BOARD зелёные

## 2. Client — store contract

- [x] 2.1 Прочитать delta `specs/game/{move,board,presence}`, `design.md`, skills `client/work-with-stores`, `client/colyseus-client`, `client/work-with-rooms`
- [x] 2.2 Pinia `game`: mirror removed tiles; listen `budgets`; `sendPeek` / `sendPeekAnswer` / `sendEndTurn`; stop assuming move advances turn locally
- [x] 2.3 Из sibling `happy-tourist.github.io`: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 3. Client — Game UX

- [x] 3.1 Прочитать delta `specs/game/{presence,board}`, skills `client/work-with-game-board`, `client/work-with-pages`, `client/work-with-localization`, `client/work-with-styles`
- [x] 3.2 Presence: свои steps/peeks (∞ в соло) + «Завершить ход» только на multi своем ходу; +N anim — SC-PRESENCE-15…18
- [x] 3.3 Board: дыры для removed `*`; eye + модалка «Правильно»/«Неправильно»; solo unlimited modal — SC-BOARD-11…13, SC-PRESENCE-19
- [x] 3.4 i18n ключи модалок/кнопки/∞
- [x] 3.5 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 4. Meta — skills / AGENTS

- [x] 4.1 Обновить `server/work-with-game`, `server/work-with-messages`, `server/work-with-schema` под budgets / peek / endTurn / removed tiles
- [x] 4.2 Обновить `client/work-with-game-board`, `client/colyseus-client` (или stores) под counters / peek / end-turn
- [x] 4.3 При необходимости — краткие строки в sibling AGENTS Business Entities (без дублирования specs)
