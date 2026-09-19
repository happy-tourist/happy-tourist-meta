## Context

См. `proposal.md`. Секции 1–7 (seed/fling/lobby/paced hop/deferred turn) уже в runtime. Playtest после §7: ход зависает после grille, снявшей peek; fling→центр без finish travel.

**Пакеты:** server (`tourist`) + client (Game board). Follow-up §8: **idle re-eval auto-end** + **always finish travel**.

## Goals / Non-Goals

**Goals:**

- Paced trap pipeline + deferred pending advance (§7) — остаются.
- На pipeline idle: pending → advance; иначе **re-eval** `maybeAutoEndTurn` / solo (F1).
- Каждый catapult hop анимируется; fling→центр — **всегда** finish travel после последнего vanish (F2).
- Экономика steps/peeks без изменений.

**Non-Goals:**

- Client ack для хода; pause/extend wall-clock `turnUntil`; новые ловушки; смена fling geometry / density.

## Decisions

### D1 — Плотность как у решёток

- Create `catapultDensity` → `0.12 / 0.22 / 0.35`; default medium; reuse grille ratio helpers. *(done)*

### D2 — Скрытие до reveal + минимальный sync

- Private hidden sets; short-lived `revealingCatapultKeys` / broken; hidden never synced. *(done)*
- Reveal живёт на hop до конца presentation budget; не чистить mid-hop так, чтобы оборвать overlay.

### D3 — Paced `resolveCellTraps` (замена мгновенной full-chain)

После принятого land (move / push / return / post-rescue) и после каждого fling-land:

1. Если pipeline уже занят для этой piece/room — не стартовать второй параллельный resolve.
2. Собрать unspent traps на **текущей** клетке; если пусто → `onTrapPipelineIdle` (maybe deferred turn).
3. Shuffle; взять **один** следующий trap (не while по всей цепочке в одном тике).
4. **Catapult:** consume from hidden; sync reveal (+ broken if no dest); **не** relocate сразу; `clock.setTimeout(presentationBudget)` → затем fling relocate или stay; clear reveal; если fling — center finish или schedule resolve **только** dest cell.
5. **Grille:** после budget «arrival на эту клетку» (если нужен) → holding + trapped (существующий path); затем idle.
6. Fling **не** −1 step. Rescue→catapult: только стоимость rescue.

**Отвергнуто:** мгновенный while/recursion всей цепочки + client reconstruction финала.  
**Отвергнуто:** client `presentationDone` как источник отпуска хода.

### D4 — Геометрия fling

- Ring-2 then ring-1; free landable; center → finish. Helpers в `touristMove.ts`. *(done)*  
- Dest pick **at fire time** того hop (после budget, перед relocate).

### D5 — Client UX / presentation

| Ассет | Путь |
|-------|------|
| Intact | `…/catapults/catapult.png` |
| Broken | `…/catapults/catapult-broken.png` |

**Порядок на hop (игрок = spectator):**

1. Visual land/arrival на клетку ловушки (`MOVE_ANIM_MS` sense; push/return — их arrival).
2. Overlay: successful ~**1000 ms**; broken 300+300 (+ vanish).
3. Fling travel `MOVE_ANIM_MS` **после** vanish (или grille drop после своего hop).
4. Следующий hop только после sync следующего reveal/holding — не заранее.
5. **Fling→центр (F2):** finish travel + fade **после** vanish того hop’а (и после prior hops в цепи). Не полагаться на `finished` в том же tick, что reveal — при paced finish приходит позже; после vanish проверить finished/center и запустить тот же path, что move/push→центр. Не телепортировать / не снимать pin без travel.

**Authority timing:** server clock задаёт момент coords/holding; client анимирует в тех же ms. Синтез full-chain из одного patch больше не нужен как основной путь (D13 atomic mirror остаётся страховкой).

### D6 — Lobby

- Второй density selector. *(done)*

### D7 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `MyRoom.ts` | paced pipeline; deferred pending; **idle re-eval** auto-end/solo (F1) |
| `MyRoomState.ts` | reveal/holding как сейчас |
| `touristMove.ts` | fling helpers / density *(done)* |
| `test/MyRoom.test.ts` | SC-MOVE-90…93 + регресс |

