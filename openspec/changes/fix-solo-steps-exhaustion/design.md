## Context

See `proposal.md` — Why. Runtime: server `../happy-tourist-server` (room `tourist`), client `../happy-tourist.github.io` (модалка по `timeExpired`).

Симптом explore: соло, ordinary last-step → center; остаток на старте + один trapped; steps=0; действий нет; **solo deadline продолжает тикать** → `timeExpired` не выставлен (иначе `clearTurnDeadline`).

Текущее состояние (канон уже близко к SC-MOVE-48):

- `MyRoom.maybeSoloStepsExhausted`: solo ∧ steps===0 ∧ !`openPeek` ∧ !`hasLegalPeek` → `timeExpired` + `clearTurnDeadline`.
- `hasLegalPeek` (`touristMove.ts`) уже **пропускает** `trapped` / finished; старт не `isTask` → legal peek false.
- После ordinary `handleMove` на center: `applyCenterFinish` → `ensureCurrentEligibleOrAdvance` → при solo `maybeSoloStepsExhausted`.
- Mocha SC-MOVE-48: last step со старта на соседнюю **не-center** → timeExpired; **нет** фикстуры last-step → **center** + jail + starts.
- Client: `watch(isMySeatTimeExpired)` открывает модалку; `hasOwnUnfinishedOnLiveTask` **не** исключает `trapped` → риск неверной timer-copy, но при отсутствии `timeExpired` модалки нет вовсе.

## Goals / Non-Goals

**Goals:**

- Гарантировать: после ordinary last-step center finish (и любого пути, где steps=0 без legal peek) solo seat сразу `timeExpired`, deadline снят (SC-MOVE-97/98).
- Mocha-репро репорта; при красном тесте — починить server gate/call-site.
- Client: infer steps-exhausted copy без учёта trapped (SC-PRESENCE-35); модалка при `timeExpired` без регресса SC-PRESENCE-21.

**Non-Goals:**

- Менять jail/rescue/all-jail/peeks∞/5‑min budget как правила экономики.
- Multi auto-end / catapult pipeline turn pass.
- Скрывать ∞ на HUD после expire (можно later UX).

## Decisions

### D1 — Сначала mocha SC-MOVE-97 (и SC-MOVE-98), потом правка кода

| Вариант | Почему |
|---------|--------|
| **A (выбран):** red→green от репортной фикстуры | Симптом однозначен; текущий код *выглядит* корректным — без теста легко «починить» не то |
| B: сразу рефакторить `maybeSoloStepsExhausted` | Риск слепой правки при возможном call-site / `openPeek` / eligibility баге |

Фикстура SC-MOVE-97: solo eligible; steps=1; piece A на center-ring adjacent to free center (напр. `(3,4)`→`(4,4)`); piece B on start; piece C trapped (helper `setTrapped` / аналог); D finished или на старте; ordinary `move` A→center; assert `timeExpired`, `steps===0`, `turnUntil===0` (deadline cleared), reject further move/peek. Существующий SC-MOVE-48 (off-center last step) оставить зелёным; при удобстве добавить assert `turnUntil===0` и туда.

SC-MOVE-98: steps=0; единственный unfinished на live `*` — trapped; **вызвать** solo-exhaustion evaluation → `timeExpired` + `turnUntil===0`. В HEAD есть только `evaluateAutoEndForTests` → `maybeAutoEndTurn` (multi); публичного hook на `maybeSoloStepsExhausted` нет — **добавить** `evaluateSoloStepsExhaustedForTests()` (зеркало auto-end hook), вызывать его из mocha после расстановки pieces/budgets. Не полагаться на multi auto-end.

Если SC-MOVE-97 уже green на текущем коде — расширить поиск (H2–H4 ниже) и всё равно оставить тест как regression; client SC-PRESENCE-35 всё равно.

### D2 — Server: выровнять критерий solo end с «нет legal peek»

Канон exhaustion: `steps===0` ∧ нет legal peek (free unfinished on live `*`). `hasLegalPeek` уже skip trapped — оставить как source of truth; **не** подменять «любой unfinished on `*`» из старой формулировки main SC-MOVE-48 и **не** «чинить» `hasLegalPeek`, чтобы trapped на `*` считался legal.

JSDoc у `maybeSoloStepsExhausted` в HEAD устарел («no unfinished on live `*`») — при правке метода выровнять комментарий на `!hasLegalPeek` (trapped/start не держат seat).

Точки врезки (при красном тесте проверять по порядку):

