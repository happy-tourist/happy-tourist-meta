## 1. Server — seed & exclusive traps

- [x] 1.1 Перед работой: `design.md` D1/D6; delta `specs/lobby/rooms` + `specs/game/board` (SC-BOARD-16/80/81/86); skills `server/work-with-game`, `server/work-with-rooms`; текущий `seedHiddenGrilles` / `seedHiddenCatapults` / density types
- [x] 1.2 Убрать/игнорировать create-time density; сид `g=min(round(n*0.12),n)`, `c=min(round(n*0.15),n-g)`, rest Q&A; clear ловушки → empty cell — mocha SC-BOARD-80/81/86 и SC-LOBBY-41
- [x] 1.3 Адаптировать/удалить mocha на old density few/medium/many и catapult+grille share; `npm test` в `happy-tourist-server` по затронутым suite

## 2. Server — turn economy, knowledge, rescue, retire push/timer

- [x] 2.1 Перед работой: `design.md` D2–D5/D7; delta `specs/game/move` (SC-MOVE-110…122); skills `server/work-with-messages`, `server/work-with-schema`, `server/work-with-game`
- [x] 2.2 Private `knowledge` Map+`client.send` (hard-cut `budgets`); public `Piece.remainingTurns`; retire turn deadline / `timeExpired` from clocks; `boardActionUsed` / one move|return; free rescue; reject `push`/`endTurn`/client `peek`; after trap-pipeline idle → open-on-land Q&A or `advanceTurn`; exclusive trap resolve (no dual-trap); fling chain; grille tick on become-current; all-jail → advance — mocha SC-MOVE-51/60/61/110…120 + retired density/push/solo/timer suites
- [x] 2.3 Correct/wrong task → knowledge / open digit; reopen-by-land; no solo leftover ∞ — mocha SC-MOVE-116/121/122 и регресс без SC-MOVE-25/40/78/79/push
- [x] 2.4 `npm test` в `happy-tourist-server` (полный прогон затронутых room/move/board тестов)

## 3. Server — finish knowledge & penultimate cut

- [x] 3.1 Перед работой: `design.md` D8; delta `specs/game/finish` (SC-FINISH-30…33, SC-FINISH-03/21); skill `server/work-with-game`
- [x] 3.2 Transfer knowledge on piece finish; full-finish/`cut` set `finishPlace=1` without place UX; penultimate cut burns carried→0 + `finished=true` when `maxSeats ≥ 2`; single-player skip cut; strip/return affordance without steps — mocha SC-FINISH-09/10/13/16/30…34
- [x] 3.3 `npm test` в `happy-tourist-server` по finish/cut suite

## 4. Client — lobby create without density

- [x] 4.1 Перед работой: delta `specs/lobby/rooms` SC-LOBBY-40/41; `design.md` D1; skills `client/work-with-lobby`, `client/work-with-stores` (game create options)
- [x] 4.2 Убрать grille/catapult density из create UI и create payload; vitest SC-LOBBY-40; `npm test` / `npm run lint` / `npm run typecheck` в `happy-tourist.github.io` по затронутым

## 5. Client — board open-on-land, badges, no peek/push

- [x] 5.1 Перед работой: delta `specs/game/board` SC-BOARD-82…87; `design.md` D3/D4/D5/D9; skills `client/work-with-game-board`, `client/colyseus-client`, `client/work-with-styles` (board)
- [x] 5.2 Убрать peek/push affordances; open modal from land sync (`peekPlace`/`peekSubmit`/`peekClose`; не слать `peek`); map public `Piece.remainingTurns` on piece sync; yellow knowledge circle BR board+strip (**own** only from private `knowledge.pieces`); red grille counter TR board+strip (`remainingTurns`); empty cell after trap/correct; локальный board-action spent после `sendMove`/`sendReturnFromFinish` === true (clear на смене current; optional silent-reject recover) — vitest GameBoardWire / board SC-BOARD-84/85/86/87 + SC-MOVE-110 UX
- [x] 5.3 `npm test` + `npm run lint` + `npm run typecheck` в client по board/game

## 6. Client — presence HUD, focus, finish modals

- [x] 6.1 Перед работой: delta `specs/game/presence` SC-PRESENCE-50…56 + `specs/game/finish` SC-FINISH-31…34; `design.md` D8/D9; skills `client/work-with-pages` (game-page), `client/work-with-stores` (game), `client/work-with-localization`
- [x] 6.2 Private `onMessage('knowledge')` вместо `budgets`; avatar knowledge counter + +N только при росте `delivered`; focus в бывший end-turn slot @3 o’clock (36×36); убрать end-turn / старый BR focus / turn ring / dual turn+grace / solo unlimited & solo-end (`soloEndModalKind`) / steps-peeks / place-badge / «Вы N-й!»; **одна** knowledge modal на `finishPlace` 0→1 (`delivered`); i18n — vitest SC-PRESENCE-06/10/14/30…33/37/50…56, SC-FINISH-31…34
- [x] 6.3 `npm test` + `npm run lint` + `npm run typecheck` в `happy-tourist.github.io`
