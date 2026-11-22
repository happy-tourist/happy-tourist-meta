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

## 7. Server polish — timer, 15% grille, all-jail wait, outcome

- [x] 7.1 Перед работой: `design.md` D1/D3/D7/D8; delta `specs/game/move` (SC-MOVE-60/61/112/119/123), `specs/game/board` (SC-BOARD-80/70), `specs/game/finish` (outcome); skills `server/work-with-game`, `server/work-with-messages`
- [x] 7.2 Сид grille **15%** / catapult **15%** (`g=min(round(n*0.15),n)`, `c=min(round(n*0.15),n-g)`); mocha SC-BOARD-80/81
- [x] 7.3 Вернуть turn deadline **60s** + `turnUntil`; timeout открытой Q&A → `peekClose` incorrect + `reason:'timeout'` + advance; без permanent leftover `timeExpired` — mocha SC-MOVE-112/123 / SC-BOARD-70
- [x] 7.4 All-jail: **не** teleport / не `allJailWarning`; pieces остаются trapped; после settle → advance — mocha SC-MOVE-60/61/63/119
- [x] 7.5 Private `knowledge.outcome` `'finish'|'cut'` на full finish / penultimate cut; mocha SC-FINISH-31/32/34
- [x] 7.6 `npm test` в `happy-tourist-server` по затронутым suite

## 8. Client polish — strip knowledge, toast, ring, z-index, finish anim

- [x] 8.1 Перед работой: `design.md` D9/D10; delta `specs/game/presence` (SC-PRESENCE-36/52/54), `specs/game/board` (SC-BOARD-85/88), `specs/game/finish` (SC-FINISH-34/35); skills `client/work-with-game-board` / `knowledge.md`, `work-with-styles`, `work-with-localization`
- [x] 8.2 Carried knowledge **только strip** TR + `mdi-lightbulb-on`; убрать board yellow badge; avatar lightbulb у delivered — vitest SC-PRESENCE-52
- [x] 8.3 Red grille badge `z-index` выше grille overlay (board + strip) — vitest/SC-BOARD-85
- [x] 8.4 Включить Quasar `Notify`; correct/wrong/timeout → toast top-center; header-only flash недостаточен — vitest SC-PRESENCE-36 / SC-BOARD-88
- [x] 8.5 Вернуть turn countdown ring (60s) + dual ring offline current — vitest SC-PRESENCE-10/54
- [x] 8.6 Две i18n формулировки knowledge modal по `outcome` finish vs cut — vitest SC-FINISH-31/32/34
- [x] 8.7 Фикс finish travel: клик по центру и catapult→центр всегда slide+fade — vitest SC-FINISH-35
- [x] 8.8 Убрать/инертить all-jail start-reset modal; `npm test` + `lint` + `typecheck` в client

## 9. Playtest follow-up — all-jail skip toast, icons, finish travel, cut copy, catapult 20%

- [x] 9.1 Перед работой: `design.md` D1/D7/D8/D9/D10; delta `specs/game/board` (SC-BOARD-80/81), `specs/game/move` (SC-MOVE-124/125), `specs/game/presence` (SC-PRESENCE-50/52/57), `specs/game/finish` (SC-FINISH-34/35); skills server `work-with-game`/`work-with-messages`, client `work-with-game-board`/`work-with-localization`/`work-with-stores`
- [x] 9.2 Server: сид catapult **20%** (`c=min(round(n*0.20),n-g)`), grille остаётся **15%**; mocha SC-BOARD-80/81
- [x] 9.3 Server: skip turn whenever seat still all-trapped after become-current tick **and** `countEligible() ≥ 2`; broadcast room skip signal; **no** toast/skip spam when eligible < 2 (end-game / solo wait) — mocha SC-MOVE-124/125 + regress SC-MOVE-60/61/119
- [x] 9.4 Client: room skip → Notify top-center «Все туристы в решётке — ход пропущен» **всем**; без start-reset modal — vitest SC-PRESENCE-57
- [x] 9.5 Client: заменить `mdi-lightbulb-on` → Material `lightbulb` у аватара и strip; убрать пустой gap — vitest SC-PRESENCE-50/52
- [x] 9.6 Client: cut i18n «которые успели» (не «что успели») — vitest SC-FINISH-34
- [x] 9.7 Client: починить finish travel (первый турист / click→центр не snap-vanish); усилить тест SC-FINISH-35 beyond source-assert
- [x] 9.8 `npm test` + `lint` + `typecheck` в client; `npm test` в server по затронутым suite
