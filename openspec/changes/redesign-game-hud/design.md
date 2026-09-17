## Context

См. `proposal.md` — Why / Scope. **Реализовано:** leave + match status в `App.vue` на Game; sticky `.game-hud`; compact 2×2 chip + upward `q-menu` picker; grille/`trapped` и finish на chrome; return только в меню; grille anim **1000 ms** на поле и chrome; say bubbles вверх; room id убран.

Пакет: **client** (`../happy-tourist.github.io`). Контракт room `tourist` без изменений. Данные: `mySeat.pieces[].trapped` / `finished`, `holdingGrilleKeys`, asset `grille.png`.
## Goals / Non-Goals

**Goals:**

- Общая шапка + sticky bottom HUD (уже сделано) + compact tourist chrome в HUD.
- Chip 2×2 (~avatar): статусы; tap → menu вверх к доске; ряд×4 select/return; board select без menu.
- Grille overlay на chip/menu при `trapped`; anim drop/rise **1000 ms** везде.
- Обновить client skills под chip/menu/anim после кода.

**Non-Goals:**

- Server / schema / messages.
- Вынос GamePage на компоненты ради рефакторинга (допустимо минимальный shared slot template).
- Idle-loop grille; смена правил trap/rescue.

## Decisions

### D1 — Chrome leave + status в `App.vue` (route-aware)

**Выбор:** расширить `src/App.vue` `q-toolbar`: при `route.name === 'game'` показывать icon-only leave слева и centered status; theme справа. Confirm + `leaveGame` в App; store `consentedLeaving` с clear после leave / уходе с Game.

**Статус:** реализовано.

### D2 — Нижняя панель: layout shell на GamePage

**Выбор:** board scroll region + sticky `.game-hud` (own | budgets | strip/chip | opponents). Say только `--bottom`.

**Статус:** реализовано (strip заменён compact chip — D6).
### D3 — Sticky к viewport

**Выбор:** `position: sticky; bottom: 0` внутри page column.

**Статус:** реализовано.

### D4 — Удаление room id chrome

**Статус:** реализовано.

### D5 — Skills (после кода)

Обновить skills под header + HUD; затем ещё раз под chip/menu/grille chrome (tasks).

### D6 — Compact chip + `q-menu` picker

**Выбор:** в `.game-hud` вместо четырёх `.my-tourist-slot` 72px — один chip ~`PRESENCE` image box (72px) с CSS grid 2×2: стороны `N E` / `W S`. Клик по chip **только** `v-model`/`q-menu` (не `selectedSide`). Меню: `anchor` к доске (вверх), без title; контент — горизонтальный ряд четырёх полноразмерных слотов (текущие размеры/иконки). Select доступного unfinished non-trapped → `onStripClick` / эквивалент → закрыть menu. Esc / click-outside → закрыть. Outside interactive turn — menu открывается, слоты без select (просмотр). Board piece click — select без menu.

**Альтернативы:** (B) `q-dialog` — отвергнуто (explore: везде menu); (C) long-press chip — отвергнуто.

**Rationale:** экономия HUD; полноразмерный выбор как раньше; Quasar menu portal меньше клипает sticky footer.

### D7 — Статусы и return

**Выбор:** на chip и в menu — finish flag на finished; grille overlay когда `trapped`. Return (`undo`) **только** в menu рядом с flag при прежних условиях (`finishPlace===0`, steps, legal ring). Chip не несёт return.

### D8 — Grille animation 1000 ms

**Выбор:** константа client `GRILLE_ANIM_MS = 1000` для board overlays и для strip chrome drop/rise (one-shot при появлении/снятии `trapped` / holding clear как на поле). Заменяет прежние ~1500 ms.

**Альтернатива:** разные длительности board vs strip — отвергнуто (explore: везде 1 s).

## Risks / Trade-offs

- [Menu у нижнего края / safe-area] → Mitigation: anchor вверх к доске; Quasar portal.
- [Мелкий 2×2 плохо читает grille] → Mitigation: те же asset + короткая anim; в menu полноразмер.
- [App знает game status] → Mitigation: только зеркало store (уже так).
- [Purpose main specs] → при `/opsx-sync` обновить Purpose / SC timing у pieces/finish/board.

## Migration Plan

Только client deploy (GitHub Pages). Rollback = revert UI commit. Server не затрагивается.

## Open Questions

Нет (D1–D8 и explore Q1–Q6 закрыты).
