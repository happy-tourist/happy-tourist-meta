## Context

См. `proposal.md` — Why. Сейчас ход на center — обычный walkable update `row`/`col`; `StartPhase` = `waiting|countdown|playing`; leave seated в `playing` всегда с confirm; strip без статусов; dispose при `seats.size === 0`. Нужен side-effect finish на center, finished-seat и place без новой HTTP-поверхности.

Пакеты: **server** (`../happy-tourist-server`) затем **client** (`../happy-tourist.github.io`). Чеклист — `tasks.md`.

## Goals / Non-Goals

**Goals:**

- Schema: piece finished; seat finish place; room monotonic place counter (или derive from seats).
- `handleMove` / occupancy: center → finish piece (off-board); skip finished в turn; reject move от finished.
- Client: disappear animation; strip finish icon; place modal; presence place badge; leave без confirm для finished.
- Тесты mocha на SC-FINISH / обновлённые SC-MOVE; client lint/typecheck.

**Non-Goals:**

- Фаза room `finished` / авто-kick.
- Новые npm-зависимости / новые Colyseus messages (finish через существующий `move`).
- Lobby обязательный `status: finished`.
- Улёт фишки к strip.

## Decisions

### D1 — Finish как side-effect `move`, без нового message

- **Выбор:** после успешной валидации, если `isCenterCell(target)` → пометить piece finished, убрать из occupancy (не держать row/col как занятую клетку: либо sentinel / clear map entry / `finished=true` + ignore в occupied set). Затем `advanceTurn()` с skip finished.
- **Альтернатива:** отдельный message `finish` — отвергнуто (лишний round-trip; цель уже клетка хода).

### D2 — Schema shape

- **Выбор:**
  - `Piece.finished: boolean` (default false); при finish = true; finished pieces не участвуют в `occupied` и не рендерятся на доске.
  - `Seat.finishPlace: number` (0 = нет места; 1..n после полного финиша).
  - `MyRoomState.nextFinishPlace: number` (стартует с 1; инкремент при выдаче места) — авторитетный счётчик порядка.
- **Альтернатива:** удалять piece из `seat.pieces` map — отвергнуто (strip 1:1 по side ломается; проще finished flag).

### D3 — Turn skip

- **Выбор:** `advanceTurn` / выбор current turn итерирует seats в join order, пропуская `finishPlace > 0`. Если current стал finished в том же apply (4-я фишка) — сразу следующий non-finished. Finished offline **не** держит turn (в отличие от active offline grace).
- Если все seats finished — `currentTurnSessionId` очистить или оставить последнего без move-gate (moves и так reject).

### D4 — Occupancy после finish

- **Выбор:** при построении occupied set учитывать только `!piece.finished`. Center cell сразу свободна.
- Disappear UX: client анимирует ход до center (существующий slide), затем короткий fade-out и снятие с DOM; strip слот сразу/после fade получает иконку.

### D5 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/schema/MyRoomState.ts` | `Piece.finished`, `Seat.finishPlace`, `nextFinishPlace` |
| `src/game/touristMove.ts` | occupancy helper ignore finished; optional `isCenterCell` в apply path |
| `src/rooms/MyRoom.ts` | `handleMove` finish branch; assign place when 4 finished; `advanceTurn` skip; move reject if seat finished / piece finished |
| `test/MyRoom.test.ts`, `test/touristMove.test.ts` | SC-FINISH-*, SC-MOVE-08/21–23, SC-PIECE-21 |

### D6 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `src/stores/game.ts` | mirror `finished` / `finishPlace`; helpers `isFinishedSeat`, piece finished by side |
| `src/pages/GamePage.vue` | board: hide finished; fade on finish; strip icons; place `q-dialog`; presence badge; `needsLeaveConfirm` = seated ∧ playing ∧ !finishPlace |
| `src/i18n/*` | модалка места (1-й / 2-й / …), a11y для иконок |
| assets / icon | простая иконка finish (флаг или check) + место на presence — Quasar icon или SVG в assets |

### D7 — Place modal

- **Выбор:** показать модалку локально при переходе `mySeat.finishPlace` с 0 → N (watch в GamePage / store event). Не broadcast отдельного message. Остальные видят place на presence.
- Комната не dispose при полном финише — finished сидят, пока сами не leave.

### D8 — Explore prerequisites

Все продуктовые D1–D10 из explore закрыты; открытых блокеров нет. Skills `work-with-game`, `work-with-game-board`, `work-with-messages`, `work-with-schema` — читать при apply.

## Risks / Trade-offs

- [Race: 4-я фишка + turn] → В одном `handleMove` сначала finish piece/seat, потом `advanceTurn` skip self.
- [Client видит finished на доске один кадр] → Снимать после анимации хода+fade; truth = synced `finished`.
- [Mid-game join при finished occupancy] → Считать `seats.size` как сейчас; finished не delete seat — поведение совпадает с D10.
- [Иконка без отдельного design system] → Минимальный badge; не блокер.

## Migration Plan

- Деплой server schema backward-compatible defaults (`finished=false`, `finishPlace=0`).
- Client и server выкатывать согласованно (иначе старый client не скроет finished / не покажет place).
- Rollback: revert siblings; старые комнаты без finish-полей ведут себя как pre-change.

## Open Questions

Нет (отложено в Out of scope proposal: lobby `status: finished`).
