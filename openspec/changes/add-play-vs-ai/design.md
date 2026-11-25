## Context

См. `proposal.md` — Why. Сейчас `tourist` умеет соло (`maxSeats=1`) и multi с людьми; seats ключуются `client.sessionId`; listing через Colyseus Lobby + metadata; ход/peek авторитетны в `MyRoom` (`resolveOpenPeek`, `touristMove.ts`). Отдельных ботов нет.

## Goals / Non-Goals

**Goals:**

- Create: opponent mode people|AI; AI → synthetic bot seats; unlist; no extra human join/spectator
- Easy bot controller на сервере: delays + always-correct peek + selfish heuristics
- Presence: bot label «AI»; leave human → dispose

**Non-Goals:**

- loadtest/отдельный бот-процесс; Hard AI; смешение людей и AI за одним столом; разные имена ботов

## Decisions

### D1 — Bot seats внутри комнаты (не отдельный Client)

**Выбор:** synthetic seat keys (`bot:1`…) + room-private `BotController`; **без** JWT/`Client` для ботов.

**Apply path (обязательно):** сегодня `handleMove` / `handleRescue` / `handleReturnFromFinish` / peek resolve завязаны на `client.sessionId`. При apply — вынести авторитетные мутации в **sessionId-scoped** internal helpers (`applyMove(sessionId, …)` и аналоги); human `onMessage` только парсит payload и вызывает helper; `BotController` вызывает **те же** helpers по ключу `bot:N`. Не эмулировать fake `Client`. Ключи `bot:N` участвуют в `turnOrder` / `currentTurnSessionId` как строки (как human sessionId). `this.clients.get(botKey)` для private `knowledge` — no-op (норма: боту WS не нужен; room-private `knowledge` map всё равно ведётся для эвристик).

**Spawn:** `spawnBotSeats()` выдаёт каждому боту уникальный `touristId` из того же пула `1…4`, что и `onJoin` человека (`pickUniform` по свободным id); `connected: true`, `isBot: true`, `ready` не требуется; pieces — как у людей, deferred до `enterPlaying`.

**Альтернатива:** `@colyseus/loadtest` / sidecar клиенты — отвергнуто (auth, ops, spawn на create).

### D2 — Create option `opponentMode: 'people' | 'ai'`

**Client:** В create modal — двухкнопочный opponent-mode radio **сразу после** блока maxSeats (секция видна вместе с maxSeats, т.е. после выбора карты; врезка: `LobbyPage` `q-card-section` после `lobby-create-max-seats`). Иконки Quasar Material (уже в бандле `material-icons`): `person` (люди) и `smart_toy` (AI). i18n: `lobby.opponentMode` (заголовок секции), `lobby.opponentPeople`, `lobby.opponentAi` (копия опций; product sense «Люди» / «AI»). Default при открытии модалки / `resetCreateForm`: `'people'`. Confirm всегда шлёт выбранный `opponentMode` в `createGame({ …, opponentMode })` — **клиент не coerce** при `maxSeats===1` (сервер сам). При `maxSeats===1` radio остаётся доступным.

**Server:** `onCreate` парсит `opponentMode` (`'ai'` только если явно передано; иначе `'people'`). **Effective AI** = `opponentMode === 'ai'` **и** `maxSeats ≥ 2`. При `maxSeats === 1` coerce → effective people/solo: без ботов, без `setPrivate`, без AI join-deny. После seat создателя в effective AI — `spawnBotSeats()` → `maybeStartCountdown()`.

### D3 — Unlist AI rooms

**Выбор:** `setPrivate(true)` на effective AI (Colyseus 0.18 `Room.setPrivate` есть). Metadata listing path для effective AI не нужен. Solo / people — listing как сейчас.

### D4 — Schema: `Seat.isBot`

