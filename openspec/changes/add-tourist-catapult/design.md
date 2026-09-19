## Context

См. `proposal.md`. Seed / `resolveCellTraps` / lobby density / sync reveal и черновая client queue (секции 1–5) уже в runtime. **Defect:** overlay часто не играется — фишка сразу на fling dest. Follow-up (секция 6): надёжный detect reveal + порядок land → overlay → fling для всех клиентов.

**Пакеты:** server (`tourist` room) + client (Lobby create + Game board). Server authority без anim-delay; presentation — client.

## Goals / Non-Goals

**Goals:**

- Create: независимый `catapultDensity` few/medium/many → **12/22/35%**, default medium.
- Seed скрытых катапульт только на `*` при `playing`; overlap с решётками OK.
- Единый land-resolve стека ловушек (move / push / return / post-fling / post-rescue).
- Fling: Chebyshev-2 free landable → else -1; else broken consume, piece stays.
- **Presentation:** у игрока и spectator одинаково: **сначала** визуальный доезд на клетку катапульты → **потом** overlay (successful ~**1000 ms**; broken 300+300) → **потом** fling / finish travel; цепочки с доездом на каждую следующую катапульту; board lock на всю последовательность.
- Надёжный enqueue несмотря на split Pinia mirror (`seats` до `revealingCatapultKeys`).
- Ассеты `catapult.png` / `catapult-broken.png`.

**Non-Goals:**

- Новые типы ловушек; правка правил решёток вне совместного резолва; редактор карт; новые HTTP/auth.
- Server clock delay для coords/`finished`.

## Decisions

### D1 — Плотность как у решёток

- **Выбор:** create option `catapultDensity: 'few' | 'medium' | 'many'` → те же `0.12 / 0.22 / 0.35`; count = `clamp(0, taskCount, Math.round(taskCount * p))` → **6 / 11 / 17** на стандартном layout.
- Переиспользовать `GRILLE_DENSITY_RATIO` / общий helper плотности, не дублировать магические числа.
- Default create: `medium`. Persist: private на room instance + seed once (как grille).

### D2 — Скрытие до reveal + минимальный sync

- Private `Set`/`Map` hidden catapult keys (зеркало `hiddenGrilleKeys`).
- Sync для UI: кратковременный reveal (ключи «сейчас анимируется» и/или событие/поле с флагом broken). Достаточно, чтобы все клиенты показали overlay; после consume ключ убрать.
- Hidden locations never synced.
- **Альтернатива отвергнута:** держать permanent broken markers на доске — продукт: гаснет и исчезает.

### D3 — Единый `resolveCellTraps(piece, cell)`

Порядок на сервере после принятого land (move/push/return) и после successful rescue (если piece free на клетке):

1. Собрать оставшиеся unspent traps на клетке (grille, catapult).
2. Shuffle порядок.
3. Пока список не пуст и piece ещё на этой клетке и free (не finished): взять следующий trap.
4. Grille → существующий trap path (может all-jail).
5. Catapult → consume; pick dest; relocate или broken stay; если relocate — **рекурсивно/циклически** `resolveCellTraps` на dest (как новый land).
6. Catapult one-shot: удалить из hidden сразу при resolve.

Fling **не** тратит лишний step. Rescue→catapult: только стоимость rescue. Server пишет coords/`finished` **сразу** (без anim delay).

### D4 — Геометрия fling

- Кандидаты: `chebyshevDistance === 2` затем `=== 1`.
- Фильтр: `isPlayableCell`, not in `removedTaskKeys`, not occupied by unfinished (excl. self when leaving cell).
- Center допустим → finish через существующий land-on-center path.
- Unit-pure helpers в `touristMove.ts`: `catapultFlingCandidates`, `pickCatapultFlingDest`.

### D5 — Client UX / ассеты / presentation sequencing

| Ассет | Путь |
|-------|------|
| Intact | `happy-tourist.github.io/src/assets/catapults/catapult.png` |
| Broken | `happy-tourist.github.io/src/assets/catapults/catapult-broken.png` |

**Порядок на каждом выстреле (игрок = spectator):**

1. **Land / доезд** на клетку катапульты с travel sense обычного шага (`MOVE_ANIM_MS`; push/return — их arrival sense). Sync может уже держать piece на fling dest — client синтезирует доезд (pin/visual override).
2. **Только после** завершения доезда — overlay на клетке выстрела.
3. **Successful:** appear→vanish суммарно ~**1000 ms** (одна презентация, не 1000+1000).
4. Пока overlay жив: piece **pinned** на клетке катапульты.
5. После полного vanish: fling travel `MOVE_ANIM_MS` на sync dest; center → finish travel+fade **после** vanish.
6. Цепочка: fling land на новую катапульту → снова доезд на ту клетку → overlay → travel (без cap).

**Broken:** после доезда — appear → **300 ms** intact → broken → **300 ms** broken → vanish; piece остаётся (нет fling travel).

**Уже на клетке** (напр. post-rescue, доезжать некуда): не синтезировать фейковый шаг; overlay после завершения текущей arrival-анимации (если ещё идёт), иначе сразу.

