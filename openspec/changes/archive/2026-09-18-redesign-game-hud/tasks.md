## 1. Client — shared header (leave + status)

- [x] 1.1 Прочитать `design.md` (D1), delta `specs/game/leave/spec.md`, `specs/game/presence/spec.md` (SC-PRESENCE-23/24), skills `work-with-pages` и `work-with-rooms` / leave UX; сверить текущий `App.vue` и chrome в `GamePage.vue`
- [x] 1.2 В `App.vue` на route Game: icon-only leave слева, centered match status, theme справа; на Login/Lobby leave и status не показывать — verify: SC-LEAVE-01/08, SC-PRESENCE-23
- [x] 1.3 Перенести leave confirm + consented leave orchestration к header control (store `leaveGame` / router), убрать локальный game-header (leave/status/roomId) с `GamePage` — verify: SC-LEAVE-02…07 без регрессии, SC-PRESENCE-24 (нет room id в chrome)

## 2. Client — bottom HUD panel (фаза 1)

- [x] 2.1 Прочитать delta `specs/game/presence/spec.md`, `specs/game/pieces/spec.md`, `specs/game/say/spec.md`, skill `work-with-game-board`; сверить `presenceLayout` / strip / say CSS в `GamePage.vue`
- [x] 2.2 Собрать sticky bottom `.game-hud`: seated — own+budgets/end-turn → strip → opponents right; spectator — occupied centered; board в scroll region над панелью — verify: SC-PRESENCE-02/03/22, SC-PIECE-09/10
- [x] 2.3 Убрать caption «Мои туристы»; say bubbles только выше аватара для всех маркеров; убрать top-row presence — verify: SC-PIECE-09 (no caption), SC-SAY-11/12

## 3. Client — skills + quality (HUD phase)

- [x] 3.1 Обновить `.agents/skills/client/work-with-game-board` и `work-with-pages` (+ при необходимости say/styles notes) под header chrome и bottom HUD — verify: skills описывают новый layout без top-row / page-local leave header
- [x] 3.2 В sibling client: `npm run lint` и `npm run typecheck` — verify: оба проходят

## 4. Client — compact tourist chip + picker menu (фаза 1; план фазы 2 отменяет chip)

- [x] 4.1 Прочитать `design.md` (D6–D8), delta `specs/game/pieces`, `game/finish`, `game/board`; сверить текущий strip / return / `GRILLE_ANIM_MS` в `GamePage.vue`
- [x] 4.2 Заменить ряд×4 в HUD на compact 2×2 chip (~avatar); клик chip только открывает `q-menu` вверх к доске без select — verify: SC-PIECE-09, SC-PIECE-29
- [x] 4.3 В меню: ряд из 4 полноразмерных слотов без title; select unfinished non-trapped → закрыть; Esc/outside без select; на чужом ходе только просмотр; board select без меню — verify: SC-PIECE-30
- [x] 4.4 Finish flag на chip и в меню; return control только в меню (не на chip) — verify: SC-FINISH-09/10/13
- [x] 4.5 Grille overlay на chip/menu при `trapped`; `GRILLE_ANIM_MS = 1000` для board и chrome drop/rise — verify: SC-PIECE-31, SC-BOARD-18/19

## 5. Client — skills + quality (chip phase)

- [x] 5.1 Обновить `.agents/skills/client/work-with-game-board` (+ pages/styles/finish notes) под compact chip, `q-menu`, grille на chrome, anim 1000 ms, return только в меню — verify: skills без «четыре 72px в ряд в HUD» / return на strip-chrome
- [x] 5.2 В sibling client: `npm run lint` и `npm run typecheck` — verify: оба проходят

## 6. Client — presence top / bottom own (фаза 2)

- [x] 6.1 Прочитать обновлённые `design.md` (D2–D3), delta `presence` / `say`; сверить текущий `presenceLayout` / HUD / bubbles в `GamePage.vue`
- [x] 6.2 Seated: opponents row над доской; sticky bottom = own + strip; spectator: все markers над доской, без bottom presence — verify: SC-PRESENCE-02/03/22
- [x] 6.3 Steps/peeks в ряд над own avatar; end-turn над нижней панелью справа — verify: SC-PRESENCE-15/25
- [x] 6.4 Say: top markers → bubbles вниз к доске; own bottom → вверх — verify: SC-SAY-11/12

## 7. Client — strip без menu + narrow 2×2

- [x] 7.1 Убрать compact chip и `q-menu`; вернуть четыре слота в bottom HUD (select unfinished с слота / доски) — verify: SC-PIECE-09/10, нет SC-PIECE-29/30 UX
- [x] 7.2 Широкая ширина: ряд N,E,W,S; узкая (~320, лучше ~300): 2×2 со слотами меньше avatar; own+strip без primary horizontal scroll — verify: SC-PIECE-32
- [x] 7.3 Grille overlay на strip-слотах при `trapped`; `GRILLE_ANIM_MS = 1000` — verify: SC-PIECE-31, SC-BOARD-18/19

## 8. Client — return modal, dim, targets, anim

- [x] 8.1 Finished: dim только если return недоступен; иначе полный вид + клик → модалка «Вернуть на поле?» — verify: SC-FINISH-09/13
- [x] 8.2 После Да: ring highlights тем же стилем, что move targets; убрать orange-only return chrome; step только на server accept; смена туриста сбрасывает return-mode — verify: SC-FINISH-13, SC-MOVE-12
- [x] 8.3 Убрать отдельную undo-кнопку на слоте; flag на слотах; после full finish четыре слота остаются — verify: SC-FINISH-10/13
- [x] 8.4 Анимация return: с ближайшего center (Chebyshev, tie row/col) на ring для всех клиентов — verify: SC-FINISH-15

## 9. Client — finish-block nearest legal center

- [x] 9.1 Клик по любому месту visual finish 2×2 → submit на nearest legal center cell относительно selected piece; без quadrant mapping — verify: SC-MOVE-65

## 10. Client — skills + quality (фаза 2)

- [x] 10.1 Обновить `.agents/skills/client/work-with-game-board` (+ pages/styles/say/finish notes) под top opponents, strip row/2×2, modal return, nearest center, return anim — verify: skills без chip/`q-menu` / «все markers внизу»
- [x] 10.2 В sibling client: `npm run lint` и `npm run typecheck` — verify: оба проходят

## 11. Client — tight top presence + budgets beside avatar (фаза 3)

- [x] 11.1 Убрать reserved gap у `.presence-row--top` (markers вплотную к доске; say absolute) — verify: нет большого пустого капа над доской
- [x] 11.2 Budgets (steps/peeks) справа от own avatar; end-turn dock без изменений — verify: SC-PRESENCE-15/25
- [x] 11.3 Обновить skills / design / presence delta; `npm run lint` + `npm run typecheck` — verify: оба проходят
