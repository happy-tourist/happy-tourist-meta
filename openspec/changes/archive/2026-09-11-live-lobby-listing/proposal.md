## Why

На странице лобби список комнат обновляется периодическим HTTP-опросом, хотя продукт — realtime multiplayer на Colyseus. Пользователь ожидает живое обновление списка без «пульса» запросов; сейчас сокет появляется только после входа в партию.

## What Changes

- Список доступных игровых комнат на лобби становится live через Colyseus Lobby (WebSocket), без периодического HTTP-poll.
- Игровая комната публикуется в realtime-листинг matchmaker’а; клиент подписан на лобби только пока открыт экран списка.
- При входе в партию подписка на лобби закрывается; при возврате на лобби — открывается снова.
- **BREAKING** (client↔server): каноническое имя игровой комнаты выравнивается на `checkers` на сервере (клиент уже использует это имя).

## Capabilities

### New Capabilities

- `lobby/rooms`: live-список игровых комнат на лобби (подписка, обновления, отписка при уходе в партию / с экрана), согласование room name `checkers` для листинга и join/create.

### Modified Capabilities

- (нет — main specs пока пусты)

## Scope

- **Capability ID:** `lobby/rooms`
- **Пакеты:** client (`happy-tourist.github.io`) и server (`happy-tourist-server`)
- **UX:** Lobby (live list, Play / Create / Join); Game — без фонового лобби-сокета
- **Контракт:** room name `lobby` (built-in listing) + `checkers` (игра); filter лобби по `name: checkers`; metadata листинга для UI статуса/заголовка при необходимости
- Auth: лобби доступно авторизованным (включая гостя), как сейчас экран Lobby; отдельный JWT-gate на LobbyRoom не вводится в этом change

## Out of scope

- Presence «кто сейчас в лобби», чат, приглашения
- Список столов / spectate с экрана партии (keep-both sockets)
- Удаление HTTP `GET /rooms/:roomName` (может остаться для отладки)
- Правила шашек, board sync, message `move`
- Кастомный LobbyRoom на `@colyseus/schema` вместо built-in

## Impact

- Client: экран лобби перестаёт poll’ить HTTP для списка; появляется кратковременный WebSocket к `lobby`.
- Server: регистрация LobbyRoom и realtime listing для `checkers`; переименование регистрации игровой комнаты с текущего scaffold-имени.
- Skills / docs meta: канон lobby (poll → live) после реализации.
- Deploy: client GitHub Pages + server VPS должны выкатиться согласованно из‑за **BREAKING** имени комнаты.

## References

- Explore: live lobby через built-in LobbyRoom + leave при входе в игру
- [Colyseus Lobby Room](https://docs.colyseus.io/matchmaker/lobby)
- Sibling: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`
- Meta skills: `.agents/skills/client/work-with-lobby`, `.agents/skills/server/work-with-rooms`
