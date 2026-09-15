## 1. Server — schema и ёмкость

- [x] 1.1 Прочитать `design.md`, delta `specs/game/start`, `specs/game/pieces`, skills `server/work-with-schema`, `server/work-with-game`, `server/work-with-rooms` и текущие `MyRoomState` / `MyRoom` — зафиксировать поля phase/maxSeats/ready/countdown перед правками
- [x] 1.2 Добавить в schema `maxSeats`, `phase` (`waiting`|`countdown`|`playing`), поле countdown, `Seat.ready`; в `onCreate` читать `maxSeats` из options (2|3|4, default 2) и выставлять metadata `maxSeats`/`seats`/`status` — проверить типами/сборкой schema
- [x] 1.3 Заменить hardcode seats≥4 / permanent lock: seat пока `seats.size < maxSeats` в любой фазе; spectator при полном столе; metadata seats обновлять на join/leave — mocha: SC-PIECE-05/06/19 (или эквивалент describe)

## 2. Server — start / ready / move gate

- [x] 2.1 Прочитать delta `specs/game/start`, `specs/game/say`, `specs/game/move` и skills `server/work-with-messages`, `server/work-with-game`
- [x] 2.2 Реализовать авто-countdown при заполнении maxSeats и all-ready (≥2 seated, все ready); server timer → phase `countdown` → `playing`; leave/drop не отменяет countdown — mocha: SC-START-01…03, SC-START-11/12
- [x] 2.3 `onMessage('ready')`: one-shot, reject для 1 seated / spectator / уже ready / не waiting; выставить ready; broadcast say preset readiness; не сбрасывать ready остальных при leave — mocha: SC-START-04…07, SC-SAY-13
- [x] 2.4 Гейтить `move` пока phase ≠ `playing`; mid-game seat append в turnOrder — mocha: SC-MOVE-18/19; whitelist say + preset `ready`
- [x] 2.5 Из sibling `happy-tourist-server`: `npm test` — все затронутые SC зелёные

## 3. Client — lobby

- [x] 3.1 Прочитать delta `specs/lobby/rooms`, skills `client/work-with-lobby`, `client/work-with-forms`, `client/work-with-localization` и `LobbyPage` / listing metadata
- [x] 3.2 Modal create: radio 2/3/4 (default 2) → `createGame({ maxSeats })`; убрать кнопку «Играть»; в ряду комнаты показывать `seats/maxSeats` из metadata — i18n ключи добавлены
- [x] 3.3 Из sibling `happy-tourist.github.io`: `npm run lint` и `npm run typecheck` после правок lobby

## 4. Client — game store и UI старта

- [x] 4.1 Прочитать delta `specs/game/start`, `specs/game/move`, skills `client/work-with-stores`, `client/work-with-game-board`, `client/colyseus-client`, `client/work-with-rooms`
- [x] 4.2 В `game` store зеркалить phase/maxSeats/ready/countdown; `sendReady()`; status/playing из phase; whitelist say + i18n readiness
- [x] 4.3 GamePage: fullscreen overlay countdown для всех; кнопка «Готов начать» у own say при eligible; lock move chrome/submit до `playing`
- [x] 4.4 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам
