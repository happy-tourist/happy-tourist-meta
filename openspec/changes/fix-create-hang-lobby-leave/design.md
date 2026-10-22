## Context

См. `proposal.md` — Why. Сейчас в client Pinia `game` store путь `_enterRoom` после успешного `client.create` / `joinById` / `reconnect` делает `_attachRoom(room)`, затем **`await this.unsubscribeLobby()`**, а `unsubscribeLobby` делает **`await lobby.leave()`** без timeout. UI Lobby (`onConfirmCreate` / join) ждёт полный `createGame`/`joinGame`, поэтому зависший leave держит диалог и не пускает на Game, хотя tourist room уже жива и seat выдан.

Порядок attach-before-unsubscribe намеренный: не пропустить первый `ROOM_STATE` в gap leave. Его сохраняем; меняем только блокирующее ожидание leave.

**Пакет:** client only. Server / room contract без изменений.

## Goals / Non-Goals

**Goals:**

- `_enterRoom` возвращает room сразу после `_attachRoom` + initiate unsubscribe, не `await`-я завершение leave лобби.
- `unsubscribeLobby` всё ещё очищает локальный listing state и пинает leave с коротким timeout (не forever); callers вроде `leaveGame`/`subscribeLobby` могут `await` этот bounded leave.
- Create и join (и `rejoinGame` через `_enterRoom`) делят один путь — один фикс.
- Vitest на SC-LOBBY-05 / SC-LOBBY-37; client skills синхронизированы с D1.

**Non-Goals:**

- Server grace / reconnect reclaim после F5; правки `LobbyRoom` / `MyRoom` / listing metadata.
- Новые server messages, schema, HTTP — create/join/leave listing **reuse** (см. блок ниже).
- Менять LobbyPage UX диалога сверх того, что следует из более быстрого resolve store (закрытие после успешного create остаётся в page `finally`/после await).
- Новые npm-зависимости.

### Reused BE contract (as-is, zero server tasks)

Этот change **не** меняет server. Client после успешного enter только иначе ждёт (или не ждёт) завершения leave лобби.

| Поверхность | Что уже есть (reuse) |
|-------------|----------------------|
| Room `lobby` | Colyseus `LobbyRoom` (`defineRoom(LobbyRoom)` в `app.config`) — live listing subscribers; **без** reconnect grace / hold (main SC-LOBBY-08) |
| Room `tourist` | `MyRoom` + `enableRealtimeListing()` — create / joinById / reconnect; seating + reconnect grace **без изменений** (main SC-LOBBY-04) |
| Leave лобби | Обычный Colyseus `room.leave()` на listing-подписке; отдельного room message / HTTP / schema для leave listing **нет** |
| Orphan после client timeout | После client-side bound leave WS может ещё жить; cleanup — disconnect / следующий subscribe; server leave-timeout **не** требуется |

Main: `openspec/specs/lobby/rooms/spec.md` (SC-LOBBY-04, SC-LOBBY-08). Runtime: `../happy-tourist-server/src/app.config.ts` (`lobby` / `tourist`).

## Decisions

### D1 — Background leave + timeout (выбрано в explore)

Разделить **локальный teardown** и **ожидание сети**:

1. `unsubscribeLobby` (все callers): **синхронно до первого `await`** — `lobbyWanted = false`, очистить `rooms`, обнулить `lobbyRoom`; если был lobby room — `leave()` через `Promise.race(leave, timeout)` (или helper) с константой D3; ошибки leave глотать как сейчас. Таким образом `leaveGame` / старт `subscribeLobby` тоже не висят forever на leave.
2. `_enterRoom` после `_attachRoom`: **инициировать** `unsubscribeLobby` **без** ожидания его завершения (`void this.unsubscribeLobby()` или эквивалент) → **сразу `return room`**. Не `await unsubscribeLobby()` до конца — иначе UI снова ждёт leave (даже с bound 1–2s create-диалог «подвис» бы на timeout). Вызов `void unsubscribeLobby()` всё равно выполняет синхронный teardown из п.1 **до** return `_enterRoom` (async-функция бежит до первого `await`) — SC-LOBBY-37 «local listing state cleared» без ожидания сети.
3. Порядок attach → initiate unsubscribe сохранить (D4).
4. Двойной вызов безопасен: LobbyPage `onUnmounted` тоже `void unsubscribeLobby()` после navigate; после п.1 `lobbyRoom` уже `null` — второй leave no-op.

**Alternatives:** (A) оставить blocking await — сохраняет hang; (B) pure fire-and-forget без timeout — риск zombie WS; отклонены.

### D2 — Точка врезки

| Файл | Роль |
|------|------|
| `../happy-tourist.github.io/src/stores/game.ts` | `LOBBY_LEAVE_TIMEOUT_MS`; `unsubscribeLobby` (sync clear + timeout-bound leave); `_enterRoom` (`void unsubscribeLobby()` после `_attachRoom`, сразу `return room`) |
| `../happy-tourist.github.io/src/pages/LobbyPage.vue` | без обязательных правок: `creating`/`joining` уже в `finally` после `await createGame`/`joinGame`; после D1 store resolve → modal close + `router.push` без ожидания leave |
| `../happy-tourist.github.io/src/stores/__tests__/game.lobbyEnter.test.ts` (новый) | store-level vitest SC-LOBBY-05 / SC-LOBBY-37; mock `lobby.leave` never-resolving + fake timers |
| Client skills (meta) | §1.4: заменить канон `await unsubscribeLobby()` после attach в `_enterRoom` на initiate/`void` + timeout в `unsubscribeLobby` — иначе apply скопирует старый blocking snippet |

Слои: pages → stores; Colyseus I/O остаётся в Pinia. Skills править в том же change (tasks §1.4), не оставлять противоречие design ↔ skill.

### D3 — Timeout величина

В `game.ts` (или рядом): константа **`LOBBY_LEAVE_TIMEOUT_MS = 1500`** (допустимый диапазон 1000–2000 ms). Использовать в `Promise.race` внутри `unsubscribeLobby`. Не продукт-facing; достаточно оборвать hang. В delta/spec — «short bound» без числа.

### D4 — Attach order

Сохранить: `connect` → `_attachRoom` → initiate unsubscribe. Не leave до attach.

### Prerequisites / блокеры

Блокеров нет: D1 зафиксирован (C). Стек Vue/Quasar/Colyseus без новых провайдеров.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Короткое окно: tourist attached, lobby leave ещё в полёте | Локально `lobbyRoom = null` / `lobbyWanted = false` сразу; quiet-resubscribe не должен реаттачить (уже есть guards) |
| Timeout оборвал leave, WS ещё жив на сервере | Следующий visit lobby / remount сделает новый subscribe; orphan lobby client уйдёт по disconnect |
| Тест flaky на real timers | Fake timers + never-resolving leave mock |

## Migration Plan

Только client deploy. Rollback = предыдущий blocking await.

## Open Questions

Нет. Диапазон timeout 1–2s закрыт решением D3 (default **1500 ms**).

Чеклист реализации — `tasks.md`.