**Authority:** client-only presentation delay. Краткий visual desync принят.

### D6 — Lobby

- Второй option-group в create modal рядом с grille density; отдельные i18n keys (sense: катапульты мало/средне/много).
- `CreateGameOptions` + room `onCreate` parse оба density.

### D7 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/MyRoom.ts` | parse `catapultDensity`; seed on enterPlaying; land-resolve после move/push/return; post-rescue catapult; consume |
| `src/rooms/schema/MyRoomState.ts` | sync reveal keys / broken flag (минимально для анимации) |
| `src/game/touristMove.ts` | density reuse; fling candidate pick; shuffle helper |
| `test/MyRoom.test.ts` (+ unit) | SC-MOVE-78…89, SC-LOBBY-17, SC-FINISH-19 |

### D8 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `src/pages/LobbyPage.vue` | catapult density UI |
| `src/stores/game.ts` | create option; mirror reveal; optional atomic snapshot для catapult watch |
| `src/pages/GamePage.vue` | queue: **land → overlay → fling**; broken holds; board busy; finish after vanish; spectator parity |
| `src/i18n/*` | lobby catapult density labels |
| `src/assets/catapults/*.png` | user-provided art |

### D9 — Skills при apply

Server: `work-with-schema`, `work-with-messages`, `work-with-game`, `work-with-rooms`, `server-work-with-test`.  
Client: `work-with-lobby`, `work-with-game-board`, `work-with-pages`, `work-with-stores`, `work-with-localization`, `colyseus-client`.  
Follow-up §6: game-board / styles / AGENTS — land-before-overlay + enqueue fix.

### D10 — Explore prerequisites (закрыты, seed/rules)

| ID | Решение |
|----|---------|
| D1 | overlap OK; random order each resolve; count like grille |
| D2 | only free landable (no holes/edge/occupied) |
| D3 | post-fling = ordinary land (move/push sense) |
| triggers | move, push, return |
| center | fling = finish |
| D5b | same 12/22/35, default medium, separate selector |
| D6 | one-shot disappear |
| D7 | task `*` only |
| D8 | broken consume, piece stays |
| dest pick | at fire time |

### D11 — Presentation follow-up (закрыты)

| ID | Решение |
|----|---------|
| Anim-D1 | Travel **после** полного исчезновения overlay |
| Anim-D2 | Цепочка последовательно (каждый выстрел: land → overlay → travel) |
| Anim-D3 | Client-only delay (не server clock) |
| Anim-D4 | **Доезд на клетку катапульты завершается до начала overlay**; затем vanish; затем fling |
| Spectator | Та же последовательность, что у игрока (синтез land при необходимости) |
| Triggers UX | move / push / return land — сначала «как шаг» на клетку, потом катапульта |
| Broken | Appear → 300 ms intact → break → 300 ms broken → vanish; no travel |
| Successful ms | Суммарно ~1000 ms appear→vanish |
| Center | Vanish first, then finish travel+fade |
| Input | Board non-interactive while **any** board anim runs (land, grille, catapult, finish, fling) |
| Chain cap | Нет |
| Already on cell | Нет фейкового шага; overlay после текущей arrival-анимации или сразу |

### D12 — Board input lock

Общий board-busy: `isInteractive` false, пока живы land/move travel, grille drop/rise, catapult overlay (+ broken holds), finish disappear, deferred fling travel. На чужом ходе и так не interactive для текущего игрока; **презентация** всё равно идёт у всех зрителей.

### D13 — Root cause missed overlay + fix

**Баг:** `_mirrorRoomState` пишет `seats` затем `revealingCatapultKeys`. Watch с `flush: 'sync'` на комбинированном getter срабатывает дважды: (1) pieces уже на fling dest, revealing ещё пуст; (2) revealing появился, но prev/next pieces уже оба на dest → `resolveFlingPieceKey` → `null` → silent skip.

**Fix (выбрать минимальный работающий):**

- Не полагаться на «кто стоит на клетке» после relocate: атрибутить piece по переходу coords / last-known / единственному moved в том же patch; **и/или**
- Один атомарный снимок mirror (seats+revealing в одном реактивном тике) / `flush: 'pre'` без промежуточного fire; **и/или**
- Не silent-skip: fallback enqueue overlay даже без piece key (хотя бы artwork).

Land-before-overlay строится поверх рабочего enqueue.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Длинная цепочка land+overlay+fling × N | Продуктово без cap; playtest |
| Client pin vs server dest | Принято; short window |
| Sync spam на reveal | Короткоживущие keys |
| Split mirror / sync flush | D13 fix обязателен в §6 |
| Spectator без локального beginMove | Синтез land travel к клетке катапульты |

## Migration Plan

- Нет DB-миграций. Старые комнаты без `catapultDensity` → default `medium` на parse.
- Rollback: убрать option + seed/resolve; ассеты безопасно оставить.
- Presentation follow-up §6 — только client (+ skills); server coords logic без изменений.
