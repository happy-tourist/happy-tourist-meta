## 1. Server — room `tourist`

- [x] 1.1 Прочитать `design.md` (D4, D7), delta `specs/lobby/rooms/spec.md` (SC-LOBBY-02…04) и `.agents/skills/server/work-with-rooms/SKILL.md`; сверить `../happy-tourist-server/src/app.config.ts`
- [x] 1.2 В `../happy-tourist-server/src/app.config.ts` переименовать регистрацию playable room `checkers` → `tourist` (+ `.enableRealtimeListing()`); убрать шашечные формулировки в затронутых комментариях `MyRoom` при правке
- [x] 1.3 Обновить `../happy-tourist-server/test/MyRoom.test.ts` и `package.json` loadtest (`--room tourist`) под имя `tourist` (SC-LOBBY-02/03/04)
- [x] 1.4 В `../happy-tourist-server`: `npm test`; при падении — починить до client

## 2. Client — room + статичное поле

- [x] 2.1 Прочитать `design.md` (D1–D5), delta `specs/game/board/spec.md` (SC-BOARD-01…05), `.agents/skills/client/work-with-game-board/SKILL.md` и `.agents/skills/client/work-with-lobby/SKILL.md`; сверить `GamePage.vue` / `stores/game.ts`
- [x] 2.2 В client заменить контракт room `checkers` → `tourist` (const, create/join/joinOrCreate, lobby filter, HTTP `/rooms/...`); убрать идентификаторы/комментарии с «checkers»
- [x] 2.3 На Game заменить 8×8 шашечную доску на статичный tourist layout (CSS Grid, max tile 60px, gap 6, radius 12, mobile full width, цвета start/task/center, сплошной центр 2×2, фон дыр = страница) — SC-BOARD-01…04
- [x] 2.4 Удалить с Game шашечный move UX (selection, targets, getTargets, pieces, sendMove из UI); поле без кликов — SC-BOARD-05
- [x] 2.5 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 3. Meta — ребренд и skills

- [x] 3.1 Переименовать `.agents/skills/server/work-with-checkers` → `work-with-game`; переписать skill под настольную игру / правила later и room `tourist` (design D6)
- [x] 3.2 Обновить client/server skills, meta + sibling `AGENTS.md`, `docs/`, `openspec/config.yaml` context: убрать/заменить шашки и `checkers` на «Счастливый турист» / настольная игра / `tourist`; `work-with-game-board` — под статичное поле
- [x] 3.3 Grep по meta + client + server на `checkers` / «шашк»: неоставшихся продуктовых упоминаний в scope (исключения только исторический archive при необходимости)
- [x] 3.4 Из корня meta: `openspec validate --change add-tourist-game-board` (sync/archive — по отдельной просьбе)
