## 1. Server — create snapshot & room metadata

- [x] 1.1 Прочитать `design.md`, delta `lobby/rooms` + `content/maps` + `content/packs`; skills `server-locate-change-points`, `work-with-rooms`, `work-with-config` / content helpers — зафиксировать точки врезки create options
- [x] 1.2 Расширить `MyRoom.onCreate`: парсить `mapId`, `packId`, `taskSetIds[]`, densities; загружать live in-catalog map + published sets; отклонять soft-unpublished / missing (SC-LOBBY-22/27, SC-MAP-60/61, SC-PACK-200/201)
- [x] 1.3 Снапшотить grid, players, touristsPerPlayer, tasks (question/difficulty/slots), pack answers; `maxSeats = players`; metadata для lobby (preview, capacity, set labels) — SC-LOBBY-25/26
- [x] 1.4 Mocha: create success/reject paths с теми же SC-ID; `npm test` в server

## 2. Server — board layout, deck, peek

- [x] 2.1 Skills `work-with-schema`, `work-with-messages`, `work-with-game`; delta `game/board` + `game/move` — убрать hardcoded layout/bag/stub из authoritative path
- [x] 2.2 Synced layout из snapshot; task deck shuffle; bind on first peek; public difficulty on cell (SC-BOARD-40/41/42/48)
- [x] 2.3 Shared peek session + place/submit messages; order validation; correct → +difficulty + remove tile; wrong/leave/timeout keep bind (SC-BOARD-43/44/46/47)
- [x] 2.4 Fresh peek spends peeks; flipped free at peeks=0; auto-end treats flipped as available (SC-BOARD-45/10, SC-MOVE-39/94/95)
- [x] 2.5 Mocha на SC-BOARD-* / SC-MOVE-* выше; `npm test`

## 3. Server — pieces without sides

- [x] 3.1 Skills `work-with-schema`, `work-with-game`; delta `game/pieces` — pcs = touristsPerPlayer; убрать side N/E/S/W; strip index by piece id
- [x] 3.2 Materialize + all-jail: greedy max distance among own pieces on free map starts (SC-PIECE-01/03/04/18/25/50/51)
- [x] 3.3 Обновить move/peek/rescue/push/return/finish handlers под piece id без side
- [x] 3.4 Mocha SC-PIECE-*; поправить регрессии старых side-тестов; `npm test`

## 4. Client — create & lobby listing

- [x] 4.1 Skills `client-locate-change-points`, `work-with-lobby`, `work-with-pages`, `work-with-stores`, `work-with-localization`; delta `lobby/rooms`
- [x] 4.2 Create modal: map picker + capacity; pack + multi-check published sets; densities; убрать maxSeats picker (SC-LOBBY-21…24/27)
- [x] 4.3 `createGame` options → server; listing row: map preview, capacity, pack/set labels (SC-LOBBY-25/26)
- [x] 4.4 Vitest SC-LOBBY-*; `npm test` в client

## 5. Client — board, peek modal, pieces strip

- [x] 5.1 Skills `work-with-game-board`, `work-with-stores`, `colyseus-client`, `work-with-styles`, `work-with-localization`; deltas `game/board`, `game/pieces`
- [x] 5.2 Рендер board из room snapshot; flipped difficulty digits; убрать hardcoded LAYOUT (SC-BOARD-01/40/42)
- [x] 5.3 Shared peek modal (question, slots, pack answers); peeker place/submit; spectators/others read-only sync (SC-BOARD-43/44/46)
- [x] 5.4 Strip из touristsPerPlayer слотов; selection by piece id (SC-PIECE-09/52)
- [x] 5.5 Vitest SC-BOARD-* / SC-PIECE-* client UX; `npm test`

## 6. Client — focus control

- [x] 6.1 Skills `work-with-pages` / presence HUD; delta `game/presence`
- [x] 6.2 Круглая кнопка между say и end-turn; фокус nearest actionable tourist; hide when none (SC-PRESENCE-30/31/32)
- [x] 6.3 Vitest SC-PRESENCE-*; `npm test`

## 7. Verify packages

- [x] 7.1 Server: `npm run lint` (если есть), `npm test`, `npm run build`
- [x] 7.2 Client: `npm run lint`, `npm run typecheck`, `npm test`, `npm run build`
