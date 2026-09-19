## Context

См. `proposal.md` и delta specs `game/move`, `game/finish`, `game/board`, `game/presence`. Push + return-strip + polish (affordance/end-turn/push→finish) уже в runtime; этот апдейт — client fix: свой ход на центр должен давать тот же travel+fade.

Чеклист — `tasks.md` (блоки 1–6 done).

## Goals / Non-Goals

**Goals:**

- Message `push` + pure validate/apply (уже).
- Client: push icons; return strip без modal; affordance top-center; end-turn `skip_next` (уже).
- Push→center travel+disappear (уже).
- **Move→center:** тот же travel с pre-move клетки + disappear; не vanish с соседней клетки.

**Non-Goals:**

- Push trapped; red rings для dest push; schema под push; новые deps; tooltips; массовая смена иконок; server finish wire.

## Decisions

### D1 — Wire: `push` (done)

| Message | Payload | Effect |
|---------|---------|--------|
| `push` | `pusherSide`, `targetSessionId`, `targetSide`, `row`, `col` | −1 step; цель на `(row,col)`; pusher coords без изменений |

### D2–D3 — Server pure + MyRoom (done)

`touristMove.ts` helpers; `onMessage('push')` → `handlePush`; auto-end + `hasLegalPush`.

### D4 — Client store / GamePage push + return (done)

`sendPush`; push affordances; approach/back; return icon; no return modal.

### D5 — UX icons (done)

| Affordance | Placement | Icon (пока) |
|------------|-----------|-------------|
| Peek | top-center над selected | `visibility` |
| Rescue | top-center над trapped | `lock_open` |
| Push | top-center над target | `swipe` |
| Return strip | top-center над finished slot | `undo` |
| End-turn | **right-center** у own avatar | `skip_next` |

### D6 — Selection / multi-push (done)

Keep `selectedSide` после push.

### D7 — Skills

При apply fix: client `work-with-game-board` (finish travel move+push); pages/styles при drift. Server skills без изменений.

### D8 — Explore push + polish (закрыты)

| ID | Решение |
|----|---------|
| cost / trapped / grille / return UX / icons / keep focus / auto-end | как раньше |
| polish D1–D4 / мелочи end-turn | done |

### D9 — End-turn chrome (done)

Dock убран; `skip_next` right-center; без dialog/label; solo без контрола.

### D10 — Push→finish presentation (done)

`lastKnownBoardCellByKey` + `finishAnimFromByKey` + `disappearingKeys` — один кадр `from`, slide на center, fade.

### D11 — Move→center finish travel (fix)

**Проблема:** push→center анимируется, свой `move` на центр визуально пропадает (кадр `from` не удерживается / sync уже с center).

**Подход (A+B):**

1. **Capture at submit:** в `submitMove` при `isCenterCell(row,col)` сразу записать текущую клетку выбранной фигуры в `lastKnownBoardCellByKey` до ответа сервера — локальный own-move не теряет `from`. **Не** seed `finishAnimFromByKey` на submit (silent reject иначе pin’ит фигуру через `pieceStyle`).
2. **Harden watch:** при новом finished key — paint `from` если last-known ≠ center; не clear `finishAnimFromByKey` до реального paint (nextTick + forced layout + double rAF); unfinished-watch сбрасывает stale `finishAnimFrom` для ещё незавершённых ключей; fade через существующий `piece--disappearing` после MOVE_ANIM. Remote finish (чужой ход / push) по-прежнему через watch + lastKnown.
3. Server не менять.

## Risks / Trade-offs

- [Чужой finish от push/move] → тот же watch path; SC-FINISH-01/17/18.
- [Двойной seed lastKnown] → идемпотентно; ключ pieceKey.
- [End-turn без текста] → aria ok (уже).

## Migration Plan

Не требуется (client UX).

## Open Questions

(нет)
