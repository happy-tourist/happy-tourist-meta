## 1. Server — schema, create AI seats, private room, join/dispose

- [x] 1.1 Перед работой: `design.md` (D1–D6, D8; effective AI = `ai` ∧ `maxSeats≥2`); delta specs `game/ai` SC-AI-01/02/09/10, `lobby/rooms` SC-LOBBY-53, `game/start` SC-START-20, `game/pieces` SC-PIECE-60/61, `game/leave` SC-LEAVE-20/21/22; skills `server-work-with-schema`, `work-with-rooms`, `work-with-game`; `MyRoom.ts` onCreate/onJoin/onLeave/refreshMetadata
- [x] 1.2 Добавить `Seat.isBot`, parse `opponentMode` (omit→people; maxSeats=1 coerce people), `setPrivate` только для effective AI, spawn bot seats до maxSeats (уникальный `touristId` 1…4), hard-deny чужой join/spectator, dispose при уходе единственного человека в любой фазе (bots не держат комнату); вынести move/rescue/return/peek apply в sessionId-helpers (D1) — по design D1–D6
- [x] 1.3 Mocha: SC-AI-01, SC-AI-02, SC-AI-09, SC-AI-10, SC-START-20, SC-LOBBY-53 (unlist / setPrivate), SC-PIECE-60, SC-PIECE-61, SC-LEAVE-20 (и SC-LEAVE-21/22 при удобной фикстуре); `npm test` в server

## 2. Server — Easy bot turn + delayed correct peek

- [x] 2.1 Перед работой: `design.md` D7; `game/ai` SC-AI-03…08; `work-with-game`, `work-with-messages`; `touristMove.ts` legal helpers; peek `resolveOpenPeek` / placements
- [x] 2.2 Добавить `src/game/botEasy.ts` + room orchestration: MOVE/ANSWER delay arrays, отдельный move-delay на rescue и на board action, always-correct peek after answer-delay; schedule on bot current turn / openPeek
- [x] 2.3 Mocha (+ unit botEasy): SC-AI-03, SC-AI-04 (эвристика ≠ idle-timeout-only), SC-AI-05, SC-AI-06, SC-AI-07 (сравнение массивов), SC-AI-08 (нет grace у bots); fake timers/clock; `npm test` в server

## 3. Client — create opponent mode + createGame payload

- [x] 3.1 Перед работой: `design.md` D2/D8 client; `lobby/rooms` SC-LOBBY-50…52; skills `work-with-lobby`, `work-with-pages`, `work-with-localization`, `client-work-with-test`; Lobby create + `game.createGame`
- [x] 3.2 После maxSeats — radio люди/AI с Material `person` / `smart_toy`; i18n `lobby.opponentMode` / `lobby.opponentPeople` / `lobby.opponentAi`; default `'people'`; `CreateGameOptions.opponentMode` + `createGame({ opponentMode })` всегда шлёт выбор (без client-coerce при maxSeats=1)
- [x] 3.3 Vitest LobbyCreateWire: SC-LOBBY-50 (иконки + default people), SC-LOBBY-51, SC-LOBBY-52; `npm test` (затронутые) в client

## 4. Client — presence label AI + store mirror isBot

- [x] 4.1 Перед работой: `game/presence` SC-PRESENCE-60; design D4; `work-with-game-board` / presence notes; `work-with-stores`; game store seat mirror
- [x] 4.2 Зеркалировать `isBot` в `GameSeat`; под аватаром bot seats — caption i18n `game.presenceAi` («AI»), одинаково для всех ботов; human seats без новой name-caption
- [x] 4.3 Vitest на SC-PRESENCE-60; `npm run lint` + `npm run typecheck` + `npm test` в client

## 5. Server lint / test / build

- [x] 5.1 Перед работой: кратко сверить Traceability SC-AI-01…10 / SC-START-20 / SC-LOBBY-53 / SC-PIECE-60/61 / SC-LEAVE-20…22 с именами тестов из §1–2
- [x] 5.2 `npm test` + `npm run build` в server (lint — если скрипт есть в package.json)