Синхронизировать `isBot: boolean` на Seat (`true` у bot seats, `false` у человека). Client: зеркало `isBot` в `GameSeat`; на Game presence для seats с `isBot` — короткая текстовая подпись **под** аватаром туристика (тот же слот top-row / bottom HUD), i18n `game.presenceAi` → литерал `AI` (en-US каталог; RU copy тот же токен); chrome подписи — muted caption (`text-caption` / `.text-muted` аналог, без отдельного Visual Spec). Все боты — одна подпись, без суффиксов. Human seats **без** новой name-caption (как сейчас: только avatar + chrome). Отдельный synced `displayName` в этом change **не** добавляем. Open-on-land peek бота — **существующий** shared peek modal (`game/board`): все клиенты видят chrome, правит только peek owner (`isPeekOwner`); человек в AI-комнате остаётся non-owner и **не** получает новый peek UI / auto-submit — сервер закрывает сессию после answer-delay.

### D5 — Join gate

В effective AI: новый `onJoin` другого клиента → throw / hard deny без seat и без spectator (не silent spectator path full people-стола). Reconnect создателя — существующий `onReconnect`.

### D6 — Dispose

Effective AI: keep-alive только пока есть ≥1 human seated (connected или в grace), **в любой фазе**. Permanent human leave / grace expiry → `disconnect()`/dispose даже если bots > 0 (override «dispose only when seats.size===0»). Боты не в grace и сами комнату не держат.

### D7 — Easy policy + delays

Модуль `src/game/botEasy.ts` (pure scoring) + orchestration в room (таймеры — `this.clock.setTimeout`, аналог trap pipeline / turn deadline; отмена в `onDispose` / human-gone dispose):

1. На `onBecomeCurrent` для bot seat — schedule bot act (не ждать human message)
2. Пока legal own rescue → **отдельный** move-delay sample → rescue via sessionId helper
3. Затем (если нужен board action) → **ещё один** независимый move-delay sample → score legal moves/returns (knowledge on piece, к задачам/финишу, cut pressure когда last-piece finish оставляет одного eligible) → move/return helper
4. Open peek для bot session → после answer-delay выставить правильные placements / `resolveOpenPeek(true)` (тот же correct-path, что human submit)

Delay arrays (random pick):

```text
MOVE_DELAY_MS   = [600, 900, 1100, 1400]
ANSWER_DELAY_MS = [1200, 1600, 2000, 2500, 3000]
```

Один уровень Easy; усложнение — later.

### D8 — Точки врезки (файлы)

**Server** (`../happy-tourist-server/`):

- `src/rooms/schema/MyRoomState.ts` — `Seat.isBot`
- `src/rooms/MyRoom.ts` — create option, private, spawn bots, join deny, dispose-on-human-gone, schedule bot acts on turn / peek
- `src/game/botEasy.ts` — NEW pure heuristics
- `test/MyRoom.test.ts` (+ узкий unit на botEasy) — SC-AI / SC-START-20 / SC-PIECE-60 / SC-LEAVE-20…

**Client** (`../happy-tourist.github.io/`):

- Lobby create UI + i18n (opponent mode radio)
- `stores/game` — `createGame` передаёт `opponentMode`; mirror `isBot`
- Game presence — label AI для bot seats
- Vitest: LobbyCreateWire + presence/AI label

Чеклист реализации — `tasks.md`.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Easy heuristics «слабые» vs ожидание «неотличимо» | Зафиксирован Easy v1; later Hard; SC описывают rescue/legal/correct, не perfect play |
| Synthetic sessionId ломает код, ждущий `Client` | Все bot-действия только через sessionId-internal helpers; тесты на turn/peek |
| Private room + joinById всё ещё возможен | Явный deny в onJoin для чужих |
| Несколько ботов + cut | Selfish per seat; как за живым столом |

## Migration Plan

Обратная совместимость: отсутствие `opponentMode` = `people`. Старые клиенты без radio создают human-комнаты. Откат: не передавать mode / feature-flag не требуется.

## Open Questions

Нет (explore закрыл D*; числа delay зафиксированы здесь; effective AI = `ai` ∧ `maxSeats≥2` канон brief-server; client UX defaults — radio после maxSeats, icons `person`/`smart_toy`, default people, AI caption под аватаром — brief-client).
