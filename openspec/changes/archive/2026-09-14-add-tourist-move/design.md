## Context

См. `proposal.md` (Why / Scope) и delta specs `game/move`, `game/board`. Сейчас: room `tourist` синхронизирует `started` + `seats` (pieces с `side`/`row`/`col`, connectivity); game messages нет; `GamePage` рисует LAYOUT локально с `pointer-events: none`; очередь хода и перемещение отсутствуют. `started` по-прежнему только «рассадка закрыта на 4-м seated», не фаза партии. Пакеты: **server** (`../happy-tourist-server`), **client** (`../happy-tourist.github.io`), **meta** (skills под контракт хода).

## Goals / Non-Goals

**Goals:**

- Авторитетный one-step move + sync текущего хода на server; pure rules + тонкий Room glue.
- Client: выбор своей фигурки (поле/strip), локальные белая/красная рамки только на своём ходу, `send` хода из Pinia, плавная анимация переезда у всех.
- Согласованный client↔server контракт message + schema без legacy draughts encoding.

**Non-Goals:**

- Start-event / ready; финал на центре; пас; авто-скип offline; sync selection/hints; новые npm-анимационные библиотеки; HTTP/auth.

## Decisions

### D1 — Sync текущего хода: `currentTurnSessionId`

- В `MyRoomState` добавить строковое поле текущего хода = `sessionId` seated-игрока (пустая строка, если seated нет).
- Очередь = порядок появления seats (стабильный join order). Хранить явный упорядоченный список sessionId в room (не в schema, если Map iteration недостаточен) **или** schema-поле + room-side array; предпочтение: **room-private `turnOrder: string[]`** + sync только `currentTurnSessionId` (меньше schema churn).
- Альтернатива: sync всего `turnOrder` — отвергнута для MVP (клиенту достаточно «чей ход»).

### D2 — Message `move`: `{ side, row, col }`

- Client → server: `room.send('move', { side: 'N'|'E'|'S'|'W', row, col })` — `side` идентифицирует фигурку отправителя; `row`/`col` — цель.
- Не использовать legacy draughts `{ from, to }` / cell encoding.
- Reject без изменения state при: не seated, не текущий ход, чужой/отсутствующий side, нелегальная цель.

### D3 — Pure rules module на server

- Новый модуль e.g. `src/game/touristMove.ts` (или `src/rooms/tourist/moveRules.ts`): playable cells (start + task + center cells layout, согласованный с client LAYOUT), Chebyshev distance === 1, occupancy set, apply.
- `MyRoom.onMessage('move')` только парсит, вызывает pure validate/apply, пишет schema, двигает turn.
- Layout geometry **дублируется** на server как константа playable cells (как уже `START_CELLS`); client LAYOUT остаётся UI-истиной для рендера/hints, не authority.

### D4 — Turn lifecycle hooks

- `onJoin` (новый seat): append в `turnOrder`; если это первый seated → `currentTurnSessionId = sessionId`.
- Permanent seat remove (`onLeave` consented / grace timeout): убрать из `turnOrder`; если удалённый был current → следующий в круге (или `''` если seats пусты → dispose как сейчас).
- `onDrop` / offline: **не** менять `currentTurnSessionId` (ход ждёт).
- `started` не гейтит ходы.

### D5 — Client hints local-only

- Selection (`selectedSide`) и computed legal targets — только в UI/store локально у клиента, чей `sessionId === currentTurnSessionId`.
- Legal targets на client: та же соседняя логика + локальный LAYOUT + mirrored seats occupancy (hint, не truth). Server остаётся арбитражем.
- Снять `pointer-events: none` с pieces/tiles/strip для ходящего; зрители и «не твой ход» — без submit.

### D6 — Анимация: CSS transform, ~250ms

- Pieces позиционировать так, чтобы смена клетки анимировалась (`transform` / absolute offsets от grid), не полагаться на tween `grid-row`/`grid-column`.
- Длительность ~200–300ms ease-out; без новых deps.
- На время анимации **своего** хода у отправителя — флаг `moveAnimating` / ignore clicks; у остальных анимация тоже видна, input и так неактивен вне их хода.
- После sync смены `row`/`col` все клиенты играют travel from previous→new (хранить prev coords per piece key).

### D7 — Meta skills lockstep

- Обновить `work-with-game`, `work-with-messages`, `work-with-schema`, `work-with-game-board`, `colyseus-client` / `work-with-stores` по необходимости: message `move`, `currentTurnSessionId`, interactive board, animation notes.
- Чеклист реализации — в `tasks.md`, не дублировать здесь.

## Risks / Trade-offs

- [Расхождение client LAYOUT vs server playable set] → нелегальный hint или ложный reject — mitigation: одна таблица клеток в design/tasks; mocha на playable/start/center; визуально сверить с LAYOUT.
- [Map iteration order seats ≠ join order] → mitigation: явный `turnOrder[]` в Room.
- [Клиент шлёт ход во время анимации / double-send] → ignore input + optional ignore duplicate until turn changes.
- [Центр как 2×2 span в UI vs 4 логические клетки] → для правил центр = четыре клетки (4,4)(4,5)(5,4)(5,5); UI span остаётся визуалом.

## Prerequisites (explore)

Все продуктовые D* из explore закрыты (ходы сразу; центр walkable без финала; chrome только ходящему; соло; свои блокируют; без паса; offline ждёт; анимация у всех; клики во время анимации игнор). Технических блокеров (libs/infra) нет.

## Migration Plan

1. Server: schema + turnOrder + pure rules + `onMessage('move')` + mocha SC-MOVE-*.
2. Client: mirror `currentTurnSessionId`, sendMove, selection/hints, animation, interactive Game UI.
3. Meta skills под контракт.
4. Rollback: git revert change в трёх репо (message/schema обратно несовместимы с новым client — деплоить согласованно).

## Open Questions

- Нет (решения зафиксированы в proposal / explore).
