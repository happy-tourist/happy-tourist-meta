## Context

See `proposal.md` — Why. Runtime: server `../happy-tourist-server` (room `tourist`). Client только зеркалит `finishPlace` / `currentTurnSessionId`; UI place-модалки менять не нужно.

Было (до фикса):

- Обычный ход на центр: `applyCenterFinish` → сразу `ensureCurrentEligibleOrAdvance()` в `handleMove` / push / return / peek-path.
- Катапульта → центр: `fireCatapultAfterPresentation` → `applyCenterFinish` → `onTrapPipelineIdle()` → только `maybeAutoEndTurn()` / solo exhaustion.
- `maybeAutoEndTurn` при `!isEligibleForTurn(current)` делает early `return` **без** передачи хода.
- Если `pendingTurnAdvance` уже true (deadline / early no-actions), idle вызывает `advanceTurn` и баг маскируется — отсюда плавающий симптом.

Стало: idle следует канону D1 (`wasCurrent` → `ensureCurrentEligibleOrAdvance` → pending absorb / условный `advanceTurn`); mocha SC-MOVE-96 / SC-FINISH-21 + pending-absorb зелёные.

## Goals / Non-Goals

**Goals:**

- На idle trap-pipeline снять ход с seat, получившего finish place mid-pipeline, если есть другой eligible — через `ensureCurrentEligibleOrAdvance` (как после ordinary center finish); helper попутно покрывает прочих ineligible (time-expired) без расширения product-scope delta.
- Pending absorb per D1: не более одного turn-advance на idle (SC-MOVE-96).
- Mocha: SC-MOVE-96 / SC-FINISH-21 (fourth piece via catapult→center + remaining actions so `pendingTurnAdvance` не выставлен заранее).

**Non-Goals:**

- Менять paced budgets, fling pick, client animations, schema/messages.
- Дублировать turn-advance внутри `applyCenterFinish` (оставить единую точку на idle / existing handlers).

## Decisions

### D1 — Точка врезки: `onTrapPipelineIdle`

Вызвать `ensureCurrentEligibleOrAdvance()` в начале idle (после сброса `trapPipelineActive`), **до** ветки `pendingTurnAdvance` / `maybeAutoEndTurn`.

| Вариант | Почему |
|---------|--------|
| **A (выбран):** `ensureCurrentEligibleOrAdvance()` на idle | Паритет с `handleMove` после center finish; покрывает finish place без `pendingTurnAdvance` |
| B: только внутри catapult center branch | Узже; легко забыть другие пути (будущий trap→finish) |
| C: править `maybeAutoEndTurn` «если ineligible → advance» | Смешивает auto-end с eligibility pass; риск двойного advance с `pendingTurnAdvance` |

При `pendingTurnAdvance`: после `ensureCurrentEligibleOrAdvance` текущий уже eligible (или пусто) — `advanceTurn` из pending всё ещё нужен для «ход уже должен был уйти по no-actions/deadline»; если ensure уже сменил current на следующего, `advanceTurn` сдвинет ещё на одного — **риск**.

**Уточнение порядка (обязательно):**

1. Снять `trapPipelineActive`.
2. Всегда вызвать `ensureCurrentEligibleOrAdvance()` (no-op, если current ещё eligible; одноразовая передача с финишера / time-expired, если нет).
3. Если был `pendingTurnAdvance`:
   - если после шага 2 current **всё ещё** тот же session, что держал ход в начале idle **и** он eligible → `advanceTurn` (классический deferred auto-end/deadline);
   - если шаг 2 уже сменил current (финишер → соперник) → **очистить** `pendingTurnAdvance` **без** второго `advanceTurn` (ход уже ушёл с финишера; повторный advance украл бы ход у соперника). Это **уточняет** main SC-MOVE-92: pending idle = не более одного логического ухода хода, не слепой второй `advanceTurn`.
4. Иначе → `maybeAutoEndTurn` + solo exhaustion как сейчас.

**Анти-паттерн (не делать):** «всегда `ensure`, затем если `pending` и current eligible → `advanceTurn`» — после finish-place ensure уже сменил current на соперника (eligible), и слепой pending-`advanceTurn` украдёт ход у соперника (двойной advance). Сравнивать с `wasCurrent` обязательно.

**Семантика pending:** pending = «этот ход уже должен был закончиться» (deadline / early no-actions). Если seat получил finish place mid-pipeline, ход «закончился» через ensure — pending **absorb** (clear without second advance). Если seat остался eligible (например grille убрал peek), pending → `advanceTurn` или idle re-eval — как SC-MOVE-91/93.

Итоговая формула idle (канон; править только её):

```
trapPipelineActive = false
wasCurrent = currentTurnSessionId
ensureCurrentEligibleOrAdvance()
if pendingTurnAdvance:
  pendingTurnAdvance = false
  if phase playing && countEligible >= 2:
    if currentTurnSessionId == wasCurrent && isEligibleForTurn(wasCurrent):
      advanceTurn(...)   # deferred auto-end/deadline still owed
    # else: ensure already moved off finished/expired — do not advance again
  return
maybeAutoEndTurn()
maybeSoloStepsExhausted()
```

### D2 — Тест-фикстура

Добавить mocha рядом с SC-MOVE-89 / SC-FINISH-19 (не ломая их): два seat; у A три piece с `finished=true`, четвёртый free, **`finishPlace === 0` до fling** (eligibility = `finishPlace` / timeExpired, не `piece.finished`); land на planted catapult с fling-only-to-center (как SC-MOVE-89 ring block); `setBudgetsForTests` с остатком actions после land (не провоцировать `pendingTurnAdvance` до idle); после idle assert `finishPlace > 0` (обычно `1`) и `currentTurnSessionId` = next eligible ≠ A; helpers: `isTrapPipelineActiveForTests` / при pending-кейсе — `isPendingTurnAdvanceForTests`.

### D3 — Client

Нет правок UI/i18n/анимаций. Два независимых sync-binding’а уже в client:

- place-модалка (`GamePage` watch `mySeat.finishPlace` 0→N) — открывается при назначении места **mid-pipeline** (после `applyCenterFinish`); **не** гейтить модалку ожиданием idle;
- turn chrome / `isMyTurn` — по `currentTurnSessionId`; после этого fix chrome уходит с финишера **на idle** (SC-MOVE-96).

Краткое перекрытие «модалка уже открыта, chrome ещё на A до idle» — ожидаемо (paced pipeline; SC-MOVE-92), не FE-баг и не повод править client.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Двойной `advanceTurn` (ensure + pending) | D1: advance pending только если current не сменился |
| Регресс SC-MOVE-91/93 | Существующие mocha + явный assert в новом тесте на path с/без pending |
| Solo full finish | `countEligible < 2` / ensure → пустой или тот же solo path; не трогать solo deadline clear отдельно сверх существующих finish rules |

## Migration Plan

- Деплой server only; клиенты получают новый sync без версии протокола.
- Rollback: revert room idle change.

## Open Questions

- Нет (блокеров из explore нет; решение D1 зафиксировано).

Чеклист реализации — `tasks.md`.
