## Context

См. `proposal.md` и delta specs `game/move`, `game/finish`. Сейчас: `move` / `rescue` / `returnFromFinish` / peek / endTurn; `touristMove.ts` validate one-step + rescue/return helpers; GamePage — eye (синий), rescue (зелёный), return через confirm modal на strip slot. Explore D*/Q* закрыты — ниже как решения, не блокеры.

Чеклист — `tasks.md`.

## Goals / Non-Goals

**Goals:**

- Message `push` + pure validate/apply в `touristMove.ts`; −1 step; side-effects посадки цели как у `move` (finish / grille trap).
- Client: push icons над целями выбранного pusher; approach/back + travel; keep selection.
- Return: зелёная иконка над strip; без модалки; слот body не стартует return.
- Auto-end учитывает legal push.

**Non-Goals:**

- Push trapped; red rings для dest push; schema-поля под push; новые deps.

## Decisions

### D1 — Wire: `push`

| Message | Payload | Effect |
|---------|---------|--------|
| `push` | `pusherSide`, `targetSessionId`, `targetSide`, `row`, `col` | −1 step текущего seat; цель на `(row,col)`; pusher coords без изменений |

- `pusherSide` — свой unfinished free piece текущего seat.
- `targetSessionId` + `targetSide` — любая unfinished free piece в room (свой или чужой).
- `(row,col)` — far-side клетка: `target + (target − pusher)` в row/col; Chebyshev(pusher,target)=1 и Chebyshev(target,dest)=1.
- Reject: не ход / steps 0 / trapped target или pusher / bad geometry / hole / occupied / not playable — без смены state.
- После apply: те же side-effects, что land после `move` для **цели** (center → finish; unspent grille → trap + all-jail check).

### D2 — Pure rules в `touristMove.ts`

- Helpers: `farSideCell(pusher, target)`, `listLegalPushes(pusherPieces, allPieces, removed)`, `validateTouristPush(...)`, `hasLegalPush(...)`.
- Landing check: переиспользовать ту же landable-логику, что `validateTouristMove` для dest (occupancy excludes target’s current cell as it leaves).
- `hasAvailableActions` / auto-end: `hasLegalPush` рядом с rescue/return.

### D3 — Server врезки

| Место | Что |
|-------|-----|
| `src/rooms/MyRoom.ts` | `onMessage('push')` → `handlePush`; auto-end + `hasAvailableActions` |
| `src/game/touristMove.ts` | validate/apply/list/hasLegalPush |
| `test/MyRoom.test.ts` (+ unit pure) | SC-MOVE-66…73 |

Порядок apply в handler (зеркало move):

1. validate push
2. −1 step
3. relocate target; clear occupancy from old cell
4. if center → finish target
5. else if grille → trap target (+ all-jail)
6. `maybeAutoEndTurn`

### D4 — Client врезки

| Место | Что |
|-------|-----|
| `src/stores/game.ts` | `sendPush(pusherSide, targetSessionId, targetSide, row, col)` |
| `src/pages/GamePage.vue` | compute push affordances от `selectedSide`; icons над targets; reuse `rescueAnimOverride` pattern для approach/back; target travel через существующий move anim; keep `selectedSide` after push; return icon на strip; удалить `returnConfirm*` dialog/flow; `onStripSlotClick` finished → noop |
| `src/i18n/*` | `game.pushAffordance`; return aria уже есть — поправить тексты если про modal |

### D5 — UX icons

- Push: зелёный круг как rescue; Material `swipe` (или `front_hand`); над **целью**.
- Eye: без изменений (над selected).
- Return strip: зелёный круг над центром finished tourist; Material `undo` / `replay`; клик → `returningSide` + red rings (как сейчас после confirm).

### D6 — Selection / multi-push

- Иконки только при selected own free + own turn + steps≥1.
- Несколько targets → несколько icons; каждый клик = один push (−1 step); selection остаётся → можно второго.

### D7 — Skills при apply

Server: `work-with-messages`, `work-with-game`, `server-work-with-test`.  
Client: `work-with-game-board`, `work-with-pages`, `work-with-stores`, `work-with-localization`, `colyseus-client`.  
Cross-package: **server first**, затем client.

### D8 — Explore (закрыты)

| ID | Решение |
|----|---------|
| cost | 1 step |
| trapped | нельзя толкать |
| grille | как move land |
| return UX | icon over strip; no modal; slot body не стартует |
| icons | над targets выбранного |
| eye vs push | eye на selected, push на target |
| keep focus | да |
| auto-end | legal push = available action |

## Risks / Trade-offs

- [Чужой finish от push] → ожидаемо по продукту; тесты SC-MOVE-69.
- [Overlap icon с rescue на соседней клетке] → rescue только над trapped; push не на trapped — конфликт нет.
- [Клиент шлёт чужой targetSessionId] → server валидирует геометрию от pusher текущего seat.

## Migration Plan

Не требуется (новое message; старые клиенты просто не шлют push; return UX только client).

## Open Questions

(нет)
