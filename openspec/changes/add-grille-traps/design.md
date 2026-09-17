## Context

См. `proposal.md`. Сейчас: create option только `maxSeats`; task rewards — private Map на `enterPlaying`; `removedTaskKeys` sync; `Piece.finished`; strip `flag`; move/peek/endTurn; анимации хода/финиша в GamePage. Ловушек нет. Explore D*/Q* закрыты — prerequisite ниже зафиксированы как решения, не открытые блокеры.

Чеклист реализации — `tasks.md`.

## Goals / Non-Goals

**Goals:**

- Create: пресеты плотности 25/45/65% (мало/средне/много), default средне.
- Seed скрытых решёток только на `*` при `playing`; reveal+trap on land; public drop/rise anim.
- Rescue своих (Chebyshev-1 вкл. диагональ, −1 step); return finished на кольцо центра (−1 step); all-jail → сразу старты стороны + warning modal only self.
- Solo = multi по правилам решёток.
- Ассет `grille.png` в client.

**Non-Goals:**

- Другие типы ловушек; редактор карт; спасение чужих; магазин; смена layout; новые HTTP/auth.

## Decisions

### D1 — Плотность как процент от task на seed

- **Выбор:** create option `grilleDensity: 'few' | 'medium' | 'many'` → 0.25 / 0.45 / 0.65; count = `clamp(0, taskCount, Math.round(taskCount * p))`. На стандартном layout: 12 / 22 / 31.
- **Почему %:** будущие карты с другим числом `*`.
- Default create: `medium`.
- Persist на room: поле schema или private + metadata по желанию; достаточно private на instance + seed once.

### D2 — Скрытие до reveal

- **Выбор:** private `Map`/`Set` hidden grille keys (как `taskRewards`); sync только **revealed** holding/spent markers нужные клиенту.
- Минимальный sync:
  - `Piece.trapped: boolean`
  - room array/map **revealed grille cells currently down** (holding) — для overlay у всех; после clear — remove from sync (spent, no re-arm this match).
- Hidden locations never synced.

### D3 — Messages

| Message | Payload (sense) | Effect |
|---------|-----------------|--------|
| existing `move` | side, row, col | + side-effect trap if grille hidden/unspent on target |
| `rescue` | trapped `side` | −1 step; clear trapped + grille if own + adjacent free piece + steps |
| `returnFromFinish` | `side`, `row`, `col` | −1 step; unfinish onto legal ring cell |

Reject codes: bad turn/budgets/ownership/geometry/occupancy/hole — без смены state.

### D4 — Land → trap order

1. `validateTouristMove` (existing)
2. apply coords; −1 step
3. if center → finish (existing; no grille on center)
4. else if cell has unspent grille → reveal + `trapped=true`
5. if seat now has 4 trapped → all-jail reset (D6) immediately
6. auto-end evaluation must include rescue/return (D7)

### D5 — Rescue / return UX

- Rescue: affordance над **trapped** piece, когда own free piece Chebyshev-1 и steps≥1 (аналог eye).
- Client anim: rescuer briefly slides toward cell and back (`MOVE_ANIM_MS`); server coords rescuer unchanged.
- Return: иконка рядом с `flag` на finished strip slot; клик → highlight ring; клик по клетке → `returnFromFinish`.
- Ring cells: all playable cells with Chebyshev distance 1 from the center **block** (any of the four center cells), excluding the four center cells themselves; exclude `removedTaskKeys` and occupied cells.

### D6 — All-jail

- Trigger: after a trap, seat has exactly 4 trapped pieces (implies 0 finished).
- Immediate: clear 4 traps; clear 4 holding grilles; place each piece on random free start of its side (`START_CELLS` pools).
- Sync state first; client of that seat opens **warning** modal (i18n); no confirm gate; other clients no modal.
- Keep `currentTurnSessionId` / remaining steps / `turnUntil`.

### D7 — Auto-end

Extend “has available action” with: legal rescue OR legal return (finishPlace===0). Same spirit as peek-on-`*` gate.

### D8 — Ассет

- Path: `happy-tourist.github.io/src/assets/grilles/grille.png`
- Import in GamePage like tourists; overlay on cell when revealed/holding.

### D9 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/MyRoom.ts` | parse density onCreate; seed on enterPlaying; move trap side-effect; onMessage rescue/returnFromFinish; all-jail; auto-end |
| `src/rooms/schema/MyRoomState.ts` | `Piece.trapped`; revealed grille keys collection |
| `src/game/touristMove.ts` (or sibling pure module) | ring-cell helpers; adjacency; density count; optional validate rescue/return |
| `test/MyRoom.test.ts` (+ unit if pure) | SC-MOVE-51…64, SC-BOARD-16/20, SC-PIECE-24…26, SC-FINISH-12/14, SC-LOBBY-14 |

### D10 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `src/pages/LobbyPage.vue` | density option-group + i18n |
| `src/stores/game.ts` | CreateGameOptions density; mirror trapped/revealed grilles; sendRescue / sendReturnFromFinish |
| `src/pages/GamePage.vue` | grille overlay+anim; rescue icon; return+ring hints; all-jail modal; lock select/peek when trapped |
| `src/i18n/*` | lobby density; all-jail warning; a11y return/rescue |
| `src/assets/grilles/grille.png` | art (user-provided) |

### D11 — Skills при apply

Server: `work-with-schema`, `work-with-messages`, `work-with-game`, `work-with-rooms`, `server-work-with-test`.  
Client: `work-with-lobby`, `work-with-game-board`, `work-with-pages`, `work-with-stores`, `work-with-localization`, `colyseus-client`.  
Cross-package: **server contract first**, then client.

### D12 — Explore prerequisites (закрыты)

| ID | Решение |
|----|---------|
| D1 | только task |
| D2 | own rescue, 1 step |
| D3 | all 4 own → starts per side |
| D4 | holding grilles cleared on rescue/all-jail |
| D5 | return = unfinished in play |
| D6 | ring w/ corners |
| D7 | grille clear ≠ tile remove; peek after OK |
| density | 25/45/65, default medium |
| Q8 | same turn after reset |
| Q9 | N/A (land reveals before peek) |
| Q12 | auto reset + warning modal self-only |

## Risks / Trade-offs

- **Высокая плотность 65%** может казаться жёсткой — продукт согласился начать с 25/45/65 и поднять позже при необходимости.
- **Return всегда при steps** ослабляет «финиш как обязательство» — осознанно; full `finishPlace` всё равно без хода/steps на практике.
- **Коллизии стартов при all-jail:** при 4 игроках теоретически тесны стартовые ряды; брать только free cells стороны (продукт: «все не заняты» в типичном кейсе). Если free pool пуст — design fallback: выбрать любую free playable start of that side after scanning; mocha на happy-path достаточно.
- Sync только revealed grilles → меньше churn, чем sync всех hidden.

## Open Questions

- (нет продуктовых; при apply уточнять только имена schema fields / reject reason strings в коде.)
