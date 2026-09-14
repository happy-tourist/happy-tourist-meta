## Context

См. `proposal.md` — Why. Сейчас `onLeave` в room `tourist` сразу удаляет seat; client после reload делает `joinById` (новый session). Colyseus 0.18 поддерживает `allowReconnection` / drop lifecycle. Пакеты: **server** (контракт sync + lifecycle), затем **client** (token + reconnect + presence), затем meta skills.

## Goals / Non-Goals

**Goals:**

- Consented leave vs unexpected disconnect с grace 30 с и тем же seat.
- Sync `connected` + reconnect deadline на seat для UI у всех.
- Dispose комнаты при 0 seated (даже со зрителями).
- Presence-кружки + `QCircularProgress` на офлайне; раскладки seated/spectator по спекам.
- Восстановление того же seat через сохранённый reconnection token после F5, новой вкладки и закрытия/открытия браузера (в пределах grace 30 с).

**Non-Goals:**

- Seat по userId вместо Colyseus reconnection; cross-device без общего storage.
- Блокировка cross-tab reconnect (вторая вкладка может забрать seat).
- Переоткрытие seating после start.
- Конец партии / ход / strip status chrome.
- Менять геометрию доски или assign pieces.
- Reconnect / `allowReconnection` для `LobbyRoom`; persist lobby reconnection token.

## Decisions

### D1 — Colyseus grace, не userId reclaim

- Server: на unexpected drop — `allowReconnection(client, 30)`; seat не удалять до timeout/reject.
- Consented `leave` — сразу cleanup seat (без allowReconnection).
- Alternate (ключ seat по JWT user id + `joinById`) — отвергнут: сложнее anonymous/два устройства; в стеке уже есть reconnection.

### D2 — Schema connectivity на Seat

- Добавить на seat sync-поля, например:
  - `connected: boolean` (default true при assign)
  - `reconnectUntil: number` (unix ms; `0` когда online / нет grace)
- На drop: `connected=false`, `reconnectUntil=now+30000`.
- На успешный reconnect: `connected=true`, `reconnectUntil=0`.
- Alternate (только room message «player offline») — хуже: не переживает поздний join зрителя.

### D3 — Client token storage (`localStorage`)

- После успешного enter tourist: сохранить `reconnectionToken` (+ `roomId`) в **`localStorage`** (не `sessionStorage`), чтобы переживать закрытие вкладки/браузера в пределах server grace.
- На mount Game при отсутствии живого room: сначала `client.reconnect(token)` для matching `roomId`; при неудаче — `joinById` как **fresh join**: до start — seat с фигурками при свободном месте (возможен параллельный offline ghost seat другого/своего grace); после start — только spectator (seating закрыт).
- На consented `leaveGame`: очистить token из `localStorage`.
- После failed reconnect (token протух / invalid) — очистить stale token из storage (разумный default).
- Cross-tab: общий `localStorage` — вторая вкладка MAY успешно `reconnect` и забрать seat; защита не требуется.
- Alternate (`sessionStorage`) — отвергнут после полевого теста: close Chrome / новая вкладка не восстанавливали seat.

### D4 — Empty seated → dispose

- После финального remove seat, если `seats.size === 0`: отключить оставшихся клиентов / дать комнате dispose (зрители не удерживают комнату).
- Если seats > 0 после leave/timeout — комната живёт; после start seating по-прежнему закрыт.

### D5 — Presence UI на Game

- Рендер только occupied seats; аватар = tourist PNG.
- Seated layout: self → bottom (home/north); opponents by join order → top, left, right.
- Spectator: join order → top, bottom, left, right.
- Offline: обернуть в `QCircularProgress` (`min=0`, `max=30` или ms-эквивалент; value = remaining); online — без кольца.
- Join order: стабильный порядок seats (порядок появления в sync map / явное `joinSeq` на seat при необходимости — выбрать минимально достаточное в apply).

### D6 — Точки врезки (файлы)

| Пакет | Куда |
|-------|------|
| server | schema Seat; room lifecycle drop/leave/reconnect **только tourist**; mocha SC-PIECE-07…16; LobbyRoom не трогать grace |
| client | game store: tourist token + reconnect; mirror connectivity; Game presence; leave = consented; lobby — D7 |
| meta | skills `work-with-rooms` (client/server), `work-with-lobby`, board/schema — tourist reconnect vs lobby fire-and-forget |

Colyseus I/O остаётся в Pinia game store; page только читает mirrored seats / presence.

### D7 — Lobby: без hold, без reservation-шума

- Подписка на `lobby` нужна **только** для live `rooms` / `+` / `-`. Отвал подписчика не охраняем.
- **Не** вызывать `allowReconnection` для LobbyRoom; **не** писать lobby token в `localStorage` / `sessionStorage`.
- Client: после drop/`onLeave` лобби — обнулить `lobbyRoom`; если пользователь всё ещё на экране Lobby — тихо снова `joinOrCreate('lobby', …)` (как resubscribe).
- Ошибки вида `seat reservation expired`, `FAILED_TO_RECONNECT` и прочий reconnect-шум **от lobby** не класть в user-facing `error` listing (не SC-LOBBY-07 «жёсткий fail»). SC-LOBBY-07 остаётся для явного провала **первичной** подписки / устойчивой недоступности списка.
- При желании: на lobby room-инстансе отключить SDK auto-reconnect (`reconnection.enabled = false`), чтобы SDK сам не долбил reservation.
- Alternate (тот же grace 30 с для lobby) — отвергнут: нет seats, продукт не требует.

## Risks / Trade-offs

- [F5 / reopen медленнее 30 с] → seat снят; после start — spectator; до start — новый seat при свободном месте — ожидаемо.
- [SDK auto-reconnect vs manual token после kill browser] → soft drop закрывает SDK; hard reopen — `localStorage` path; оба сходятся на server `allowReconnection`.
- [Cross-tab steal] → вторая вкладка может забрать seat через тот же token — принято (Q3).
- [Join без token во время своего grace] → fresh `joinById` + ghost offline seat («двое меня») до timeout — принято (Q2).
- [Часы клиента для countdown] → считать remaining от sync `reconnectUntil` (server time), не от локального «30» без deadline.
- [Зрители при last-seat leave] → резкий disconnect — ок по продукту.
- [SDK auto-reconnect на lobby → seat reservation expired в banner] → D7: не surface; optional disable auto-reconnect; quiet resubscribe.

## Migration Plan

1. Server: schema + drop/leave/reconnect + dispose-empty + mocha.
2. Client: token + reconnect + mirror + presence UI; lint/typecheck.
3. Meta skills; Traceability → covered где применимо.
4. Rollback: убрать allowReconnection / connectivity fields / presence chrome; leave снова immediate.

## Technical prerequisites

- Explore D1–D4, Q1–Q9 закрыты; follow-up: `localStorage` (D3 revised), Q1=после start только зритель, Q2=без token = fresh join, Q3=cross-tab OK.
- Colyseus 0.18 already in both packages — новых npm-зависимостей не требуется.
- Первая реализация apply уже на `sessionStorage` — follow-up tasks переводят persist на `localStorage` и skills.

## Open Questions

- Нет.
