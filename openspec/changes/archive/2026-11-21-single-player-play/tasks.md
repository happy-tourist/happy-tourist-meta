## 1. Server — start full table of one

- [x] 1.1 Перед работой: `server-work-with-rooms`, `server-work-with-test`; `proposal.md` Scope; delta `specs/game/start/spec.md` (SC-START-14); `design.md` D2; `MyRoom.maybeStartCountdown`
- [x] 1.2 Исправить старт при `seated >= maxSeats` (в т.ч. maxSeats=1) без ослабления ready (≥2); mocha SC-START-14; `npm test` в `happy-tourist-server`

## 2. Server — single-player budgets vs solo leftover

- [x] 2.1 Перед работой: `server-work-with-game`, `work-with-messages`; delta `specs/game/move/spec.md` (SC-MOVE-100…106 + Authoritative turn deadline); `design.md` D1/D3/D4; `syncSoloInfiniteMode` / `applyTurnGrant` / `resyncTurnDeadline` / `startTurnDeadline` / `onTurnTimeout` / `maybeAutoEndTurn`
- [x] 2.2 Ввести различение single-player (`maxSeats === 1`) vs solo leftover (`maxSeats ≥ 2 && eligible === 1`): finite +1/+1 и без deadline в одиночной; `onTurnTimeout` не ставит `timeExpired` при maxSeats=1; ∞ + 60с только leftover; mocha SC-MOVE-100/101/102/106; `npm test`

## 3. Server — no-actions +1/+1 refresh

- [x] 3.1 Перед работой: delta SC-MOVE-103/104/105; `design.md` D3 (risk loop + pendingTurnAdvance); `seatHasAvailableActions` / call sites `maybeAutoEndTurn` (в т.ч. post-peek, post-move/rescue/push/return, `onTrapPipelineIdle` F1)
- [x] 3.2 При maxSeats=1 и отсутствии действий — +1 step +1 peek тому же seat; defer = early-return при `openPeek` / `trapPipelineActive` **без** `pendingTurnAdvance`; после peek close / idle F1 — тот же refresh, не `advanceTurn`; mocha SC-MOVE-103/104/105; `npm test`

## 4. Client — HUD / ring / modal gates

- [x] 4.1 Перед работой: `work-with-game-board`, `client-work-with-stores`, `client-work-with-test`; delta `specs/game/presence/spec.md` (SC-PRESENCE-09/19/38…42); `design.md` D5; `game` store `isSoloBudget` / `isSoloBudgetFromSeats` / `canSendEndTurn` / GamePage modal+ring
- [x] 4.2 Одиночная (`maxSeats === 1`): числа не ∞; `showTurnRing` / turn ring chrome — false при maxSeats=1 (не только пустой `turnUntil`); без unlimited-модала (`budgetsInfinite` watch + gate maxSeats); `canSendEndTurn` / chrome — reject maxSeats=1; `isSoloBudgetFromSeats(seats, maxSeats)` — red ring только leftover (`maxSeats ≥ 2` && eligible===1); +N на refresh (существующий finite fall); retarget leftover vitest seeds с `maxSeats: 1` → `≥ 2`; vitest SC-PRESENCE-38…42 (+ регресс 09/19); `npm test` / `npm run lint` / `npm run typecheck` в `happy-tourist.github.io`
