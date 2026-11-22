## 0. Server — no product work

- [x] 0.1 Confirm: room `tourist` finish / `returnFromFinish` / sync `finished`+coords authority stays as-is (proposal Scope + design Non-Goals / D1 alt rejected) — verify: no server tasks below; no deploy of `happy-tourist-server` for this change

## 1. Client — travel clip core (finish + return)

- [x] 1.1 Перед работой: design.md D1–D4; delta `specs/game/finish` (SC-FINISH-15/19/20/35/36/37); skill `.agents/skills/client/work-with-game-board` (+ client AGENTS game board); текущие finish/return paths в Game board (seed/`from`/defer/disappearing)
- [x] 1.2 Ввести presentation-owned travel clip (`kind` finish|return, from/to, phases hold→travel→fade; return без finish-fade) и встроить в `boardPieces` / `pieceStyle` (`--prow`/`--pcol`) / `.piece--disappearing`+suppress / `hasVisualBoardPresentation`+busy по design D1/D4 — verify: фигура с активным finish clip остаётся на доске при sync `finished`+center (SC-FINISH-36); visibility finished = D3 (a) clip **или** (b) catapult-owned pre-clip (SC-FINISH-37)
- [x] 1.3 Старт clip: own click→center (`to` = center target); catapult after vanish→center (`from` = catapult cell); observed чужой finish (lastKnown mirror); return nearest-center→ring (D2); idempotent per pieceKey — verify: SC-FINISH-19/20 порядок vanish→travel; early `finished` во время overlay не снимает фигуру (D3 b / SC-FINISH-37); return не snap на кольцо
- [x] 1.4 Удалить reconstruct-стек по D3 pinned (`finishAnimFrom`+rAF clear, `disappearingKeys`/`catapultDefer` как истину finish-fade/visibility **вне** catapult-owned pre-clip, `finishTravelFrom` helper + его unit-тесты, finish-seed через boardActionPre); lastKnown mirror оставить только для observed `from`; pin/visual сохранить для center hop pre-clip и non-center fling — verify: нет обязательной зависимости finish travel от double-rAF clear; catapult-owned finished остаётся видимым до старта clip (SC-FINISH-37)

## 2. Client — tests + meta канон

- [x] 2.1 Перед работой: client-work-with-test; Traceability SC-FINISH-15/35/36; существующие GameBoardWire / finishTravelFrom тесты
- [x] 2.2 Behavioral vitest: finish с non-center origin требует travel `from` (SC-FINISH-35/36); early `finished` во время catapult overlay не drop (SC-FINISH-37); return clip from nearest center (SC-FINISH-15); заменить/удалить unit-тесты `finishTravelFrom`; **обновить/снять** `GameBoardWire` source-asserts старого reconstruct-стека (`finishAnimFrom`+rAF / `disappearingKeys`+`catapultDefer` как истину) — SFC regex alone не cover SC-FINISH-35 — verify: `npm test` (client) падает без `from` / проходит с clip; нет красных asserts на удалённый стек
- [x] 2.3 Обновить meta skill `work-with-game-board` / client AGENTS: канон travel clip вместо `finishTravelFrom` / `finishAnimFrom`+rAF — verify: ссылки не указывают на удалённый helper как истину
- [x] 2.4 Client `npm run lint` + `npm run typecheck` + `npm test` — verify: все три зелёные
