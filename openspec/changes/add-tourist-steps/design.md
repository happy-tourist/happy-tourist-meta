## Context

Change `add-tourist-steps` ломает модель «один move = конец хода» и добавляет приватные бюджеты шагов/просмотров, peek на `*`, walkable holes и кнопку «Завершить ход». Explore-решения (D1–D12) зафиксированы в proposal; технических внешних блокеров (новых npm/провайдеров) нет.

## Goals / Non-Goals

**Goals**

- Server-authoritative steps/peeks, peek resolve, removed task cells, end-turn / auto-end / timeout-with-open-peek.
- Client UX: counters + «Завершить ход» у своего presence; глаз + stub-модалка; дыры на доске; keep-focus после move; анимации +N ~2 с; соло ∞ модалка.
- Private budgets: не в публичном `Seat` schema — room-private + `client.send` владельцу (паттерн как `turnOrder` / say).

**Non-Goals**

- Вопросы/ловушки/магазин; кап «до 10»; публичные чужие счётчики; смена таймеров layout; ambient-подсветка peekable клеток.

## Decisions

1. **Приватность бюджетов** — `Map<sessionId, { steps, peeks, infinite, peekedThisTurn }>` в room (не sync schema). На join/reconnect/change: `client.send('budgets', payload)`. UI только свой.
2. **Снятые тайлы** — sync всем: например `removedTaskKeys: string[]` или `Map` в schema (`"r,c"`), чтобы дыры видели spectators.
3. **Преген наград** — room-private `Map<cellKey, 1|2|3>` на transition в `playing`; shuffle мешка 28/14/6; клиенту только при открытии peek (в модалке / peek-open ack).
4. **Messages**
   - `move` — как сейчас + gate по steps / infinite.
   - `peek` — `{ side }` или `{ row, col }` (сервер сверяет piece на `*`).
   - `peekAnswer` — `{ correct: boolean }` для открытого peek.
   - `endTurn` — без payload.
5. **Не advance после move** — убрать `advanceTurn()` из успешного `handleMove`; вызывать из `endTurn`, auto-check, timeout; при grant хода: `steps++`, `peeks++` если не solo-infinite.
6. **Solo** — при `eligible === 1`: `infinite = true`, модалка budgets; без endTurn; 5:00 как сейчас; time-expired режет move/peek.
7. **Auto-end** — после каждого успешного move/peekAnswer и при смене budgets: если multi и нет legal move и нет legal peek → `advanceTurn()`.
8. **Timeout + open peek** — force `peekAnswer` incorrect → remove tile → `advanceTurn()`.
9. **Client анимации** — локально на рост `steps`/`peeks` (+1 grant, +N reward); длительность fall-in около **2 с** (`BUDGET_FALL_MS` / CSS); без отдельного sync-события «анимируй».
10. **Keep-focus после move** — после успешного `sendMove` **не** обнулять `selectedSide`, если кусок остался unfinished (не центр-финиш). После окончания move-anim снова показывать белую рамку; красные targets если steps>0 (или ∞); глаз если peeks позволяют и выбранный кусок на ещё живом `*`. Ambient-подсветка других peekable клеток **не** нужна — только иконка глаза над выбранным. Сброс selection: смена хода / not playing / finished / time-expired (как сейчас).

## Server

| Точка | Что сделать |
|-------|-------------|
| `src/rooms/schema/MyRoomState.ts` | Sync поле(я) removed task cells |
| `src/rooms/MyRoom.ts` | private budgets/rewards; messages; move без advance; endTurn; peek flow; solo infinite; auto-end; timeout force wrong |
| `src/game/touristMove.ts` (или соседний pure module) | playable includes removed `*`; helpers «есть legal move / peek» |
| `test/MyRoom.test.ts` | SC-MOVE-33… / SC-BOARD-07… |

Слои: room authoritative; pure rules вне I/O; schema только то, что должны видеть все.

## Client

| Точка | Что сделать |
|-------|-------------|
| `src/stores/game.ts` | listen `budgets`; send `peek` / `peekAnswer` / `endTurn`; mirror `removedTaskKeys` |
| `src/pages/GamePage.vue` | counters + кнопка у self presence; eye + dialog Correct/Wrong; tile hole render; keep-focus после move; +N anim ~2 с; solo modal; hide end-turn in solo |
| `src/i18n` | строки модалок / кнопки |

Слои: Colyseus I/O в Pinia; UI в GamePage (+ тонкие components при необходимости). Skills: `work-with-game-board`, `work-with-messages`, `work-with-schema`, `colyseus-client`, `work-with-localization`.

## Risks / Trade-offs

- Авто-end при `steps=0` и peeks>0 но не на `*` — игрок «сжигает» просмотры до следующего хода; принято явно.
- Room-private budgets проще filter schema, но нужны аккуратные resend на reconnect.
- Клиентские подсказки peek/move не truth — сервер отвергает.

## Migration

BREAKING для игроков: после одного шага ход не уходит — нужна кнопка/авто/таймер. Старые клиенты без `endTurn` застрянут на ходе до timeout — деплоить server+client вместе.

## Open questions

- (нет продуктовых; при apply уточнять только UX-мелочи копирайта модалок.)

## Tasks

Чеклист реализации — `tasks.md`.
