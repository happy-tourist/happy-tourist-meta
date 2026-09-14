## 1. Server — schema и reconnect lifecycle

- [x] 1.1 Прочитать `design.md` (D1–D4), delta `specs/game/pieces/spec.md`, skills `server/work-with-rooms`, `server/work-with-schema`, `server/work-with-game`; сверить текущие `MyRoom` / `MyRoomState` / `test/MyRoom.test.ts`
- [x] 1.2 Добавить на Seat sync-поля connectivity (`connected`, `reconnectUntil`) по design D2; при assign seat — online (`connected=true`, deadline 0); проверить, что schema собирается (`npm run build` в server)
- [x] 1.3 Реализовать unexpected drop → grace 30 с + hold seat; consented leave → сразу remove seat; успешный reconnect → seat online; timeout → remove как leave; verify: mocha SC-PIECE-11…14 (и обновлённые SC-PIECE-07/08)
- [x] 1.4 После финального remove seat: при `seats.size === 0` закрыть комнату (зрители не удерживают); verify: mocha SC-PIECE-15; sync offline/deadline — SC-PIECE-16
- [x] 1.5 Прогнать `npm test` и `npm run build` в `happy-tourist-server`; починить падения

## 2. Client — token, reconnect, mirror, lobby quiet

- [x] 2.1 Прочитать `design.md` (D3, D5–D7), delta pieces/presence/`lobby/rooms`, skills `client/work-with-rooms`, `client/work-with-lobby`, `client/work-with-stores`, `client/colyseus-client`; сверить `stores/game` и Game mount rejoin
- [x] 2.2 Сохранять reconnection token в `sessionStorage` **только** для tourist после enter; чистить на consented leave; на Game mount без живого room — `reconnect` затем fallback `joinById`; verify: typecheck/lint и поведение по design D3 *(исторически: до revision D3 → `localStorage`; см. §5)*
- [x] 2.3 Зеркалировать `connected` / `reconnectUntil` в store seats; verify: `npm run typecheck` в client
- [x] 2.4 Lobby per design D7 / SC-LOBBY-08: не persist lobby token; не `allowReconnection` на lobby; optional `reconnection.enabled = false` на lobby room; drop → clear + quiet resubscribe на Lobby; не класть `seat reservation expired` / reconnect-шум lobby в user-facing listing `error` (SC-LOBBY-07 только для реального fail подписки)

## 3. Client — presence UI

- [x] 3.1 Прочитать delta `specs/game/presence/spec.md`, skills `client/work-with-game-board`, `client/work-with-styles`; реализовать кружки только occupied на Game (SC-PRESENCE-01/05)
- [x] 3.2 Раскладка seated (self bottom; opponents top/left/right) и spectator (top/bottom/left/right) — SC-PRESENCE-02/03
- [x] 3.3 Офлайн: `QCircularProgress` по remaining от `reconnectUntil` вокруг tourist image; online без кольца — SC-PRESENCE-04
- [x] 3.4 Прогнать `npm run lint` и `npm run typecheck` в `happy-tourist.github.io`; починить падения

## 4. Meta skills

- [x] 4.1 Обновить client/server `work-with-rooms` + `work-with-lobby` (и при необходимости board/schema/game): tourist reconnect grace vs lobby fire-and-forget / quiet resubscribe; token только tourist; presence; сверить с design — без противоречий skills ↔ specs

## 5. Client — `localStorage` token (D3 revision)

- [x] 5.1 Перевести tourist reconnect persist с `sessionStorage` на `localStorage` (ключ/формат те же); clear на consented leave; clear stale после failed `reconnect`; не писать lobby token; verify SC-PIECE-17/18 поведению и `npm run typecheck` / `lint`
- [x] 5.2 Обновить client skills (`work-with-rooms`, `work-with-stores`, при необходимости lobby/board) и server notes если упоминают sessionStorage: канон = `localStorage`, cross-tab steal OK, без token = fresh join
