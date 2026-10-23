## 1. Client — wire board-action icons

- [x] 1.1 Перед работой: `design.md` D1–D4; delta specs `game/board|move|finish|presence|say`; skills `work-with-game-board` (+ `focus.md`), `work-with-styles/board.md`; сверить наличие `src/assets/board/{peek,push,return,flag,focus,end-turn,rescue,say}.png` в client
- [x] 1.2 В `GamePage` заменить Material `q-icon` на `<img>` illustrated icons для peek / rescue / push / return / flag / focus / end-turn / say; убрать solid colored circular chrome; hit-area и clock offsets без изменений на этом шаге; budgets + ready не трогать — verify: affordances рендерят PNG без цветного fill
- [x] 1.3 Обновить Vitest под SC-BOARD-13/14, SC-MOVE-74/77, SC-FINISH-09/13, SC-PRESENCE-17/30/33, SC-SAY-07/15 — verify: `npm test` (релевантные suites) в client
- [x] 1.4 Прогнать `npm run lint` и `npm run typecheck` в client — verify: exit 0

## 2. Meta — skills notes (wire)

- [x] 2.1 Перед работой: сверка code ↔ `design.md` D1–D4; skills `work-with-game-board` / `board.md` / game-page
- [x] 2.2 Обновить notes: illustrated icons, no colored circular chrome — verify: skills ↔ design ↔ GamePage

## 3. Server — peekClose reason

- [x] 3.1 Перед работой: `design.md` D7; delta `game/board` SC-BOARD-70, `game/move` SC-MOVE-42; skills `work-with-messages`, `work-with-game`; сверить `resolveOpenPeek` / `onTurnTimeout` / peekSubmit в `MyRoom.ts`
- [x] 3.2 Расширить `resolveOpenPeek` / `peekClose` broadcast полем `reason: 'wrong' | 'timeout'` (correct omit/ignore; timeout callers → `timeout`; wrong submit / leave force → `wrong`) — verify: mocha assert `peekClose.reason` на wrong vs timeout (SC-BOARD-70 / SC-MOVE-42)
- [x] 3.3 Прогнать `npm test` (релевантные peek/timeout suites) в server — verify: exit 0

## 4. Client — affordance scale + circumference

- [x] 4.1 Перед работой: `design.md` D5–D6; delta `game/board|move|finish|presence|say`; skills `work-with-game-board` (+ `focus.md`), `work-with-styles/board.md`; сверить CSS say/focus/end-turn / `.board-action-icon` / return / flag в `GamePage`
- [x] 4.2 Увеличить glyph ~×2 и hit (~48–56 board/strip return, ~56–64 presence) для peek/rescue/push/return/say/focus/end-turn; **flag размер не менять**; presence hit-centers на окружность маркера (say 12 / focus 9 / end-turn 3 на этом шаге); transparent chrome сохранить — verify: Vitest clock/CSS offsets + размерные asserts
- [x] 4.3 Прогнать релевантные client suites + `npm run lint` + `npm run typecheck` — verify: exit 0

## 5. Client — peek status flash

- [x] 5.1 Перед работой: `design.md` D8; delta `game/presence` SC-PRESENCE-23/36, `game/board` SC-BOARD-43/44; skills `work-with-stores` (game), `work-with-pages` shell/game, `work-with-localization`; сверить `App.vue` `statusLabel` и `_onPeekCloseMessage`
- [x] 5.2 Store: читать `peekClose.correct` + `reason`; flash 10 с (`correct` → Правильно!, `wrong` → Ошибка!, `timeout` → Не успел!); App header слот `game-status` — verify: Vitest store +/or App status flash (SC-PRESENCE-36)
- [x] 5.3 Добавить i18n ключи RU «Правильно!» / «Ошибка!» / «Не успел!» — verify: ключи используются в status flash
- [x] 5.4 Прогнать релевантные suites + `npm run lint` + `npm run typecheck` — verify: exit 0

## 6. Meta — skills notes (scale / flash)

- [x] 6.1 Перед работой: сверка code ↔ `design.md` D5–D9 после блоков 3–5; skills `work-with-game-board` / `board.md` / game-page/shell / `work-with-messages`
- [x] 6.2 Обновить notes: ×2 sizes, circumference 12/9/3, `peekClose.reason`, header flash 10 с — verify: skills ↔ design ↔ runtime

## 7. Client — clearance + focus under peeks + say-picker

- [x] 7.1 Перед работой: `design.md` D10–D12; delta `game/board` SC-BOARD-13/71, `game/move` push/rescue, `game/finish` return, `game/presence` SC-PRESENCE-30/33/37, `game/say` SC-SAY-15; skills `work-with-game-board` (+ `focus.md`), `work-with-styles/board.md`; сверить affordance CSS, budgets, `.say-picker` в `GamePage`
- [x] 7.2 Поднять board/strip affordances: якорь верх клетки/слота, `translate(-50%, -100%)` (+ ~8 px gap) — hit целиком над спрайтом (peek/rescue/push/return); flag не трогать — verify: Vitest CSS/transform asserts (SC-BOARD-13/71, SC-MOVE-*, SC-FINISH-13)
- [x] 7.3 Перенести focus **под peeks** в own budgets; убрать focus с окружности @9; say @12 и end-turn @3 без изменений — verify: Vitest DOM/CSS (SC-PRESENCE-30/33/37)
- [x] 7.4 Сместить `.say-picker` **вправо**, чтобы левый край пресетов не обрезался — verify: Vitest/CSS + SC-SAY-15
- [x] 7.5 Прогнать релевантные client suites + `npm run lint` + `npm run typecheck` — verify: exit 0

## 8. Client — finish travel longer + race

- [x] 8.1 Перед работой: `design.md` D13; delta `game/finish` / `game/board` catapult→center; skills `work-with-game-board`; сверить `MOVE_ANIM_MS` / `FINISH_FADE_MS`, `finishAnimFrom`, `catapultDeferFinishKeys`, `releaseDeferredFinishAfterVanish`, finish watcher в `GamePage`
- [x] 8.2 Удлинить finish slide ~**550 ms** и fade ~**300 ms** (delay = slide) для всех enter-center paths: ordinary move (в т.ч. клик по center), push→center, catapult→center; overlay catapult (~1500 ms) не трогать — verify: Vitest/source timing asserts
- [x] 8.3 Audit/fix race: finished sync не роняет piece без travel+fade (`boardPieces` / deferred pin / `disappearingKeys`); catapult path всегда paint `finishAnimFrom` → center — verify: Vitest deferred/race path (SC-FINISH-19/20 family)
- [x] 8.4 Прогнать релевантные suites + `npm run lint` + `npm run typecheck` — verify: exit 0

## 9. Meta — skills notes (follow-up)

- [x] 9.1 Перед работой: сверка code ↔ `design.md` D10–D14 после блоков 7–8; skills `work-with-game-board` / `board.md` / `focus.md`
- [x] 9.2 Обновить notes: clearance над спрайтом; focus under peeks; say-picker вправо; finish travel ~550/300 + deferred race — verify: skills ↔ design ↔ runtime

## 10. Client — crop icons + restore sizes + focus under end-turn

- [x] 10.1 Подрезать `src/assets/board/*.png` по opaque bbox (круглые → квадрат)
- [x] 10.2 Вернуть размеры: glyph ~18; board/strip hit ~32; presence hit ~36 (снять ×2)
- [x] 10.3 Focus под end-turn на угол квадрата аватара (`calc(50% + 36px)`); не в budgets
- [x] 10.4 Vitest + delta specs + skills notes — verify: suites exit 0
