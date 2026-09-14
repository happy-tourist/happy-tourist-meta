## 1. Server — say contract

- [x] 1.1 Прочитать `design.md` (D1–D2), delta `specs/game/say/spec.md`, skills `work-with-messages` / `work-with-rooms` / `server-work-with-test`; сверить `MyRoom.ts` и существующий `move` handler как аналог
- [x] 1.2 Реализовать `onMessage('say')`: whitelist `hello`|`luck`; только seated + connected; room-private live list + TTL 10s; max 3; `broadcast('say', { sessionId, presetId, at })`; silent reject иначе — SC-SAY-01…06, SC-SAY-10 (server)
- [x] 1.3 Mocha на SC-SAY-01…06 и concurrent max-3 (SC-SAY-10); обновить Traceability в delta `game/say` на covered (server) где применимо
- [x] 1.4 `npm test` в `../happy-tourist-server`; починить до перехода к client

## 2. Client — store + presence bubbles

- [x] 2.1 Прочитать `design.md` (D3–D4), skills `work-with-game-board` / `work-with-stores` / `colyseus-client` / `work-with-rooms`; сверить `stores/game.ts` и presence-блок `GamePage.vue`
- [x] 2.2 Store: `sendSay(presetId)` + `room.onMessage('say')` → ephemeral events; pages без прямого `room.send` — verify: typecheck
- [x] 2.3 Game UI: affordance только у своего online маркера; пикер «Всем привет» / «Удачи»; закрытие сразу; bubbles TTL 10s, max 3, стек по slot (D4) — SC-SAY-07…12
- [x] 2.4 `npm run lint` + `npm run typecheck` в `../happy-tourist.github.io`

## 3. Meta — skills

- [x] 3.1 Обновить `work-with-messages` и `work-with-game-board` (при необходимости `colyseus-client` / `work-with-stores`): контракт `say`, whitelist, bubbles у presence; сверить с proposal/design/specs
