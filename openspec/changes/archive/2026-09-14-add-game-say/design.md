## Context

См. `proposal.md` (Why / Scope) и delta `specs/game/say/spec.md`. Сейчас: room `tourist` принимает только `move`; presence-маркеры на Game (`GamePage` + `game/presence`); room I/O в Pinia `stores/game.ts` (`sendMove`, `onStateChange`). Пресетных реплик нет. Пакеты: **server** (`../happy-tourist-server`), **client** (`../happy-tourist.github.io`), **meta** (skills messages / board / rooms при lockstep).

Prerequisite из explore: закрыты (D1–D3, Q1–Q5, A = anytime seated+online). Блокеров нет.

## Goals / Non-Goals

**Goals:**

- Ephemeral room protocol `say` (whitelist `presetId`) + server concurrent limit (max 3 / 10s TTL) + broadcast всем в комнате.
- Client: affordance + пикер у своего presence; стек комикс-облаков по слоту; listen через Pinia; без Notify как носителя.

**Non-Goals:**

- Sync say в `@colyseus/schema` / БД; свободный текст; lobby messages; новые npm-зависимости; Quasar Notify plugin.

## Decisions

### D1 — Message names and payloads

- Client → server: `room.send('say', { presetId: 'hello' | 'luck' })`.
- Server → all clients: `client.send` / `room.broadcast('say', { sessionId, presetId, at })` где `at` — server timestamp (ms) для единого TTL.
- Display strings только на client: `hello` → «Всем привет», `luck` → «Удачи».
- Альтернатива schema Map bubbles — отвергнута (эфемерность, лишний sync churn).

### D2 — Server authority in `MyRoom`

- Точка врезки: рядом с `onMessage('move')` / `handleMove` в `src/rooms/MyRoom.ts`.
- Accept: client имеет seat, seat `connected`, `presetId` ∈ whitelist, число **ещё живых** say этого `sessionId` < 3 (по server clock + `at` + 10_000 ms).
- Reject: silent (как move) — не мутировать gameplay state.
- Хранить room-private ring/list `{ sessionId, at }[]` (не schema); чистить просроченные при каждом say и опционально по timer.
- Не требовать `currentTurnSessionId`.

### D3 — Client I/O in Pinia `game` store

- `sendSay(presetId)` → `room.send('say', …)` только из store (pages не зовут `room.send`).
- Подписка `room.onMessage('say', …)` при bind tourist room; складывать ephemeral list / emit в page-local state.
- Pages/UI читают mirrored events; не держать Colyseus в шаблоне.

### D4 — UI на presence в `GamePage`

- Affordance (иконка `chat_bubble_outline` / CSS bubble) **только** у маркера с `sessionId === game.sessionId` и seat connected.
- Клик → две кнопки пресетов; выбор → `sendSay` + закрыть пикер сразу; клик мимо — закрыть (разумный default).
- Bubbles: абсолют/relative рядом с `.presence-marker`; классы по `slot` (`top`/`bottom`/`left`/`right`).
- Стек: новое ближе к аватару; left/right — новое снизу, старое вверх (`column` / `column-reverse` по слоту).
- TTL 10s от `at` (или local receive time если `at` отсутствует — предпочитать server `at`).
- Max 3: клиент не шлёт 4-й, пока локально/по `at` живы три; server — истина.

### D5 — Tests and skills

- Server mocha: SC-SAY-01…06 (accept whitelist, reject free/unknown, spectator, offline, off-turn ok, max-3, broadcast).
- Client: lint/typecheck; UX SC-SAY-07…12 покрыты реализацией (как presence/move client scenarios).
- Meta: обновить `work-with-messages`, `work-with-game-board` (и при необходимости `colyseus-client` / `work-with-stores`) под `say`.
- Чеклист — `tasks.md`.

## Risks / Trade-offs

- [Клиентский clock skew для TTL] → Mitigation: server `at`; клиент expire по `at + 10_000`.
- [Спам кликами] → Mitigation: server max 3 live; silent reject.
- [Тесный presence grid + 3 bubbles] → Mitigation: короткий текст пресетов; max-width; не Notify.
- [Свободный текст «потом»] → Mitigation: продуктовый запрет в proposal/spec; только whitelist ids.

## Migration Plan

- Деплой server с handler `say` до или вместе с client (старый client просто не шлёт).
- Rollback: убрать handler + UI; schema не затронута.
