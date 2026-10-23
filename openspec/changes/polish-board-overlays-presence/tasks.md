## 1. Server — catapult presentation budgets

- [x] 1.1 Перед работой: `design.md` D2; delta `specs/game/board` SC-BOARD-23/24; skill `.agents/skills/server/work-with-game`; `MyRoom.ts` exports `CATAPULT_ANIM_MS` / `CATAPULT_BROKEN_PRESENTATION_MS` / `resetTrapPresentationBudgets` (+ `setTrapPresentationBudgetsForTests`)
- [x] 1.2 Выставить production defaults: `CATAPULT_ANIM_MS = 1500`, `CATAPULT_BROKEN_PRESENTATION_MS = 450 + 450 + 300` (= **1200**; и те же в `resetTrapPresentationBudgets`; `MOVE_ANIM_MS` / `GRILLE_ANIM_MS` без смены) — verify: значения в исходниках; mocha `setTrapPresentationBudgetsForTests` API без регрессии
- [x] 1.3 `npm test` (затронутые room/trap suites) в `happy-tourist-server` — verify: зелёные

## 2. Client — overlay contrast + catapult timings

- [x] 2.1 Перед работой: `design.md` D1/D2; delta `specs/game/board` SC-BOARD-23/24/58; skill `.agents/skills/client/work-with-game-board`; `GamePage.vue` `CATAPULT_*` + `.catapult-overlay` / `.grille-overlay` / `.my-tourist-chrome-grille`
- [x] 2.2 Константы: `CATAPULT_ANIM_MS = 1500`, broken hold **450**, vanish **300**; CSS `--catapult-anim-ms` fallback **и** `.catapult-overlay--vanish` (сейчас hardcoded `200ms`) → **300** (или `--catapult-broken-vanish-ms`) — verify: TS + CSS duration sync
- [x] 2.3 `filter: drop-shadow(...)` per design D1 starter на board catapult/grille и strip chrome-grille (без opaque underlay) — verify: SC-BOARD-58 (assert filter string in SFC / shared token; **не** полагаться на jsdom `getComputedStyle` для scoped CSS)
- [x] 2.4 Vitest затронутых Game board / catapult suites + `npm run lint` / `npm run typecheck` / `npm test` (затронутые) в `happy-tourist.github.io` — verify: зелёные

## 3. Client — presence clock (say 12 / focus 9 / skip 3)

- [x] 3.1 Перед работой: `design.md` D3/D5; delta `specs/game/presence` SC-PRESENCE-14/30/33 + `specs/game/say` SC-SAY-07; skills `work-with-game-board` / `focus.md` (не переименовывать historic scenario/requirement titles в delta)
- [x] 3.2 CSS: `.say-affordance` top-center (12, `top: -40px` + `left: 50%` + `translateX(-50%)`), `.focus-affordance` left-center (9), `.end-turn-affordance` right-center без смены; clear stale `right` on say/focus; say-picker без clip (SC-SAY-15); ready TL не пересекает say — verify: styles match design D3
- [x] 3.3 Vitest presence placement (SC-PRESENCE-14/30/33, SC-SAY-07): заменить «between say and end-turn» / `focusTopPct = 32` на clock-axis (12/9/3); assert say center-x ≈ 50%, focus left edge, end-turn right; при co-visible ready+say — hit boxes не пересекаются (D3 `top: -40px`); DOM-order alone недостаточен — verify: `npm test` затронутых suites зелёные

## 4. Skills notes

- [x] 4.1 Перед работой: `design.md` D4; текущие `work-with-game-board` (+ `focus.md`), `work-with-game` (server)
- [x] 4.2 Client skill: timings 1500 / broken ×1.5; drop-shadow; clock 12/9/3 — verify: notes без устаревших «1000 ms catapult» / «say top-right» / «focus between say and end-turn on the right»
- [x] 4.3 Server skill: presentation budget числа 1500 / 1200 broken — verify: совпадают с `MyRoom` defaults
