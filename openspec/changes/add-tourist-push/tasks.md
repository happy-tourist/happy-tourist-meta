## 1. Server — push rules + handler

- [x] 1.1 Прочитать `design.md` (D1–D3, D7–D8), delta `specs/game/move/spec.md`, skills `server/work-with-game`, `server/work-with-messages`, `server/server-work-with-test` и аналоги `handleRescue` / `validateTouristMove` — зафиксировать payload и точки врезки
- [x] 1.2 `touristMove.ts`: `farSideCell`, `validateTouristPush`, `hasLegalPush` / list helpers — unit или mocha на геометрию SC-MOVE-67 и reject hole/occupied/trapped SC-MOVE-68/71
- [x] 1.3 `MyRoom`: `onMessage('push')` → `handlePush` (−1 step, relocate target, finish/trap side-effects, no turn advance) — mocha: SC-MOVE-66/69/70/72
- [x] 1.4 Auto-end / `hasAvailableActions`: legal push = available — mocha: SC-MOVE-73 (и согласованность с SC-MOVE-62)
- [x] 1.5 Из sibling `happy-tourist-server`: `npm test` — SC-MOVE-66…73 зелёные

## 2. Client — store + board push UX

- [x] 2.1 Прочитать `design.md` D4–D6, delta `specs/game/move` (SC-MOVE-74/75), skills `client/colyseus-client`, `client/work-with-stores`, `client/work-with-game-board`
- [x] 2.2 Pinia `game.sendPush(pusherSide, targetSessionId, targetSide, row, col)` — typecheck / вызов room.send
- [x] 2.3 GamePage: push affordances над targets выбранного free pusher (turn + steps≥1); клик → sendPush; approach/back + target travel; keep `selectedSide` — SC-MOVE-74/75
- [x] 2.4 i18n `game.pushAffordance` (RU); из sibling client: `npm run lint` и `npm run typecheck` по затронутым файлам

## 3. Client — return strip icon (no modal)

- [x] 3.1 Прочитать delta `specs/game/finish` (SC-FINISH-13/16), `design.md` D5, skills `client/work-with-pages`, `client/work-with-localization`
- [x] 3.2 Удалить return-confirm dialog; зелёная иконка над finished strip tourist при `canReturn`; клик → return-mode; slot body finished не стартует return — SC-FINISH-13/16
- [x] 3.3 Почистить i18n ключи modal, если больше не нужны; `npm run lint` + `npm run typecheck`

## 4. Meta — skills

- [x] 4.1 Обновить server skills (`work-with-game`, `work-with-messages`) — message `push`, auto-end
- [x] 4.2 Обновить client skills (`work-with-game-board`, `work-with-pages`, `colyseus-client` / stores, localization) — push icons, return icon без modal
- [x] 4.3 При необходимости — краткие строки в sibling AGENTS Business Entities (без дублирования specs)

## 5. Client — affordance polish + end-turn + push finish anim

- [x] 5.1 Прочитать `design.md` D5/D9/D10, delta `specs/game/move` (SC-MOVE-74/76/77), `game/board` (SC-BOARD-13), `game/presence` (SC-PRESENCE-17/25), `game/finish` (SC-FINISH-17); skills board/pages/styles
- [x] 5.2 GamePage CSS: peek / rescue / push — top-center над piece (общий якорь как return strip) — SC-MOVE-74/77, SC-BOARD-13
- [x] 5.3 Убрать `.end-turn-dock` / текстовый btn; icon `skip_next` right-center на own avatar; клик → endTurn сразу; solo без контрола — SC-PRESENCE-17/18/25
- [x] 5.4 Push→center: travel с pre-push клетки + disappear как move finish (D10) — SC-MOVE-76 / SC-FINISH-17
- [x] 5.5 Обновить client skills (board/pages/styles/localization aria); `npm run lint` + `npm run typecheck`