### D8 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `GamePage.vue` | hop-sync; grille defer; **finish travel after vanish even if finish sync late** (F2) |
| `game.ts` | mirror; без presentationDone |
| skills / AGENTS | idle re-eval + finish-after-vanish blurbs |

### D9 — Skills при apply §8

Server: `work-with-game`, `work-with-rooms`, `server-work-with-test`.  
Client: `work-with-game-board`.

### D10 — Explore prerequisites seed/rules — закрыты

*(без изменений; fling без −1 step подтверждён S2)*

### D11 — Presentation UX — закрыты (уточнение)

| ID | Решение |
|----|---------|
| Anim-D1…D4 / spectator / broken / lock | как в §5–6 |
| **Anim-D3** | **Снято:** client-only delay при мгновенном server full-chain. Канон: **server-paced** + общие ms |
| Chain | sequential; no cap |
| Grille | только на своём hop после prior catapult hops |

### D12 — Board input lock

`isInteractive` false на весь pipeline (land, catapult, fling travel, grille drop/rise, finish). Во время pipeline действия хода не принимать (сервер тоже не advance’ит).

### D13 — Missed overlay (client) — done §6

Atomic seats+revealing / attribution / no silent skip — остаётся.

### D14 — Presentation budget (общие ms)

Канон констант (имена могут жить в server export + client mirror):

| Hop phase | Budget |
|-----------|--------|
| Land/arrival sense | `MOVE_ANIM_MS` (как обычный шаг) |
| Successful catapult overlay | ~**1000 ms** |
| Broken | 300 + 300 + vanish (~как сейчас) |
| Fling travel | `MOVE_ANIM_MS` |
| Grille drop | `GRILLE_ANIM_MS` (1000) |

Сервер ждёт сумму фаз **до применения эффекта следующего sync** так, чтобы клиент успел показать текущий hop. Точная нарезка (один timeout на hop vs несколько) — implementation detail, поведение — specs.

### D15 — Deferred turn advance

- **Не** звать `advanceTurn` из `maybeAutoEndTurn` / deadline handler, пока trap pipeline active.
- Если auto-end или deadline уже «должен» сменить ход — поставить `pendingTurnAdvance` и выполнить при idle.
- **Не** pause/extend wall-clock `turnUntil`; **не** client ack.
- Push/return/rescue land — тот же pipeline и defer.

### D16 — Idle re-eval after traps (F1) — §8

На `onTrapPipelineIdle`:

1. `trapPipelineActive = false`.
2. Если `pendingTurnAdvance` — сбросить флаг и `advanceTurn` (deadline / заранее no-actions). **Не** глотать deadline re-eval’ом.
3. Иначе — `maybeAutoEndTurn()` и (solo) `maybeSoloStepsExhausted()`.

Закрывает: land с peeks на live `*` → grille → peek недоступен → без re-eval ход застревает.

### D17 — Always finish travel under paced sync (F2) — §8

- Enqueue на reveal **не** обязан видеть `finished` сразу.
- После successful vanish: если piece уже finished / sync dest center — `markDeferredFinish` + finish travel с клетки катапульты (как SC-FINISH-01/19), не clear pin без travel.
- Цепочка catapult→…→center: анимация **каждого** hop; finish travel только после **последнего** vanish.
- Spectator = игрок.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Длинная цепочка × budget | Продуктово без cap; playtest |
| Тесты с реальным clock | mocha accelerate timeouts / stub clock как у turn budgets |
| Client FPS отстаёт от server unlock hop | Краткий visual lag принят; ход всё равно общий |
| Старый client reconstruction | Упростить queue; не ломать D13 fallback |
| Finish sync mid-overlay | Pin + deferred finish until vanish (D17) |

## Migration Plan

- Нет DB. §8 — точечные server idle + client finish path; 1–7 остаются.

## Open Questions

- (нет — F1/F2/Q1/Q2 закрыты explore)
