## 1. Server — budgets, removed tiles, move without advance

- [x] 1.1 Прочитать `design.md`, delta `specs/game/{move,board}`, skills `server/work-with-schema`, `server/work-with-messages`, `server/work-with-game`, `server/server-work-with-test` и текущие `MyRoom` / `MyRoomState` / `touristMove` — зафиксировать точки врезки
- [x] 1.2 Schema: sync removed task cells (keys `r,c`); room-private budgets + reward bag; на `playing` преген 28/14/6 — проверить типами schema
- [x] 1.3 Messages `peek`, `peekAnswer`, `endTurn`; `budgets` private send на grant/change/reconnect; `move` тратит step и **не** вызывает advanceTurn — mocha: SC-MOVE-33…36, SC-MOVE-43/44
- [x] 1.4 Peek flow + remove tile + Correct/Wrong; one peek/turn multi; timeout force wrong — mocha: SC-BOARD-07…10, SC-MOVE-39/42
- [x] 1.5 End-turn / auto-end when no actions; grant +1/+1 next seat; solo infinite + no end-turn — mocha: SC-MOVE-37/38/40/41/45
- [x] 1.6 Playable includes removed `*`; helpers legal move/peek — unit/mocha green
- [x] 1.7 Из sibling `happy-tourist-server`: `npm test` — новые/обновлённые SC-MOVE / SC-BOARD зелёные

## 2. Client — store contract

- [x] 2.1 Прочитать delta `specs/game/{move,board,presence}`, `design.md`, skills `client/work-with-stores`, `client/colyseus-client`, `client/work-with-rooms`
- [x] 2.2 Pinia `game`: mirror removed tiles; listen `budgets`; `sendPeek` / `sendPeekAnswer` / `sendEndTurn`; stop assuming move advances turn locally
- [x] 2.3 Из sibling `happy-tourist.github.io`: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 3. Client — Game UX

- [x] 3.1 Прочитать delta `specs/game/{presence,board}`, skills `client/work-with-game-board`, `client/work-with-pages`, `client/work-with-localization`, `client/work-with-styles`
- [x] 3.2 Presence: свои steps/peeks (∞ в соло) + «Завершить ход» только на multi своем ходу; +N anim — SC-PRESENCE-15…18
- [x] 3.3 Board: дыры для removed `*`; eye + модалка «Правильно»/«Неправильно»; solo unlimited modal — SC-BOARD-11…13, SC-PRESENCE-19
- [x] 3.4 i18n ключи модалок/кнопки/∞
- [x] 3.5 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 4. Meta — skills / AGENTS

- [x] 4.1 Обновить `server/work-with-game`, `server/work-with-messages`, `server/work-with-schema` под budgets / peek / endTurn / removed tiles
- [x] 4.2 Обновить `client/work-with-game-board`, `client/colyseus-client` (или stores) под counters / peek / end-turn
- [x] 4.3 При необходимости — краткие строки в sibling AGENTS Business Entities (без дублирования specs)

## 5. Client — keep-focus после move + медленнее +N

- [x] 5.1 `GamePage`: после успешного non-finishing `sendMove` **не** сбрасывать `selectedSide`; после move-anim снова белая рамка + красные targets при steps>0/∞ — SC-MOVE-46
- [x] 5.2 Глаз без повторного select, если keep-focus на живом `*` и peeks позволяют; дыра / не `*` — без глаза — SC-BOARD-14
- [x] 5.3 +N fall: CSS/`BUDGET_FALL_MS` ≈ **2 с** — SC-PRESENCE-20
- [x] 5.4 Обновить `client/work-with-game-board` (keep-focus; +N ~2 с; без ambient peekable)
- [x] 5.5 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 6. Server — incorrect peek KEEP tile

- [x] 6.1 `resolveOpenPeek`: `markTaskRemoved` только при `correct === true`; incorrect (кнопка / timeout / leave / endTurn) — −peek + `peekedThisTurn`, тайл и reward остаются — mocha: SC-BOARD-09, SC-MOVE-42 (и leave/endTurn mid-peek)
- [x] 6.2 Обновить skills `server/work-with-game` / `server/work-with-messages` (и AGENTS при необходимости) — incorrect KEEP
- [x] 6.3 Из sibling `happy-tourist-server`: `npm test` — обновлённые SC-BOARD-09 / SC-MOVE-42 зелёные

## 7. Server — auto-end, multi peek, unlandable holes, solo peeks∞

- [x] 7.1 Auto-end: держать ход при peeks≥1 и unfinished на живом `*`; иначе auto-end как раньше — mocha: SC-MOVE-38, SC-MOVE-47
- [x] 7.2 Снять gate `peekedThisTurn` / лимит 1 peek/ход; peek пока peeks (или solo ∞) и на живом `*` — mocha: SC-MOVE-39, SC-BOARD-08
- [x] 7.3 `touristMove` / validate: landing на `removedTaskKeys` reject; stand на дыре ok; hasLegalMove без дыр — mocha: SC-MOVE-49, SC-BOARD-11/15
- [x] 7.4 Solo: infinite только peeks; steps finite; step-loss (steps=0 ∧ ¬hasLegalPeek) → timeExpired — mocha: SC-MOVE-40/41/45/48 *(solo become-current +1 шаг — блок 9)*
- [x] 7.5 Budgets payload / syncSolo: peeks∞ flag без steps∞; skills `work-with-game` / `work-with-messages` / `work-with-schema` / `server-work-with-test`
- [x] 7.6 Из sibling `happy-tourist-server`: `npm test` — обновлённые SC зелёные

## 8. Client — holes, solo counters/modals, dual end copy

- [x] 8.1 Red targets / legal moves: не предлагать removed holes; фишка на дыре остаётся — SC-BOARD-11/12/15, SC-MOVE-46
- [x] 8.2 Presence: соло ∞ только peeks, steps числом; соло-модалка peeks-unlimited — SC-PRESENCE-16/19
- [x] 8.3 i18n + UX: разные модалки timer-expired vs steps-exhausted — SC-PRESENCE-21, SC-MOVE-45/48
- [x] 8.4 Обновить `client/work-with-game-board`, `work-with-localization` (и stores/AGENTS при необходимости)
- [x] 8.5 Из sibling client: `npm run lint` и `npm run typecheck` — без ошибок по затронутым файлам

## 9. Server — solo become-current +1 step

- [x] 9.1 Прочитать delta `specs/game/move` (SC-MOVE-40/50), `design.md` Decision 5–6, skill `server/work-with-game` — `applyTurnGrant` / leave/finish paths
- [x] 9.2 `applyTurnGrant`: multi → +1/+1; solo become-current → **+1 step only** (после syncSolo peeks∞); already-current→solo без лишнего шага — mocha: SC-MOVE-50, SC-MOVE-40
- [x] 9.3 Обновить skills `server/work-with-game` / `work-with-messages` (и AGENTS при необходимости)
- [x] 9.4 Из sibling `happy-tourist-server`: `npm test` — SC-MOVE-40/50 и смежные соло зелёные
