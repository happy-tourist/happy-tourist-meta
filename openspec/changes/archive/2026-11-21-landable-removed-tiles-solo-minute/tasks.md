# Tasks: landable-removed-tiles-solo-minute

Порядок: server (правила + room) → client (hints / HUD / i18n). Подход — `design.md`.

## 1. Server — landable removed + unit

- [x] 1.1 Перед работой: skills `server-work-with-structure`, `work-with-game`, `server-work-with-test`; `design.md` D1; delta `specs/game/move` (SC-MOVE-49/71) + `specs/game/board` (SC-BOARD-11/15); `src/game/touristMove.ts` + `test/touristMove.test.ts`
- [x] 1.2 Убрать reject `removedKeys` из validate move / push / return / catapult fling candidates; layout never-playable по-прежнему reject — проверить unit (SC-MOVE-49 accept; SC-MOVE-07 still reject; push/return/fling на removed OK)
- [x] 1.3 `npm test` в `happy-tourist-server` (хотя бы touristMove) — зелёный

## 2. Server — solo ∞ budgets + 60s + room tests

- [x] 2.1 Перед работой: skills `work-with-messages`, `work-with-schema`, `server-work-with-test`; `design.md` D2–D3; delta `specs/game/move` (SC-MOVE-29/40/45/50/64/99; **rewrite** 48/97/98 — ID сохранить, не OpenSpec REMOVED); `src/rooms/MyRoom.ts` budgets / `SOLO_BUDGET_SECONDS` / exhaustion gate
- [x] 2.2 `SOLO_BUDGET_SECONDS = 60`; соло → `budgets.infinite = true` (= ∞ peeks **и** ∞ steps, design D2); `canSpendStep` / gates steps≥1 учитывают `infinite`; не декрементить steps при `infinite`; become-current solo без +1 step/+1 peek; удалить `maybeSoloStepsExhausted` + `evaluateSoloStepsExhaustedForTests` и все вызовы
- [x] 2.3 Обновить / добавить mocha: SC-MOVE-29/40/45/50/64/99; **переписать** SC-MOVE-48/97/98 (без time-lock / без exhaustion); поправить связанные assertions (300→60; `infinite` ⇒ оба бюджета); ориентироваться на **тела/THEN** delta (заголовки Requirement/Scenario могут совпадать с main и звучать устаревше — design D5)
- [x] 2.4 `npm test` в `happy-tourist-server` — зелёный

## 3. Client — landable hints + board

- [x] 3.1 Перед работой: skills `work-with-game-board`, `client-work-with-test`; `design.md` D4; delta `specs/game/board` SC-BOARD-11/15; GamePage `isLandableCell` / board helpers vs `removedTaskKeys`
- [x] 3.2 Убрать reject removed из landable helper; red hints / push / return targets включают removed; layout never-playable — нет; vitest SC-BOARD-15 (hint includes removed)
- [x] 3.3 `npm test` (затронутые) + `npm run lint` в `happy-tourist.github.io`

## 4. Client — solo HUD / modal / end copy

- [x] 4.1 Перед работой: skills `work-with-localization`, `client-work-with-test`, `work-with-pages`; `design.md` D4; delta `specs/game/presence` SC-PRESENCE-09/16/19/21/35; `soloEndModalKind` / i18n `soloUnlimitedModal` / budgets display / red ring
- [x] 4.2 Store/UI (design D4): `stepsBudgetDisplay` + peeks → ∞ при `budgetsInfinite`; skip steps `+N` fall when infinite; **все** gates `game.steps <= 0` (`legalTargets`, return/push/rescue, watch `returningPieceId`) → `!budgetsInfinite && steps <= 0`; i18n `soloUnlimitedModal` → «Вы один. Ходы и просмотры безлимитны. У вас 1 минута.»; убрать ветку steps-exhausted modal / `inferSoloEndModalKind` `'steps'` (timer-only); заменить `isSoloBudgetSeconds(===300)` / getter `isSoloBudget` на count eligible seats (`finishPlace === 0` ∧ `!timeExpired`) для red ring SC-PRESENCE-09 (соло и multi оба 60 с; spectators видят seats)
- [x] 4.3 Vitest: SC-PRESENCE-16/19; **переписать** SC-PRESENCE-21/35 (ID сохранить; timer-only / no steps-exhausted copy); red-ring: 1 eligible → red, ≥2 → blue (не через `turnBudgetSeconds`)
- [x] 4.4 `npm test` + `npm run lint` + `npm run typecheck` в `happy-tourist.github.io`
