## 1. Server — four pieces per seat

- [x] 1.1 Прочитать design/skills и сверить `MyRoom` / `MyRoomState` *(базовое чтение; контракт ниже устарел относительно v1 — перечитать при 1.2)*
- [x] 1.2 **Rework** schema: seat = `touristId` + ровно 4 pieces (`side` N/E/S/W + `row`/`col`); убрать модель «один side на seat» — verify: compile / room в тестах
- [x] 1.3 **Rework** `onJoin`/`onLeave`: уникальный kind; 4 клетки из свободных стартов сторон; start на 4-м seated; leave снимает все 4 — SC-PIECE-01…08
- [x] 1.4 **Переписать** mocha SC-PIECE-01…08; Traceability pieces → covered где применимо
- [x] 1.5 `npm test` в server; починить до client

## 2. Client — board ×4 + strip ×4

- [x] 2.1 Прочитать board skills / сверить ассеты *(повторно сверить при 2.2)*
- [x] 2.2a Сохранить `_attachRoom` до `unsubscribeLobby` (race fix) — не ломать
- [x] 2.2 **Rework** `stores/game`: mirror seats с массивом/map pieces + `touristId` / `started` / `sessionId`
- [x] 2.3 **Rework** `GamePage`: все pieces всех seats на клетках; strip из 4 своих слотов N→E→S→W (тот же PNG); без статусов; non-interactive — SC-PIECE-09/10, SC-BOARD-01/05
- [x] 2.4 `npm run lint` + `npm run typecheck` в client

## 3. Meta — skills

- [x] 3.1 Обновить `work-with-game-board` / schema / game / rooms: 4 tokens/player, strip×4, free cells; verify vs specs

## Notes

- Rework с seating v1 (1 piece / strip×1) на 4 tokens/player выполнен; все задачи закрыты.
- SC-PIECE-10 = spectator без strip; нумерация в spec актуальна.
