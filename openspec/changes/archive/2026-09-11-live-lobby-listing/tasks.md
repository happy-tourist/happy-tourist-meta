## 1. Server — contract (lobby + checkers)

- [x] 1.1 Прочитать `design.md` (D1–D4), `specs/lobby/rooms/spec.md` (SC-LOBBY-01…04), skill `.agents/skills/server/work-with-rooms/SKILL.md` и текущий `../happy-tourist-server/src/app.config.ts` — зафиксировать точку импорта `LobbyRoom`
- [x] 1.2 В `../happy-tourist-server/src/app.config.ts` зарегистрировать `lobby: defineRoom(LobbyRoom)` и заменить `my_room` на `checkers: defineRoom(MyRoom).enableRealtimeListing()`; проверить, что сервер стартует без ошибок конфига
- [x] 1.3 При необходимости минимальный `setMetadata` в `../happy-tourist-server/src/rooms/MyRoom.ts` для полей списка (title/status), если UI их уже показывает; иначе оставить listing без metadata
- [x] 1.4 Обновить `../happy-tourist-server/test/**` и loadtest (`--room checkers` / package script) под имя `checkers`
- [x] 1.5 Добавить mocha-тест(ы) на live listing: клиент в lobby получает снимок/`+` при create `checkers` и `-` при dispose (SC-LOBBY-02, SC-LOBBY-03); пометить сценарии ID в тесте
- [x] 1.6 В `../happy-tourist-server`: `npm test`; при падении — починить до перехода к client

## 2. Client — live lobby subscribe

- [x] 2.1 Прочитать skill `.agents/skills/client/work-with-lobby/SKILL.md`, `../happy-tourist.github.io/src/stores/game.ts`, `../happy-tourist.github.io/src/pages/LobbyPage.vue` и design D2/D3
- [x] 2.2 В `game` store добавить `subscribeLobby` / `unsubscribeLobby` (joinOrCreate `lobby` с filter `name: checkers`, handlers `rooms`/`+`/`-`, error → `error`); убедиться, что Colyseus I/O остаётся в store
- [x] 2.3 Вызвать `unsubscribeLobby` после успешного входа в `checkers` (`_enterRoom`); при leave/logout — через `leaveGame`; при ошибке enter — оставить live lobby (SC-LOBBY-05 / SC-LOBBY-01)
- [x] 2.4 В `LobbyPage`: mount → `subscribeLobby`, unmount → `unsubscribeLobby`; удалить `setInterval(5000)` и вызовы poll `refreshRooms` с страницы; loading/error через существующий banner (SC-LOBBY-01, SC-LOBBY-06, SC-LOBBY-07)
- [x] 2.5 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 3. Meta skills + ручная проверка

- [x] 3.1 Обновить `.agents/skills/client/work-with-lobby/SKILL.md` (poll → LobbyRoom; leave policy) и `.agents/skills/server/work-with-rooms/SKILL.md` (`lobby` + `checkers` + realtime listing); при необходимости одна строка в sibling AGENTS.md
- [ ] 3.2 Ручной сценарий (агент или пользователь): два клиента на lobby — create в третьем → список обновляется без HTTP poll; enter game → нет фонового lobby WS; возврат на lobby → снова live (SC-LOBBY-01…07)
- [x] 3.3 Из корня meta: `openspec validate --change live-lobby-listing` (и при готовности к merge — `/opsx-sync` / archive по отдельной просьбе)
