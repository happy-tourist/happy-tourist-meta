## 1. Server — schema, seed, trap on land

- [x] 1.1 Прочитать `design.md` (D1–D12), delta `specs/{lobby/rooms,game/board,game/move,game/pieces}`, skills `server/work-with-schema`, `server/work-with-game`, `server/work-with-rooms`, `server/server-work-with-test` и текущие `MyRoom` / `MyRoomState` / `touristMove` — зафиксировать поля и точки врезки
- [x] 1.2 Schema: `Piece.trapped`; sync collection revealed/holding grille keys; `onCreate` parse `grilleDensity` (few/medium/many → originally 25/45/65; superseded by block 6 → **12/22/35**, default medium) — проверить типами schema / parse helper
- [x] 1.3 На `enterPlaying`: seed hidden grilles на random task cells по %; unit/mocha count для medium = 22 на 48 task — SC-BOARD-16
- [x] 1.4 `handleMove`: после accept −1 step; unspent grille → reveal + `trapped`; occupancy учитывает trapped — mocha: SC-MOVE-51, SC-PIECE-24/26
- [x] 1.5 Reject move/peek для trapped piece; free siblings OK — mocha: SC-MOVE-52/53
- [x] 1.6 Из sibling `happy-tourist-server`: `npm test` — новые SC-BOARD-16 / SC-MOVE-51…53 / SC-PIECE-24/26 зелёные

## 2. Server — rescue, return, all-jail, auto-end

- [x] 2.1 Прочитать delta `specs/game/{move,finish,pieces}`, `design.md` D3–D7, skills `server/work-with-messages`, `server/work-with-game`
- [x] 2.2 Message `rescue`: own + Chebyshev-1 free piece + steps≥1 → −1 step, clear trapped + grille; reject other seat / no adj / no steps — mocha: SC-MOVE-54/55/56
- [x] 2.3 Message `returnFromFinish`: finishPlace 0 + finished side + legal ring cell (incl. corners, no hole/occupancy) → −1 step, unfinish on cell — mocha: SC-MOVE-57/58/59, SC-FINISH-12/14
- [x] 2.4 All-jail: 4 trapped → immediate free + clear 4 holding grilles + place on free starts per side; keep turn/steps/timer; private/event for self warning — mocha: SC-MOVE-60/61, SC-PIECE-25
- [x] 2.5 Auto-end: legal rescue/return = available action — mocha: SC-MOVE-62
- [x] 2.6 Solo: те же правила rescue — mocha: SC-MOVE-64; spent grille не снимает task reward — SC-BOARD-20
- [x] 2.7 Из sibling `happy-tourist-server`: `npm test` — SC-MOVE-54…64 / SC-FINISH-12/14 / SC-PIECE-25 / SC-BOARD-20 зелёные

## 3. Client — create option + store contract

- [x] 3.1 Прочитать `design.md` D1/D10, delta `specs/lobby/rooms`, skills `client/work-with-lobby`, `client/work-with-stores`, `client/colyseus-client`
- [x] 3.2 Lobby create modal: density few/medium/many (labels мало/средне/много), default medium; pass in `createGame` options — SC-LOBBY-13/15
- [x] 3.3 Pinia `game`: mirror `trapped` + revealed grilles; `sendRescue` / `sendReturnFromFinish`; create options typing
- [x] 3.4 Server create parse покрыт тестом SC-LOBBY-14 (mocha options → density); client typecheck после 3.2–3.3
- [x] 3.5 Из sibling `happy-tourist.github.io`: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 4. Client — board UX, strip return, all-jail modal

- [x] 4.1 Прочитать delta `specs/game/{board,move,finish,pieces}`, skills `client/work-with-game-board`, `client/work-with-pages`, `client/work-with-localization`, `client/work-with-styles`
- [x] 4.2 Положить ассет `src/assets/grilles/grille.png`; overlay + drop/rise anim для всех клиентов — SC-BOARD-17/18/19; trapped visible — SC-PIECE-27
- [x] 4.3 Rescue affordance над trapped при adj+steps; lock move/peek UI для trapped; spent cell peekable — SC-MOVE-54 UX, SC-BOARD-20
- [x] 4.4 Strip: return control beside flag; ring highlights; submit return — SC-FINISH-13; SC-MOVE-57/59 UX
- [x] 4.5 All-jail warning modal only own seat (informational) — SC-MOVE-63; i18n keys
- [x] 4.6 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 5. Meta — skills / AGENTS

- [x] 5.1 Обновить server skills (`work-with-game`, `work-with-messages`, `work-with-schema`, `work-with-rooms`) под grille density / trap / rescue / return / all-jail
- [x] 5.2 Обновить client skills (`work-with-lobby`, `work-with-game-board`, `colyseus-client` / stores) под density create, grille overlay, rescue/return
- [x] 5.3 При необходимости — краткие строки в sibling AGENTS Business Entities (без дублирования specs)

## 6. Follow-up — density, anim 1500, leave clear

> Supersedes earlier density 25/45/65 and anim ~320 ms from blocks 1–4; keep prior `[x]` as historical apply. Implement the deltas below.

- [x] 6.1 Server: `GRILLE_DENSITY` / parse → **0.12 / 0.22 / 0.35**; обновить mocha SC-BOARD-16 (medium = **11**), SC-LOBBY-14 (many → **17** на 48)
- [x] 6.2 Server: в `onLeave` перед/при remove seat — clear holding ключей клеток trapped pieces уходящего; unexpected `onDrop` не clear — mocha SC-PIECE-28
- [x] 6.3 Из sibling server: `npm test` — SC-BOARD-16 / SC-LOBBY-14 / SC-PIECE-28 зелёные
- [x] 6.4 Client: `GRILLE_ANIM_MS = 1500` для drop и rise (включая clear по leave) — SC-BOARD-18/19/21
- [x] 6.5 При необходимости — краткие правки skills (`work-with-game`, `work-with-game-board`, density notes) под 12/22/35 и leave-clear
- [x] 6.6 Client: `npm run lint` + `npm run typecheck` по затронутым файлам
