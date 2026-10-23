## 0. Server

- [x] 0.1 Нет server-задач: change **client-only**; tourist-room messages `peek` / `rescue` / `push` / `returnFromFinish` / `endTurn` / `say`, schema / HTTP / DB — **без изменений**; focus — локальный UI — verify: этот change не правит `happy-tourist-server`

## 1. Client — wire board-action icons

- [x] 1.1 Перед работой: `design.md` D1–D4; delta specs `game/board|move|finish|presence|say`; skills `work-with-game-board` (+ `focus.md`), `work-with-styles/board.md`; сверить наличие `src/assets/board/{peek,push,return,flag,focus,end-turn,rescue,say}.png` в client
- [x] 1.2 В `GamePage` заменить Material `q-icon` на `<img>` illustrated icons для peek / rescue / push / return / flag (`.my-tourist-finish-icon`) / focus / end-turn / say; убрать solid colored circular chrome на кнопках-affordance (D1); flag — только glyph swap (PNG as-is, без Material `color` tint); hit-area ~32–36px и clock offsets без изменений; budgets + ready не трогать; `alt=""` на img в кнопках, aria-label как сейчас — verify: affordances рендерят PNG без цветного fill
- [x] 1.3 Обновить Vitest под SC-BOARD-13/14, SC-MOVE-74/77, SC-FINISH-09/13, SC-PRESENCE-17/30/33, SC-SAY-07/15 (D4: clock offsets сохранить; CSS — нет solid fill / medallion shadow; DOM — `<img>` где смотрят glyph) — verify: `npm test` (релевантные suites) в client
- [x] 1.4 Прогнать `npm run lint` и `npm run typecheck` в client — verify: exit 0

## 2. Client — skills notes (verify after wire)

- [x] 2.1 Перед работой: meta skills уже выровнены под D1/D3 (`work-with-game-board` + `focus.md` / `peek.md`, `work-with-styles/board.md`, `work-with-pages/game-page.md`) — после 1.2 сверить, что код совпал с notes
- [x] 2.2 Если после 1.2 wording skills снова разошёлся с фактическим chrome — поправить notes — verify: skills ↔ design D1/D3 ↔ GamePage
