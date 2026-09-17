## 1. Client — shared header (leave + status)

- [x] 1.1 Прочитать `design.md` (D1), delta `specs/game/leave/spec.md`, `specs/game/presence/spec.md` (SC-PRESENCE-23/24), skills `work-with-pages` и `work-with-rooms` / leave UX; сверить текущий `App.vue` и chrome в `GamePage.vue`
- [x] 1.2 В `App.vue` на route Game: icon-only leave слева, centered match status, theme справа; на Login/Lobby leave и status не показывать — verify: SC-LEAVE-01/08, SC-PRESENCE-23
- [x] 1.3 Перенести leave confirm + consented leave orchestration к header control (store `leaveGame` / router), убрать локальный game-header (leave/status/roomId) с `GamePage` — verify: SC-LEAVE-02…07 без регрессии, SC-PRESENCE-24 (нет room id в chrome)

## 2. Client — bottom HUD panel

- [x] 2.1 Прочитать delta `specs/game/presence/spec.md`, `specs/game/pieces/spec.md`, `specs/game/say/spec.md`, skill `work-with-game-board`; сверить `presenceLayout` / strip / say CSS в `GamePage.vue`
- [x] 2.2 Собрать sticky bottom `.game-hud`: seated — own+budgets/end-turn → strip → opponents right; spectator — occupied centered; board в scroll region над панелью — verify: SC-PRESENCE-02/03/22, SC-PIECE-09/10
- [x] 2.3 Убрать caption «Мои туристы»; say bubbles только выше аватара для всех маркеров; убрать top-row presence — verify: SC-PIECE-09 (no caption), SC-SAY-11/12

## 3. Client — skills + quality (HUD phase)

- [x] 3.1 Обновить `.agents/skills/client/work-with-game-board` и `work-with-pages` (+ при необходимости say/styles notes) под header chrome и bottom HUD — verify: skills описывают новый layout без top-row / page-local leave header
- [x] 3.2 В sibling client: `npm run lint` и `npm run typecheck` — verify: оба проходят

## 4. Client — compact tourist chip + picker menu

- [x] 4.1 Прочитать `design.md` (D6–D8), delta `specs/game/pieces`, `game/finish`, `game/board`; сверить текущий strip / return / `GRILLE_ANIM_MS` в `GamePage.vue`
- [x] 4.2 Заменить ряд×4 в HUD на compact 2×2 chip (~avatar); клик chip только открывает `q-menu` вверх к доске без select — verify: SC-PIECE-09, SC-PIECE-29
- [x] 4.3 В меню: ряд из 4 полноразмерных слотов без title; select unfinished non-trapped → закрыть; Esc/outside без select; на чужом ходе только просмотр; board select без меню — verify: SC-PIECE-30
- [x] 4.4 Finish flag на chip и в меню; return control только в меню (не на chip) — verify: SC-FINISH-09/10/13
- [x] 4.5 Grille overlay на chip/menu при `trapped`; `GRILLE_ANIM_MS = 1000` для board и chrome drop/rise — verify: SC-PIECE-31, SC-BOARD-18/19

## 5. Client — skills + quality (chip phase)

- [x] 5.1 Обновить `.agents/skills/client/work-with-game-board` (+ pages/styles/finish notes) под compact chip, `q-menu`, grille на chrome, anim 1000 ms, return только в меню — verify: skills без «четыре 72px в ряд в HUD» / return на strip-chrome
- [x] 5.2 В sibling client: `npm run lint` и `npm run typecheck` — verify: оба проходят
