## Context

См. `proposal.md` — Why / Scope. **Уже в коде (фаза 1 + 2):** leave + match status в `App.vue`; sticky `.game-hud`; grille `GRILLE_ANIM_MS = 1000`; room id убран; opponents/spectator top; seated bottom = own + strip (row / `@container` ≤~420 → 2×2; **нет** chip/`q-menu`); budgets над аватаром; end-turn dock; return confirm modal + красные targets; nearest-center finish click + return anim. **Отменено фазой 2:** chip 2×2 + `q-menu`; все markers внизу; budgets рядом с аватаром; return undo на слоте; оранжевые return-targets; `resolveCenterClick` по квадрантам.

Пакет: **client** (`../happy-tourist.github.io`). Server / `returnFromFinish` без изменений.

## Goals / Non-Goals

**Goals:**

- Presence: opponents (и spectator all) над доской; bottom = own + budgets-над-аватаром + strip; end-turn над панелью справа.
- Strip без menu: ряд на широком / 2×2 меньше avatar; CQ break при own+row ≈420 (покрывает ≤320/300 без primary scroll).
- Return: modal → красные targets → step только на accept; dim finished только если return недоступен; anim с ближайшего center.
- Finish click: любой клик по 2×2 → nearest legal center (Chebyshev).

**Non-Goals:**

- Server / schema / messages.
- Вынос GamePage на компоненты ради рефакторинга.
- Idle-loop grille.

## Decisions

### D1 — Chrome leave + status в `App.vue` (фаза 1)

**Статус:** реализовано — не трогать.

### D2 — Presence: top opponents / bottom own

**Выбор:** row над board для чужих (seated) или всех occupied (spectator). Sticky `.game-hud` только для seated own cluster + strip. Say: top markers → bubbles вниз; own bottom → вверх.

**Альтернатива:** все снизу (фаза 1) — отвергнуто explore.

### D3 — Budgets над аватаром; end-turn над панелью справа

**Выбор:** `.presence-budgets` горизонтальный ряд над own avatar. End-turn — отдельный control в board region / над HUD, `right` alignment. Не в budgets row.

### D4 — Strip без chip/menu; responsive ряд ↔ 2×2

**Выбор:** удалить chip/`q-menu`. Слоты в HUD: `@container game-hud (max-width: 420px)` (own 96 + gap 12 + 4×72 + gaps ≈ 420) → CSS grid 2×2 с slot меньше avatar; иначе flex row N,E,W,S. Покрывает product ≤320/300 без primary horizontal scroll.

**Альтернатива:** всегда chip — отвергнуто. Breakpoint «ровно 320» — отвергнуто: при avatar 72 / outer 96 ряд не влезает уже ~420.

### D5 — Return modal + conditional dim + same-color targets

**Выбор:** клик finished при `canReturn(side)` → `q-dialog` «Вернуть на поле?» → `returningSide`. Targets: тот же class/outline, что `.tile--target` (убрать отдельный orange). Dim (`opacity`) только когда `!canReturn`. Переключение `selectOwnSide` сбрасывает return-mode. Step — только server accept.

### D6 — Return anim from nearest center

**Выбор:** на всех клиентах при появлении piece после return: стартовая позиция = nearest of `CENTER_CELLS` к целевой клетке (Chebyshev; tie row, col); slide `MOVE_ANIM_MS` на ring (зеркало finish disappear).

### D7 — Finish-block click → nearest legal center

**Выбор:** заменить `resolveCenterClick` quadrant: при клике по `tile.kind === 'center'` выбрать argmin Chebyshev среди `CENTER_CELLS ∩ legalTargets` относительно `selectedCell`; иначе no-op.

### D8 — Grille 1000 ms

**Статус:** оставить `GRILLE_ANIM_MS = 1000` на board + strip slots.

### D9 — Skills

**Статус:** сделано — `work-with-game-board` / pages / styles / localization / AGENTS индексы отражают фазу 2 (420 CQ, return modal, top presence, nearest center).

## Risks / Trade-offs

- [Узкая ширина + rings 96px] → Mitigation: уменьшить PRESENCE outer/avatar вместе со strip на narrow.
- [End-turn над панелью перекрывает доску] → Mitigation: только когда available; правый край; pointer-events локально.
- [Return anim без server «from» cell] → Mitigation: чисто client presentation от nearest center.
- [Purpose main specs] → при `/opsx-sync` обновить presence/pieces/finish/move/say.

## Migration Plan

Только client deploy. Rollback = revert UI commit. Server не затрагивается.

## Open Questions

Нет (explore D1–D4, D3b, F1/F2, A1 закрыты).
