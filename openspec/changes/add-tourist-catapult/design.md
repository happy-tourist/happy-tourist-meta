## Context

См. `proposal.md`. Секции 1–6 (seed/fling/lobby/client land→overlay→fling + D13) уже в runtime. Playtest: ход уходит до анимации; full-chain sync показывает решётку на финале «заранее».

**Пакеты:** server (`tourist`) + client (Game board). Follow-up §7: **server-paced** hop + **deferred turn**.

## Goals / Non-Goals

**Goals:**

- Paced trap pipeline: на клетке — только следующий trap; после presentation budget — эффект; fling-dest = новый land (без −1 step).
- Общие presentation ms client↔server (без `presentationDone`).
- Auto-end и deadline-triggered advance — только после pipeline idle (если переход уже «нужен»).
- Один sequential timeline: catapult hops затем grille; board lock.
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

**Authority timing:** server clock задаёт момент coords/holding; client анимирует в тех же ms. Синтез full-chain из одного patch больше не нужен как основной путь (D13 atomic mirror остаётся страховкой).

### D6 — Lobby

- Второй density selector. *(done)*

### D7 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `MyRoom.ts` | paced trap pipeline; presentation budgets; deferred `advanceTurn` / deadline |
| `MyRoomState.ts` | reveal/holding как сейчас; не слать всю цепочку разом |
| `touristMove.ts` | fling helpers / density *(done)* |
| `test/MyRoom.test.ts` | SC-MOVE-90…92 + регресс 78…89 |

### D8 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `GamePage.vue` | queue следует hop-sync; grille не раньше своего land; board busy на pipeline |
| `game.ts` | mirror; без presentationDone message |
| skills / AGENTS | paced + deferred turn blurbs |

### D9 — Skills при apply §7

Server: `work-with-game`, `work-with-messages`, `work-with-rooms`, `server-work-with-test`.  
Client: `work-with-game-board`, `work-with-stores`, `colyseus-client`.

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
- Если auto-end или deadline уже «должен» сменить ход — поставить `pendingTurnAdvance` (или эквивалент) и выполнить при `onTrapPipelineIdle`.
- **Не** pause/extend wall-clock `turnUntil` (Out of scope).
- **Не** client ack.
- Push/return/rescue land — тот же pipeline и тот же defer.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Длинная цепочка × budget | Продуктово без cap; playtest |
| Тесты с реальным clock | mocha accelerate timeouts / stub clock как у turn budgets |
| Client FPS отстаёт от server unlock hop | Краткий visual lag принят; ход всё равно общий |
| Старый client reconstruction | Упростить queue; не ломать D13 fallback |

## Migration Plan

- Нет DB. Rollback: вернуть sync full-chain resolve (хуже UX).
- §7 — server + client + skills; секции 1–6 остаются.

## Open Questions

- (нет — S1/S2 закрыты explore)
