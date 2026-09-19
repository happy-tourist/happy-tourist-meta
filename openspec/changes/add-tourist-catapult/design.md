## Context

См. `proposal.md`. Сейчас: решётки (private seed + `holdingGrilleKeys` + `Piece.trapped`), push/rescue/return, create option `grilleDensity`. Катапульт нет. Explore D* закрыты — prerequisite ниже как решения.

Чеклист реализации — `tasks.md`.

**Пакеты:** server (`tourist` room) + client (Lobby create + Game board).

## Goals / Non-Goals

**Goals:**

- Create: независимый `catapultDensity` few/medium/many → **12/22/35%**, default medium.
- Seed скрытых катапульт только на `*` при `playing`; overlap с решётками OK.
- Единый land-resolve стека ловушек (move / push / return / post-fling / post-rescue).
- Fling: Chebyshev-2 free landable → else -1; else broken consume, piece stays.
- Public fade in/out ~**1000 ms**; broken sprite на vanish; ассеты `catapult.png` / `catapult-broken.png`.

**Non-Goals:**

- Новые типы ловушек; правка правил решёток вне совместного резолва; редактор карт; новые HTTP/auth.

## Decisions

### D1 — Плотность как у решёток

- **Выбор:** create option `catapultDensity: 'few' | 'medium' | 'many'` → те же `0.12 / 0.22 / 0.35`; count = `clamp(0, taskCount, Math.round(taskCount * p))` → **6 / 11 / 17** на стандартном layout.
- Переиспользовать `GRILLE_DENSITY_RATIO` / общий helper плотности, не дублировать магические числа.
- Default create: `medium`. Persist: private на room instance + seed once (как grille).

### D2 — Скрытие до reveal + минимальный sync

- Private `Set`/`Map` hidden catapult keys (зеркало `hiddenGrilleKeys`).
- Sync для UI: кратковременный reveal (ключи «сейчас анимируется» и/или событие/поле с флагом broken). Достаточно, чтобы все клиенты показали fade; после consume ключ убрать.
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

Fling **не** тратит лишний step. Rescue→catapult: только стоимость rescue.

### D4 — Геометрия fling

- Кандидаты: `chebyshevDistance === 2` затем `=== 1`.
- Фильтр: `isPlayableCell`, not in `removedTaskKeys`, not occupied by unfinished (excl. self when leaving cell).
- Center допустим → finish через существующий land-on-center path.
- Unit-pure helpers в `touristMove.ts` (или sibling): `catapultFlingCandidates`, `pickCatapultFlingDest`.

### D5 — Client UX / ассеты

| Ассет | Путь |
|-------|------|
| Intact | `happy-tourist.github.io/src/assets/catapults/catapult.png` |
| Broken | `happy-tourist.github.io/src/assets/catapults/catapult-broken.png` |

- `CATAPULT_ANIM_MS ≈ 1000` (fade in затем out; broken подменяет src на fade-out).
- Overlay на клетке при reveal; все клиенты.
- Piece travel на dest — существующий `MOVE_ANIM_MS`; finish — существующий finish travel.

### D6 — Lobby

- Второй option-group в create modal рядом с grille density; отдельные i18n keys (sense: катапульты мало/средне/много).
- `CreateGameOptions` + room `onCreate` parse оба density.

### D7 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/MyRoom.ts` | parse `catapultDensity`; seed on enterPlaying; land-resolve после move/push/return; post-rescue catapult; consume |
| `src/rooms/schema/MyRoomState.ts` | sync reveal keys / broken flag (минимально для анимации) |
| `src/game/touristMove.ts` | density reuse; fling candidate pick; optional stack shuffle helper |
| `test/MyRoom.test.ts` (+ unit) | SC-MOVE-78…89, SC-LOBBY-17, SC-FINISH-19, SC-BOARD seed-related server bits |

### D8 — Точки врезки (client)

| Место | Что |
|-------|-----|
| `src/pages/LobbyPage.vue` | catapult density UI |
| `src/stores/game.ts` | create option; mirror reveal; no new message if sync-driven |
| `src/pages/GamePage.vue` | overlay + fade/broken; piece reloc anim |
| `src/i18n/*` | lobby catapult density labels |
| `src/assets/catapults/*.png` | user-provided art |

### D9 — Skills при apply

Server: `work-with-schema`, `work-with-messages`, `work-with-game`, `work-with-rooms`, `server-work-with-test`.  
Client: `work-with-lobby`, `work-with-game-board`, `work-with-pages`, `work-with-stores`, `work-with-localization`, `colyseus-client`.  
Cross-package: **server contract first**, then client.

### D10 — Explore prerequisites (закрыты)

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
| D8 | broken fade-out, piece stays, consume |
| dest pick | at fire time |
| anim | opacity 0→1 then fade out ~1000 ms |

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Длинная цепочка fling→fling | One-shot + grille stop; цепочка ≤ числа катапульт |
| Sync spam на reveal | Короткоживущие keys / одно событие на consume |
| Race client anim vs sync relocate | Сначала sync coords + reveal flag; клиент анимирует по diff |
| Overlap densити сильно забивает поле | Продуктово принято; playtest later |

## Migration Plan

- Нет DB-миграций. Старые комнаты без `catapultDensity` → default `medium` на parse.
- Rollback: убрать option + seed/resolve; ассеты безопасно оставить.
