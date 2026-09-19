## Context

См. `proposal.md` и delta specs `game/move`, `game/finish`, `game/board`, `game/presence`. Push + return-strip уже в runtime; этот апдейт — client UX polish поверх. Explore polish D*/мелочи закрыты — ниже как решения.

Чеклист — `tasks.md` (блоки 1–4 done; блок 5 — polish).

## Goals / Non-Goals

**Goals:**

- Message `push` + pure validate/apply (уже).
- Client: push icons; return strip без modal (уже).
- Affordance anchor: peek / rescue / push — top-center над piece (как `.return-affordance`).
- End-turn: icon `skip_next` right-center у own avatar; сразу `endTurn`; без dock/label/dialog; solo без контрола.
- Push→center: тот же travel + disappear, что move→center.

**Non-Goals:**

- Push trapped; red rings для dest push; schema под push; новые deps; tooltips; массовая смена иконок.

## Decisions

### D1 — Wire: `push` (done)

| Message | Payload | Effect |
|---------|---------|--------|
| `push` | `pusherSide`, `targetSessionId`, `targetSide`, `row`, `col` | −1 step; цель на `(row,col)`; pusher coords без изменений |

### D2–D3 — Server pure + MyRoom (done)

`touristMove.ts` helpers; `onMessage('push')` → `handlePush`; auto-end + `hasLegalPush`.

### D4 — Client store / GamePage push + return (done)

`sendPush`; push affordances; approach/back; return icon; no return modal.

### D5 — UX icons (расширено polish)

| Affordance | Placement | Icon (пока) |
|------------|-----------|-------------|
| Peek | top-center над selected | `visibility` |
| Rescue | top-center над trapped | `lock_open` |
| Push | top-center над target | `swipe` |
| Return strip | top-center над finished slot | `undo` |
| End-turn | **right-center** у own avatar (зеркало say top-center) | `skip_next` |

Общий board CSS-якорь (вместо угла `+ cell - 14px` / `- 6px`): центр клетки по X, чуть выше по Y — как strip `left: 50%; transform: translateX(-50%); top: -10px` в координатах board overlay.

### D6 — Selection / multi-push (done)

Keep `selectedSide` после push.

### D7 — Skills

При apply polish: client `work-with-game-board`, `work-with-pages`, `work-with-styles`, localization (aria only), presence notes; server skills уже покрывают push.

### D8 — Explore push (закрыты) + polish

| ID | Решение |
|----|---------|
| cost / trapped / grille / return UX / icons / keep focus / auto-end | как раньше |
| polish D1 | peek+rescue+push top-center |
| polish D2 | end-turn без dialog |
| polish D3 | `skip_next` |
| polish D4 | push→finish как ход |
| мелочь | end-turn right-center; без видимого текста; solo без кнопки |

### D9 — End-turn chrome

| Было | Станет |
|------|--------|
| `.end-turn-dock` + `q-btn` label «Завершить ход» | убрать dock; button на own `presence-marker` справа по центру |
| Confirm | нет — `@click` → существующий `onEndTurnClick` / `sendEndTurn` |

Budgets stack справа от аватара **не** включает end-turn (SC-PRESENCE-25). Say остаётся top (или top-center по продукту say — не менять в этом change, если уже top-right; end-turn — right-center независимо).

### D10 — Push→finish presentation

Проблема: при sync `row/col=center` + `finished` цель может отрисоваться сразу на центре / пропасть без кадра на старой клетке.

Подход: при появлении нового finished key (вкл. от push) — если есть last-known board cell ≠ center, один кадр paint `from` (как `returnAnimFromByKey`), затем slide на center + `piece--disappearing` (тот же MOVE_ANIM + FINISH_FADE, что move finish). Не менять server.

## Risks / Trade-offs

- [Чужой finish от push] → ожидаемо; SC-MOVE-69 / SC-FINISH-17.
- [End-turn без текста] → aria-label ok; видимые tooltips out of scope.
- [Say сейчас top-right в CSS] → не блокер; end-turn right-center по продукту.

## Migration Plan

Не требуется (client UX; push message уже на сервере).

## Open Questions

(нет)
