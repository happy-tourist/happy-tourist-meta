## 1. Server — seating + schema

- [x] 1.1 Прочитать `design.md` (D1–D3, D5), delta `specs/game/pieces/spec.md` (SC-PIECE-01…07), `.agents/skills/server/work-with-schema/SKILL.md`, `.agents/skills/server/work-with-rooms/SKILL.md`, `.agents/skills/server/work-with-game/SKILL.md`; сверить `../happy-tourist-server/src/rooms/MyRoom.ts` и `schema/MyRoomState.ts`
- [x] 1.2 Заменить scaffold `MyRoomState` на product sync: `started` + map seats (`touristId`, `side`, `row`, `col`); wired `setState` в room — verify: schema компилируется / room поднимается в тестах
- [x] 1.3 В `MyRoom.onJoin` / `onLeave` реализовать назначение из оставшихся пулов, старт на 4-й seat, leave до/после start (design D2–D3); без `maxClients=4` — SC-PIECE-01…07
- [x] 1.4 Добавить mocha-тесты с ID `SC-PIECE-01`…`SC-PIECE-07` в `../happy-tourist-server/test/`; обновить Traceability в delta `game/pieces` на covered где применимо
- [x] 1.5 В `../happy-tourist-server`: `npm test`; при падении — починить до client

## 2. Client — фигурки + strip

- [x] 2.1 Прочитать `design.md` (D4), delta `specs/game/pieces/spec.md` (SC-PIECE-08…09), delta `specs/game/board/spec.md` (SC-BOARD-01/05), `.agents/skills/client/work-with-game-board/SKILL.md`, `.agents/skills/client/work-with-stores/SKILL.md`, `.agents/skills/client/work-with-rooms/SKILL.md`; сверить `GamePage.vue` / `stores/game.ts` и наличие `src/assets/tourists/tourist{1-4}.png`
- [x] 2.2 В `stores/game` зеркалировать seats / `started` / свой `sessionId` из room state (`onStateChange`) — verify: store отражает sync после join
- [x] 2.3 На Game отрисовать фигурки на стартовых клетках по seats + strip «мой турист» только для seated (ассеты `touristN.png`); доска/фигурки non-interactive — SC-PIECE-08/09, SC-BOARD-01/05
- [x] 2.4 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 3. Meta — skills (кратко)

- [x] 3.1 Обновить `.agents/skills/client/work-with-game-board/SKILL.md` и при необходимости `.agents/skills/server/work-with-schema/SKILL.md` / `work-with-game/SKILL.md`: sync seats, фигурки, без ходов (design D6); verify: skills не противоречат specs
