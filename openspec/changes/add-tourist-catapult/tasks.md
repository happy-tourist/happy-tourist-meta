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
