## 1. Server — turn + move contract

- [x] 1.1 Прочитать `design.md` (D1–D4), delta `specs/game/move/spec.md`, skills `work-with-game` / `work-with-schema` / `work-with-messages` / `server-work-with-test`; сверить `MyRoom.ts` / `MyRoomState.ts`
- [x] 1.2 Добавить в schema sync `currentTurnSessionId` (string); room-private `turnOrder: string[]`; на первом seated выставить ход, на append/remove обновлять очередь по design D1/D4 — verify: compile + join отражает ход в state
- [x] 1.3 Pure rules module (playable cells = start+task+center как client LAYOUT; Chebyshev 1; occupancy включая свои): validate/apply `{ side, row, col }` — verify: unit/mocha на accept/reject без Room I/O где удобно
- [x] 1.4 `onMessage('move')` в `MyRoom`: только current-turn seated; apply → обновить piece `row`/`col` → advance turn; reject без изменения state — SC-MOVE-04…10
- [x] 1.5 Mocha SC-MOVE-01…03, 16–17 (очередь, соло, leave advances, offline waits) + SC-MOVE-04…10; обновить Traceability в delta `game/move` на covered где применимо
- [x] 1.6 `npm test` в `../happy-tourist-server`; починить до перехода к client

## 2. Client — store + UX хода

- [x] 2.1 Прочитать `design.md` (D2/D5/D6), skills `work-with-game-board` / `work-with-stores` / `colyseus-client` / `work-with-rooms`; сверить `stores/game.ts` и `GamePage.vue`
- [x] 2.2 Mirror `currentTurnSessionId` в game store; `send('move', { side, row, col })` только из store; getter «мой ход» — verify: typecheck / нет `room.send` из page в обход store
- [x] 2.3 Game UI: выбор своей фигурки (поле + strip), белая рамка выбранного тайла, красные легальные цели (локальный hint); только при своём ходе; перещёлкивание до submit; зритель/не-ход без интерактива — SC-MOVE-11…13, SC-BOARD-05/06
- [x] 2.4 Плавная анимация переезда фигурки у всех клиентов (~200–300ms, transform); пока своя анимация — игнор кликов — SC-MOVE-14…15
- [x] 2.5 `npm run lint` + `npm run typecheck` в `../happy-tourist.github.io`

## 3. Meta — skills

- [x] 3.1 Обновить meta skills (`work-with-game`, `work-with-messages`, `work-with-schema`, `work-with-game-board`, при необходимости stores/colyseus-client): message `move`, `currentTurnSessionId`, interactive hints, animation; сверить с proposal/design/specs
