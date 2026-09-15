## Context

См. `proposal.md` — Why. Сейчас `MyRoom` жёстко сидит до 4, ставит `started=true` на 4-м seat и **не** гейтит `move`; client `LobbyPage` создаёт комнату без options, кнопка «Играть» делает `joinOrCreate`. Нужен согласованный контракт client+server для ёмкости, фаз старта и ready.

Пакеты: **server** (`../happy-tourist-server`) затем **client** (`../happy-tourist.github.io`). Чеклист — `tasks.md`.

## Goals / Non-Goals

**Goals:**

- Schema + metadata: `maxSeats`, фаза старта, ready на seat, countdown.
- Message `ready` → seat.ready + say preset readiness.
- Авто countdown при `seats.size === maxSeats`; all-ready при недоборе (≥2).
- Gate `move` до `playing`; seats открыты пока `seats.size < maxSeats` в любой фазе.
- Lobby: modal maxSeats; `occupied/maxSeats`; без «Играть».
- Game: overlay countdown; кнопка ready у say; lock move chrome.

**Non-Goals:**

- Matchmaking / восстановление «Играть».
- Конец партии.
- Новые npm-зависимости.
- Переписывание reconnect grace (только «не отменять countdown»).

## Decisions

### D1 — Фаза отдельно от legacy `started`

- **Выбор:** synced `phase`: `waiting` | `countdown` | `playing` (+ `maxSeats`, `countdownEndsAt` или `countdownRemaining`). Поле `started` либо deprecate (зеркало `phase === 'playing'` / seating full), либо заменить чтением phase на client — в apply предпочесть **явный `phase`**, а `started` оставить временно как `phase !== 'waiting'` только если ломает меньше тестов; канон skills после apply: phase-first.
- **Альтернатива:** перегрузить `started` = seating lock — отвергнуто (seats остаются открытыми mid-game).

### D2 — Countdown авторитетен на server

- **Выбор:** `clock.setTimeout` / sequential ticks в `MyRoom`; в schema публиковать оставшиеся секунды или `countdownEndsAt` (ms), client рисует overlay из synced state (не локальный setInterval как truth).
- Длительность: 5 секунд, отображение 5…1.
- **Альтернатива:** только client timer — отвергнуто (рассинхрон).

### D3 — Ready message + say

- **Выбор:** `onMessage('ready')` (пустой payload): если eligible → `seat.ready = true`, broadcast `say` с preset id readiness (тот же канал, что greeting/luck). Client `sendReady()` в Pinia `game` store; UI не шлёт сырой `room.send`.
- Preset id: `ready` (whitelist server + client `SAY_PRESET_IDS`); i18n «Готов начать!».
- **Альтернатива:** client сам `sendSay('ready')` + отдельно ready — отвергнуто (два round-trip / рассинхрон флага).

### D4 — Create options

- **Выбор:** `client.create('tourist', { maxSeats: 2|3|4 })`; server `onCreate` валидирует, default 2, иначе clamp/reject invalid → default 2.
- Metadata для lobby listing: `{ title, status, maxSeats, seats }` обновлять на join/leave/phase; listing UI читает metadata (не `clients/maxClients` как seats).

### D5 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/schema/MyRoomState.ts` | `maxSeats`, `phase`, countdown field; `Seat.ready` |
| `src/rooms/MyRoom.ts` | options; seat iff `seats.size < maxSeats`; trigger countdown; `ready` handler; move gate; metadata; убрать hardcode `>= 4` / permanent lock |
| `test/MyRoom.test.ts` (+ при необходимости новые describe) | SC-START / rewritten SC-PIECE-05/06/08 / SC-MOVE-18 / SC-SAY-13 |
| messages skill path | whitelist + ready |

### D6 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `src/pages/LobbyPage.vue` | modal radio 2/3/4; убрать «Играть»; capacity `seats/maxSeats` из metadata |
| `src/stores/game.ts` | mirror phase/maxSeats/ready/countdown; `createGame({ maxSeats })`; `sendReady`; status из phase |
| `src/pages/GamePage.vue` | overlay; ready btn у own say affordance; lock move chrome if not playing |
| `src/i18n/*` | create modal, countdown text, ready btn, say.ready |

### D7 — Explore prerequisites (закрыты)

Все D1–D6 / Q1–Q5 / M1–M2 из explore закрыты продуктово; в design нет открытых блокеров. Skills `work-with-game`, `work-with-lobby`, `work-with-game-board`, `work-with-messages` обновить после кода (check-changes / align) — не блокер реализации.

## Risks / Trade-offs

- [Лобби metadata vs clients] → Явно писать `seats`/`maxSeats` в `setMetadata` на каждое изменение seats; UI не использовать raw `clients` как occupied.
- [Рассинхрон countdown UI] → Только schema clock; client может анимировать, но переход `playing` только от server state.
- [Mid-game seat + фигуры] → Тот же assign path, что join в waiting; turnOrder append (уже паттерн).
- [Старые клиенты] → BREAKING; деплой server+client вместе.
- [Тесты SC-PIECE-05/08] → Переписать под maxSeats / reopen seats; не оставлять «always 4 / no reopen».

## Migration Plan

1. Задеплоить server с phase/maxSeats (старый client сломает ожидания started — не оставлять mixed).
2. Сразу задеплоить client с modal/overlay/ready.
3. Rollback: simultaneous revert обоих siblings.

## Open Questions

Нет (отложенных без влияния на specs/tasks).
