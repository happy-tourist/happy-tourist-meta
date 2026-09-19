## 1. Server — seed, fling helpers, land-resolve

- [x] 1.1 Прочитать `design.md` (D1–D4, D7, D10), delta `specs/game/move` + `lobby/rooms` (SC-MOVE-78…89, SC-LOBBY-17), skills `server/work-with-game`, `server/work-with-schema`, `server/work-with-rooms`, `server/server-work-with-test` и аналоги grille seed / land trap — зафиксировать sync reveal и точки врезки
- [x] 1.2 `touristMove.ts`: reuse density ratios; `catapultFlingCandidates` / `pickCatapultFlingDest` (ring-2 then ring-1, free landable) — unit/mocha: SC-MOVE-81/82 фильтры
- [x] 1.3 `MyRoom` + schema: parse `catapultDensity` (default medium); private hidden catapult set; seed on enterPlaying (task-only, overlap OK); minimal sync for reveal/broken — mocha: SC-MOVE-78/79, SC-LOBBY-17
- [x] 1.4 Единый `resolveCellTraps` после move/push/return land и после successful rescue; catapult one-shot fling/broken; post-fling land chain; center → finish — mocha: SC-MOVE-80/83/84/85/86/87/88, SC-FINISH-19
- [x] 1.5 Из sibling `happy-tourist-server`: `npm test` — SC-MOVE-78…89, SC-LOBBY-17, SC-FINISH-19 зелёные

## 2. Client — lobby catapult density

- [x] 2.1 Прочитать `design.md` D6, delta `specs/lobby/rooms` (SC-LOBBY-16…18), skills `client/work-with-lobby`, `client/work-with-stores`, `client/work-with-localization`
- [x] 2.2 Create modal: отдельный селектор catapult density (few/medium/many, default medium) + pass в create options; i18n мало/средне/много — SC-LOBBY-16/18
- [x] 2.3 Из sibling client: `npm run lint` и `npm run typecheck`

## 3. Client — board catapult overlay + assets

- [x] 3.1 Прочитать `design.md` D5/D8, delta `specs/game/board` (SC-BOARD-22…24) + finish SC-FINISH-19, skills `client/work-with-game-board`, `client/work-with-styles`, `client/colyseus-client`
- [x] 3.2 Положить ассеты `src/assets/catapults/catapult.png` и `catapult-broken.png` (user art)
- [x] 3.3 Game store: mirror reveal/broken sync; GamePage: hidden until reveal; fade in→out ~1000 ms; broken на vanish; piece travel / finish parity — SC-BOARD-22…24, SC-FINISH-19
- [x] 3.4 `npm run lint` + `npm run typecheck`

## 4. Meta — skills

- [x] 4.1 Обновить server skills (`work-with-game`, `work-with-schema`, `work-with-rooms`, `work-with-messages` при необходимости) — catapultDensity, land-resolve stack, fling
- [x] 4.2 Обновить client skills (`work-with-lobby`, `work-with-game-board`, `work-with-stores`, localization) — density UI, catapult overlay/assets
- [x] 4.3 При необходимости — краткие строки в sibling AGENTS Business Entities (без дублирования specs)

## 5. Client — sequential catapult presentation + board lock (follow-up)

- [x] 5.1 Прочитать `design.md` D5/D11/D12, delta `specs/game/board` (SC-BOARD-23…27) + `game/finish` SC-FINISH-19, skill `client/work-with-game-board`
- [x] 5.2 GamePage: pin piece on catapult cell during successful overlay; after full vanish → travel to sync dest / finish travel — SC-BOARD-23/25, SC-FINISH-19
- [x] 5.3 Broken timeline: appear → 300 ms intact → broken → 300 ms broken → vanish; no travel — SC-BOARD-24
- [x] 5.4 Sequential queue for fling→catapult chains (no cap) — SC-BOARD-26
- [x] 5.5 Board-busy lock: `isInteractive` false during move/grille/catapult/finish/fling anims — SC-BOARD-27
- [x] 5.6 `npm run lint` + `npm run typecheck`; обновить client skills / AGENTS blurbs про sequencing + lock

## 6. Client — land-before-overlay + reliable enqueue (follow-up)

- [x] 6.1 Прочитать `design.md` D5/D11/D13, delta `specs/game/board` (SC-BOARD-23…28) + `game/finish` SC-FINISH-19, skill `client/work-with-game-board`
- [x] 6.2 Fix missed overlay: D13 (atomic mirror и/или attribution without «piece still on cell»; no silent skip) — overlay стартует при reveal — SC-BOARD-23/24/28
- [x] 6.3 Queue order: visual land/arrival on catapult cell → then overlay (~1000 ms success / broken 300+300) → then fling/finish travel; same for spectators; push/return as step; already-on-cell no fake step — SC-BOARD-23…26/28, SC-FINISH-19
- [x] 6.4 Board-busy lock covers land-before-overlay + full chain — SC-BOARD-27
- [x] 6.5 `npm run lint` + `npm run typecheck`; обновить client skills / AGENTS blurbs (land→overlay→fling, spectator parity, D13)

## 7. Server+client — paced trap hops + deferred turn (follow-up)

- [x] 7.1 Прочитать `design.md` D3/D14/D15, delta `specs/game/move` (SC-MOVE-90…92) + `game/board` (SC-BOARD-29/30), skills `server/work-with-game`, `server/work-with-rooms`, `client/work-with-game-board`
- [x] 7.2 Server: заменить мгновенный full-chain `resolveCellTraps` на paced pipeline (один hop → presentation budget → эффект → следующий land); общие ms с client; без `presentationDone` — SC-MOVE-90
- [x] 7.3 Server: deferred `maybeAutoEndTurn` + deadline advance до pipeline idle (`pendingTurnAdvance`); push/return/rescue тот же path — SC-MOVE-91/92
- [x] 7.4 Client: queue/grille follow hop-sync; не показывать holding grille на финале во время prior catapult hops; board lock на pipeline — SC-BOARD-26/29/30/27
- [x] 7.5 Server `npm test` (SC-MOVE-90…92 + регресс 78…89); client `npm run lint` + `npm run typecheck`; обновить skills / AGENTS blurbs (paced + deferred turn)

## 8. Server+client — idle re-eval + always finish travel (follow-up)

- [x] 8.1 Прочитать `design.md` D15–D17, delta `specs/game/move` (SC-MOVE-93) + `game/finish` (SC-FINISH-20) + `game/board` (SC-BOARD-31), skills `server/work-with-game`, `server/work-with-rooms`, `client/work-with-game-board`
- [x] 8.2 Server `onTrapPipelineIdle`: если `pendingTurnAdvance` → `advanceTurn`; иначе `maybeAutoEndTurn` + solo exhaustion — SC-MOVE-93
- [x] 8.3 Client: после successful catapult vanish — если piece finished / dest center, всегда finish travel (даже если finish sync после reveal enqueue); цепочка hop→…→центр — анимация каждого hop, finish travel после последнего vanish — SC-FINISH-20, SC-BOARD-31
- [x] 8.4 Server `npm test` (SC-MOVE-93 + регресс 90…92); client `npm run lint` + `npm run typecheck`; обновить skills / AGENTS blurbs (idle re-eval + finish-after-vanish)
