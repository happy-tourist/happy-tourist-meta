## 1. Server — peekClose reason

- [x] 1.1 Перед работой: `design.md` D3; delta `game/board` SC-BOARD-70, `game/move` SC-MOVE-42; skills `work-with-messages`, `work-with-game`; сверить `resolveOpenPeek` / `onTurnTimeout` / peekSubmit в `MyRoom.ts`
- [x] 1.2 Расширить `resolveOpenPeek` / `peekClose` broadcast полем `reason: 'wrong' | 'timeout'` (correct omit/ignore; timeout callers → `timeout`; wrong submit / leave force → `wrong`); KEEP/remove поведение без изменений — verify: mocha assert `peekClose.reason` на wrong vs timeout (SC-BOARD-70 / SC-MOVE-42)
- [x] 1.3 Прогнать `npm test` (релевантные peek/timeout suites) в server — verify: exit 0

## 2. Client — affordance scale + circumference

- [x] 2.1 Перед работой: `design.md` D1–D2; delta `game/board|move|finish|presence|say`; skills `work-with-game-board` (+ `focus.md`), `work-with-styles/board.md`; сверить текущие CSS say/focus/end-turn / `.board-action-icon` / return / flag в `GamePage`
- [x] 2.2 Увеличить glyph ~×2 и hit (~48–56 board/strip return, ~56–64 presence) для peek/rescue/push/return/say/focus/end-turn; **flag размер не менять**; presence hit-centers на окружность маркера 12/9/3 (D2); transparent chrome сохранить — verify: Vitest clock/CSS offsets + размерные asserts где уже смотрят affordances (SC-BOARD-13, SC-MOVE-74/77, SC-FINISH-13, SC-PRESENCE-17/30/33, SC-SAY-07/15)
- [x] 2.3 Прогнать релевантные client suites + `npm run lint` + `npm run typecheck` — verify: exit 0

## 3. Client — peek status flash

- [x] 3.1 Перед работой: `design.md` D4; delta `game/presence` SC-PRESENCE-23/36, `game/board` SC-BOARD-43/44; skills `work-with-stores` (game), `work-with-pages` shell/game, `work-with-localization`; сверить `App.vue` `statusLabel` и `_onPeekCloseMessage`
- [x] 3.2 Store: читать `peekClose.correct` + `reason`; выставлять flash 10 с (`correct` → Правильно!, `wrong` → Ошибка!, `timeout` → Не успел!); App header тот же слот `game-status` показывает flash с цветом, затем обычный status — verify: Vitest store +/or App status flash (SC-PRESENCE-36)
- [x] 3.3 Добавить i18n ключи RU copy «Правильно!» / «Ошибка!» / «Не успел!» — verify: ключи используются в status flash
- [x] 3.4 Прогнать релевантные suites + `npm run lint` + `npm run typecheck` — verify: exit 0

## 4. Meta — skills notes

- [x] 4.1 Перед работой: сверка code ↔ `design.md` D1–D4 после блоков 1–3; skills `work-with-game-board` / `board.md` / game-page/shell / `work-with-messages`
- [x] 4.2 Обновить notes: ×2 sizes, circumference 12/9/3, `peekClose.reason`, header flash 10 с — verify: skills ↔ design ↔ runtime