1. Call-site после ordinary center в `handleMove` (уже есть: `applyCenterFinish` → `ensureCurrentEligibleOrAdvance` → при solo `resyncTurnDeadline({ preserveSoloBudget: true })` → `maybeSoloStepsExhausted`) — подтвердить, что ветка solo вызывается при частичном finish (`finishPlace===0`); полный finish place → seat ineligible, exhaustion не нужен.
2. **`openPeek` MUST NOT block solo steps exhaustion** (решение, не «по тесту»): при `steps===0` ∧ `!hasLegalPeek` server MUST clear/force-close orphaned `openPeek` for that seat (incorrect resolve без remove tile — как multi timeout SC-MOVE-42 — или эквивалентный null + sync), затем выставить `timeExpired` + `clearTurnDeadline`. HEAD early-return `if (this.openPeek) return` в `maybeSoloStepsExhausted` — известный риск H2; убрать/обойти при красном 97/98 или отдельной orphaned-peek проверке.
3. Не опираться на `seatHasAvailableActions` / `maybeAutoEndTurn` для solo (`countEligible < 2` уже no-op) — solo end только через `maybeSoloStepsExhausted` (+ timer).

Анти-паттерн: дублировать turn-advance / finish place в center branch; не трогать multi pending-absorb; не считать trapped «живой задачей» в критерии exhaustion; не оставлять soft-lock из‑за висящего `openPeek` при steps=0 без legal target.

### D3 — Client: `hasOwnUnfinishedOnLiveTask` исключает trapped

В Game presence end-modal infer: unfinished ∧ present task ∧ **!trapped** (зеркало server `hasLegalPeek`, который уже skip trapped). Тогда SC-PRESENCE-35 даёт steps-copy. Модалка по-прежнему только на переходе `timeExpired` false→true (server fix первичен; без exclude trapped после fix откроется timer-copy).

Контракт helper (pure, вне `GamePage.vue`):

- Вход: own `pieces` (`finished` / `trapped` / `row` / `col`), `steps`, и predicate `isPresentTaskCell(row, col)` (как на странице: `boardGeometry.isTask` ∧ ключ не в `removedTaskKeys`).
- Legal-peek target: `!finished` ∧ `!trapped` ∧ `isPresentTaskCell`.
- Kind: `steps <= 0` ∧ нет legal-peek target → `'steps'`; иначе `'timer'` (в т.ч. steps>0 при timer expiry).
- Имена ок: `hasOwnUnfinishedOnLiveTask` + `inferSoloEndModalKind` (или один модуль `src/lib/soloEndModalKind.ts`).

Vitest (pattern A / `client-work-with-test` → `composables.md`): **не** монтировать GamePage; кейс steps=0 + только trapped на `*` → kind `steps`; start/path remainder → `steps`; регресс free unfinished на live `*` + steps=0 → kind `timer` (не steps).

Не трогать: i18n keys `game.stepsExhaustedModal*` / `game.timeExpiredModal*` (уже есть); `watch(isMySeatTimeExpired)` open/clear selection; leave-confirm `!timeExpired`. Orphaned peek UI: server clear `openPeek` → `:model-value="Boolean(game.openPeek)"` закрывается сам — отдельной client-ветки под exhaustion не нужно.

Канон infer уже в meta skill `work-with-game-board` (SC-PRESENCE-21/35); при apply не откатывать формулировку к «любой unfinished on `*`».

### D4 — Гипотезы, если mocha green без server diff

| ID | Гипотеза | Как проверить |
|----|----------|---------------|
| H1 | Регресс уже в коде; репорт на старом build | green 97 + deploy |
| H2 | `openPeek` держал exhaustion | логи/тест с orphaned peek |
| H3 | Client не показал модалку при выставленном expired | маловероятно при тикающем timer |
| H4 | У игрока был free на `*`, peek UI сломан | вне scope; peeks∞ должен давать open |

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Ложное timeExpired при steps=0 и free на `*` | `hasLegalPeek` must stay true → no expire; mocha opposite case |
| Orphaned `openPeek` блокирует end | D2.2: clear/force-close then expire; mocha 97/98 или orphaned fixture |
| Неверная timer-copy после fix | SC-PRESENCE-35 / exclude trapped |
| Регресс SC-MOVE-48 off-center | оставить существующий mocha зелёным |

## Migration Plan

- Деплой server (+ client при правке infer); без protocol version bump.
- Rollback: revert room + GamePage infer.

## Open Questions

- Нет (продуктовый D1 из explore закрыт: нет шагов и нет legal peek → end; совпадает со SC-MOVE-48 уточнённым).

Чеклист реализации — `tasks.md`.
