## Context

См. `proposal.md` — Why. Сейчас: `currentTurnSessionId` без дедлайна; presence — синяя обводка + optional `q-circular-progress` на reconnect (скачок solo ↔ progress); pieces создаются при seat в любой фазе. Нужны authoritative turn/solo deadlines, dual rings без скачка, deferred pieces до `playing`.

Пакеты: **server** (`../happy-tourist-server`) затем **client** (`../happy-tourist.github.io`). Чеклист — `tasks.md`.

Explore prerequisites (закрыты): D1 pass; D2 tick during grace; D3 server deadline; D4 pieces after start; D5 lock + clear selection; D6 room like finish; D7 fresh 5:00; Q1 only `playing`; dual rings outer turn / inner reconnect; spectators see timers; leave time-expired без confirm.

## Goals / Non-Goals

**Goals:**

- Synced `turnUntil` (+ флаг/бюджет solo vs 60s) и `timeExpired` на seat/room; room clock → advanceTurn или lock.
- Deferred piece spawn: seat+kind в waiting/countdown; materialize на `playing`.
- Client: dual circular progress, reserved size, убрать `--turn` outline; solo modal; mirror state в Pinia.
- Mocha на SC-MOVE-25…30 / SC-PIECE deferred; client lint/typecheck.

**Non-Goals:**

- Новые HTTP routes / новые Colyseus messages (только synced state + clock).
- Auto-kick / finish place для time-expired.
- Новые npm-зависимости.

## Decisions

### D1 — Synced deadline shape

- **Выбор:** room-level `turnUntil: number` (unix ms, `0` = нет активного таймера) + `Seat.timeExpired: boolean` (default false). Solo vs multi определяется числом eligible seats при set deadline (60s vs 5min); отдельный enum не обязателен — клиент красит red, когда eligible count === 1 и `turnUntil > 0` (или явный `turnBudgetSeconds` в state для однозначности UI).
- **Рекомендация UI:** синкнуть также `turnBudgetSeconds` (60 | 300) при установке дедлайна, чтобы цвет/max кольца не угадывать.
- **Альтернатива:** только client-local timer — отвергнуто (D3 / D1 auto-pass).

### D2 — Clock lifecycle

- **Выбор:** при каждом назначении current turn в `playing` (вход в playing, после move, после 60s pass, после leave-advance) — `clearTimeout` предыдущего, `turnUntil = now + budget`, `clock.setTimeout` на остаток. Budget = 300s если ровно один eligible, иначе 60s. При появлении «остался один» mid-turn — **сброс** на ровно 5:00 (explore D7). При finish последнего / time-expired / нет eligible — `turnUntil = 0`, cancel timer.
- Offline grace **не** паузит timeout (D2).
- Waiting/countdown: `turnUntil = 0`, таймер хода не ставится.

### D3 — Solo expiry

- **Выбор:** `seat.timeExpired = true`; `turnUntil = 0`; reject `move` как finished; room не dispose. Client watch `mySeat.timeExpired` → modal (i18n); clear selection. Leave: `needsLeaveConfirm` = seated ∧ playing ∧ !finishPlace ∧ !timeExpired.

### D4 — Deferred pieces

- **Выбор:** вынести create-four-pieces из join path; на join в waiting/countdown — только seat+kind+turnOrder; на `phase = playing` (конец countdown) — `materializePiecesForAllSeats()`; join уже в playing — сразу pieces как сейчас.
- Consented leave / grace: если pieces пусты — просто remove seat/kind.

### D5 — Presence chrome

- **Выбор (исправлено после регресса пустых маркеров):** контейнер 52px; **sibling** rings — outer turn `q-circular-progress` (52px, absolute behind; primary/blue или negative/red), inner reconnect (40px, warning или transparent). **Avatar:** всегда отдельный `<img class="presence-avatar">` sibling поверх колец (как pre-timer solo img) — **не** default slot `q-circular-progress`. Quasar рисует default slot **только при `show-value`**; без него `<img>` в шаблоне не попадает в DOM. Не вкладывать progress в progress.
- Удалить CSS `--turn` box-shadow. Не использовать solo-img ветку для смены размера маркера (размер держит reserved outer chrome).
- Max outer = `turnBudgetSeconds`; value = remaining from `turnUntil - now`.

### D6 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/schema/MyRoomState.ts` | `turnUntil`, optional `turnBudgetSeconds`; `Seat.timeExpired` |
| `src/rooms/MyRoom.ts` | schedule/clear turn timer; timeout → advance or expire; defer pieces; materialize on playing; move reject if timeExpired |
| `test/MyRoom.test.ts` | SC-MOVE-25…30, SC-PIECE-01/05/18, SC-START-13 (ускорить clock / stub grace timings) |

### D7 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `src/stores/game.ts` | mirror `turnUntil`, `turnBudgetSeconds`, `timeExpired`; helpers |
| `src/pages/GamePage.vue` | sibling dual rings + sibling avatar img; reserved size; timeout modal; clear selection; leave confirm gate |
| `src/i18n/*` | copy модалки timeout (смысл: не успели довести туристов) |

### D8 — Skills при apply

Server: `work-with-schema`, `work-with-game`, `work-with-rooms`, `server-work-with-test`. Client: `work-with-game-board`, `work-with-stores`, `work-with-rooms`, `work-with-localization`.

## Risks / Trade-offs

- [Долгие mocha на 60s/300s] → В тестах уменьшать константы через export / inject budget, или `clock` fake; не ждать реальных 5 мин в CI.
- [Рассинхрон wall clock client] → Рисуем remaining от synced `turnUntil`; tick `nowMs` локально (как reconnect).
- [Пустые presence без avatar] → Не класть img в slot progress без `show-value`; канон — sibling `<img>` поверх колец (D5). Align Axis C: nested progress + missing `show-value` → hard defect.
- [Старые клиенты без timeExpired] → Согласованный деплой; defaults безопасны (`timeExpired=false`, `turnUntil=0`).

## Migration Plan

- Server schema defaults backward-compatible.
- Выкатывать server + client вместе.
- Rollback: revert siblings.

## Open Questions

Нет.
