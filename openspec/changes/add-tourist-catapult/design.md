## Context

См. `proposal.md`. Базовый seed / `resolveCellTraps` / lobby density / sync reveal уже в runtime. Gap: клиент показывает overlay параллельно с мгновенным sync-relocate → фишка «пропадает» во время fade. Follow-up — presentation sequencing + board input lock.

Чеклист — `tasks.md` (секции 1–4 done; секция 5 — follow-up).

**Пакеты:** server (`tourist` room) + client (Lobby create + Game board). Server authority без anim-delay; presentation — client.

## Goals / Non-Goals

**Goals:**

- Create: независимый `catapultDensity` few/medium/many → **12/22/35%**, default medium.
- Seed скрытых катапульт только на `*` при `playing`; overlap с решётками OK.
- Единый land-resolve стека ловушек (move / push / return / post-fling / post-rescue).
- Fling: Chebyshev-2 free landable → else -1; else broken consume, piece stays.
- **Presentation:** после полного vanish overlay — piece travel (`MOVE_ANIM_MS`) / finish travel; цепочка catapult→catapult последовательно; broken hold 300+300; board lock на любой board-анимации.
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

**Successful fling (есть dest):**

1. Reveal overlay на клетке выстрела (~fade in→out, порядок ~**1000 ms** как сейчас / grille magnitude).
2. Пока overlay жив: piece **pinned** визуально на клетке катапульты (local override; sync dest уже известен).
3. После полного vanish: снять pin → CSS travel `MOVE_ANIM_MS` на sync dest; если center — существующий finish travel + disappear **после** vanish (не параллельно с overlay).
4. Цепочка: приземление на новую катапульту → снова полный overlay → затем следующий travel (без cap длины).

**Broken (нет free ring-2/1):**

1. Appear целая.
2. Hold **300 ms** целая.
3. Swap → broken.
4. Hold **300 ms** broken.
5. Vanish.
6. Piece остаётся на клетке (нет travel).

**Authority:** client-only presentation delay (проще). Краткий visual desync ~1 с принят.

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
| `src/stores/game.ts` | create option; mirror reveal; no new message if sync-driven |
| `src/pages/GamePage.vue` | overlay + broken holds; **pin→delay→travel**; board busy lock; finish after vanish |
| `src/i18n/*` | lobby catapult density labels |
| `src/assets/catapults/*.png` | user-provided art |

### D9 — Skills при apply

Server: `work-with-schema`, `work-with-messages`, `work-with-game`, `work-with-rooms`, `server-work-with-test`.  
Client: `work-with-lobby`, `work-with-game-board`, `work-with-pages`, `work-with-stores`, `work-with-localization`, `colyseus-client`.  
Cross-package: **server contract first**, then client. Follow-up: обновить game-board / styles / AGENTS blurbs про sequencing + lock.

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
| Anim-D2 | Цепочка последовательно (каждый выстрел: vanish → travel) |
| Anim-D3 | Client-only delay (не server clock) |
| Broken | Appear → 300 ms intact → break → 300 ms broken → vanish; no travel |
| Center | Vanish first, then finish travel+fade |
| Input | Board non-interactive while **any** board anim runs (move, grille, catapult, finish, fling travel) |
| Chain cap | Нет |

### D12 — Board input lock

Сейчас `moveAnimating` только на own submit (~250 ms / 2× rescue-push) — **дыра** на grille/catapult/finish. Расширить до общего board-busy: `isInteractive` false, пока живы move travel, grille drop/rise, catapult overlay (+ broken holds), finish disappear, pending fling travel после pin. На чужом ходе и так не interactive.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Длинная цепочка fling→fling × ~1 s | Продуктово без cap; playtest |
| Client pin vs server dest (другая фишка на «пустую» клетку катапульты) | Принято ради простоты; short window |
| Sync spam на reveal | Короткоживущие keys |
| Nested timers pin/travel/chain | Очередь презентаций на client; один active catapult play за раз на piece |

## Migration Plan

- Нет DB-миграций. Старые комнаты без `catapultDensity` → default `medium` на parse.
- Rollback: убрать option + seed/resolve; ассеты безопасно оставить.
- Follow-up presentation — только client (+ skills); server coords logic без изменений.
