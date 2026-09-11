## Context

См. `proposal.md` (Why / Scope). Сейчас:

- Client: `LobbyPage` на mount вызывает `refreshRooms` + `setInterval(5000)`; store `game.refreshRooms` → `client.http.get('/rooms/checkers')`.
- Server: в `app.config` зарегистрирован scaffold `my_room` без `enableRealtimeListing`; built-in `LobbyRoom` не подключён.
- Клиентский контракт уже `CHECKERS_ROOM = 'checkers'` — рассинхрон с сервером блокирует join и листинг.

Пакеты: **server** + **client** (cross-package). Meta: skills после кода.

## Goals / Non-Goals

**Goals:**

- Built-in Colyseus `LobbyRoom` + `.enableRealtimeListing()` на игровой комнате — минимум кода.
- Client: один Pinia-канал подписки на lobby; убрать HTTP-poll списка.
- Политика: leave lobby перед входом в `checkers` (один игровой сокет на GamePage).
- Выровнять имя комнаты сервера на `checkers`.

**Non-Goals:**

- Своя LobbyRoom на schema / presence / чат.
- Keep-both sockets на GamePage.
- Удаление HTTP `GET /rooms/:name`.
- Отдельный JWT `onAuth` на LobbyRoom в этом change.

## Decisions

### D1: Built-in LobbyRoom, не кастомная schema-room

- **Выбор:** `defineRoom(LobbyRoom)` + сообщения клиента `rooms` / `+` / `-` (канон Colyseus 0.18).
- **Почему:** меньше изменений, чем своя schema; покрывает live list.
- **Альтернатива:** своя Room + ArraySchema — отклонена (больше кода без выгоды для текущего UI).

### D2: Leave lobby при входе в игру

- **Выбор:** в `_enterRoom` сначала `_leaveCheckersRoom` (lobby остаётся на время попытки); `unsubscribeLobby` **после успешного** connect в `checkers`; на GamePage только `checkers` room. Неуспешный enter оставляет live-list на лобби.
- **Почему:** проще lifecycle Pinia, меньше CCU на lobby, нет фоновых апдейтов списка.
- **Альтернатива:** keep both — отклонена (нет UI списка на GamePage).

### D3: Filter по `name: checkers`

- **Выбор:** `joinOrCreate('lobby', { filter: { name: CHECKERS_ROOM } })`.
- **Почему:** в листинг не попадают чужие room types, если появятся позже.

### D4: Rename `my_room` → `checkers` в том же change

- **Выбор:** единственный ключ игровой комнаты `checkers: defineRoom(MyRoom).enableRealtimeListing()`.
- **Почему:** без этого live listing и клиентский join несогласованы; отдельный change увеличил бы риск рассинхрона deploy.
- **Prerequisite / blocker:** тесты и loadtest сейчас на `--room my_room` — обновить вместе с регистрацией.

### D5: HTTP listing не удаляем

- **Выбор:** оставить built-in/HTTP путь как есть для отладки; UI его больше не poll’ит.
- **Почему:** ноль риска breaking для внешних скриптов; YAGNI на удаление.

## Точки врезки

### Server (`../happy-tourist-server`)

| Место | Что сделать |
|-------|-------------|
| `src/app.config.ts` | Импорт `LobbyRoom`; `lobby: defineRoom(LobbyRoom)`; заменить `my_room` → `checkers` + `.enableRealtimeListing()` |
| `src/rooms/MyRoom.ts` | При наличии UI-полей — `setMetadata({ title?, status? })` на create/join/leave по мере готовности статуса (минимально достаточно для появления в list; metadata — по текущему scaffold) |
| `test/MyRoom.test.ts` (и аналоги) | Room name `checkers`; добавить/расширить тест: клиент в lobby видит `+`/`rooms` при create `checkers` (SC-LOBBY-02/03) |
| `loadtest/example.ts` / npm script | `--room checkers` |

### Client (`../happy-tourist.github.io`)

| Место | Что сделать |
|-------|-------------|
| `src/stores/game.ts` | Состояние `lobbyRoom`; actions `subscribeLobby` / `unsubscribeLobby`; handlers `rooms`/`+`/`-` → `rooms[]`; `unsubscribeLobby` после успешного `_enterRoom` и из `leaveGame` / logout; `refreshRooms` HTTP — не использовать с LobbyPage (можно оставить метод unused или тонкий fallback — предпочтительно не звать с UI) |
| `src/pages/LobbyPage.vue` | `onMounted` → `subscribeLobby`; `onUnmounted` → `unsubscribeLobby`; удалить `setInterval(5000)`; loading/error через store + существующий banner |
| Play/Create/Join | Без изменений контракта навигации; enter через store: после успешного connect в `checkers` — `unsubscribeLobby` (D2); при ошибке enter live-list остаётся |

### Meta (после кода)

| Место | Что сделать |
|-------|-------------|
| `.agents/skills/client/work-with-lobby/SKILL.md` | Poll 5s → LobbyRoom subscribe / leave policy |
| `.agents/skills/server/work-with-rooms/SKILL.md` | `lobby` + `checkers` + `enableRealtimeListing` |
| Sibling AGENTS.md (по необходимости) | Одна строка про live lobby |

Слои client: page → store → `client` из boot; Colyseus I/O не размазывать по UI.

## Risks / Trade-offs

- **[BREAKING room name]** Старый клиент/loadtest с `my_room` не найдёт комнату → Mitigation: одновременный deploy client+server; обновить loadtest/tests в том же PR-наборе.
- **[Короткий gap]** Между leave lobby и join checkers список не live → приемлемо (пользователь уходит с экрана).
- **[Нет JWT на LobbyRoom]** Теоретически можно join lobby без токена, если SDK позволяет → Mitigation: экран лобби уже за `requiresAuth`; жёсткий onAuth — follow-up при необходимости.
- **[Metadata пустая]** Список может показывать только roomId/clients → Mitigation: минимальный `setMetadata` в MyRoom, если UI уже ждёт `title`/`status`.

## Migration Plan

1. Задеплоить server с `lobby` + `checkers` + realtime listing.
2. Задеплоить client с subscribe/unsubscribe (без poll).
3. Rollback: вернуть предыдущие версии обоих; HTTP poll на старом client снова работает только если server ещё отдаёт `/rooms/...` (остаётся).

## Open Questions

Нет блокирующих. Импорт `LobbyRoom` уточнить по фактическому entry `colyseus` / `@colyseus/core` в зависимостях сервера при apply (одна строка import).
