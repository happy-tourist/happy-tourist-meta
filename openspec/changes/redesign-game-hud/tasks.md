## 1. Client — shared header (leave + status)

- [x] 1.1 Прочитать `design.md` (D1), delta `specs/game/leave/spec.md`, `specs/game/presence/spec.md` (SC-PRESENCE-23/24), skills `work-with-pages` и `work-with-rooms` / leave UX; сверить текущий `App.vue` и chrome в `GamePage.vue`
- [x] 1.2 В `App.vue` на route Game: icon-only leave слева, centered match status, theme справа; на Login/Lobby leave и status не показывать — verify: SC-LEAVE-01/08, SC-PRESENCE-23
- [x] 1.3 Перенести leave confirm + consented leave orchestration к header control (store `leaveGame` / router), убрать локальный game-header (leave/status/roomId) с `GamePage` — verify: SC-LEAVE-02…07 без регрессии, SC-PRESENCE-24 (нет room id в chrome)

## 2. Client — bottom HUD panel

- [x] 2.1 Прочитать delta `specs/game/presence/spec.md`, `specs/game/pieces/spec.md`, `specs/game/say/spec.md`, skill `work-with-game-board`; сверить `presenceLayout` / strip / say CSS в `GamePage.vue`
- [x] 2.2 Собрать sticky bottom `.game-hud`: seated — own+budgets/end-turn → strip → opponents right; spectator — occupied centered; board в scroll region над панелью — verify: SC-PRESENCE-02/03/22, SC-PIECE-09/10
- [x] 2.3 Убрать caption «Мои туристы»; say bubbles только выше аватара для всех маркеров; убрать top-row presence — verify: SC-PIECE-09 (no caption), SC-SAY-11/12

## 3. Client — skills + quality

- [x] 3.1 Обновить `.agents/skills/client/work-with-game-board` и `work-with-pages` (+ при необходимости say/styles notes) под header chrome и bottom HUD — verify: skills описывают новый layout без top-row / page-local leave header
- [x] 3.2 В sibling client: `npm run lint` и `npm run typecheck` — verify: оба проходят
