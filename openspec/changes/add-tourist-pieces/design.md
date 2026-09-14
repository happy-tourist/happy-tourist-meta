## Context

См. `proposal.md` (Why / Scope). Сейчас: client `GamePage` — статичный LAYOUT без фигурок; ассеты `src/assets/tourists/tourist{1-4}.png` уже на месте; `stores/game` join/leave без product schema; server `MyRoom` stub + `MyRoomState.mySynchronizedProperty`; room `tourist` без seating. Пакеты: **server** (authority seating/sync), затем **client** (рендер по state).

## Goals / Non-Goals

**Goals:**

- Авторитетная рассадка ≤4 в `tourist` + флаг start на 4-й seat.
- Sync seats → все клиенты рисуют фигурки; seated — strip «мой турист»; spectator — без strip.
- Leave: до start вернуть pool; после start не раздавать; убрать фигурку ушедшего.

**Non-Goals:**

- Ходы / «дошёл до центра» / reconnect policy / forfeit; лимит `maxClients`; новые room messages.

## Decisions

### D1 — Server authority + schema seats

- Заменить scaffold state на product fields, например:
  - `started: boolean`
  - `seats: Map` keyed by `sessionId` → `{ touristId: 1..4, side: "N"|"E"|"S"|"W", row, col }`
- Назначение только в room `onJoin` / снятие в `onLeave`; client только читает sync.
- Альтернатива (client-local random) — отвергнута: гости и rejoin разойдутся.

### D2 — Пулы и рандом

- До `started`: из оставшихся `touristId` и `side` выбрать равновероятно; клетку — равновероятно из 4 стартов стороны (геометрия как LAYOUT client: N row0 cols3–6; E col9 rows3–6; S row9 cols3–6; W col0 rows3–6).
- При 4-й выдаче выставить `started = true` (и при желании metadata `status: "playing"` для лобби).
- После `started` join не пишет в `seats`.

### D3 — Leave

- `!started`: delete seat → kind/side снова доступны.
- `started`: delete seat; `started` остаётся true; новым seats не давать.
- Альтернатива «оставить ghost piece» — отвергнута для этого change (убрать из sync).

### D4 — Client render

- `stores/game`: зеркалировать seats/`started`/sessionId из `onStateChange`; Colyseus I/O не в page.
- `GamePage`: оверлей фигурок на CSS Grid (img по `touristId` → `@/assets/tourists/touristN.png`); strip под доской только если `seats.has(mySessionId)`.
- Доска остаётся non-interactive (`pointer-events: none` на тайлах/фигурках).
- Альтернатива: отдельный component — по желанию, если page раздуется; не обязателен.

### D5 — Без лимита клиентов

- Не ставить `maxClients = 4`; политика «игроков ≤ 4» только через seats/`started`.
- JWT `onAuth` без изменений.

### D6 — Skills / канон (meta, по необходимости)

- Обновить `work-with-game-board` / `work-with-schema` / `work-with-game` коротко: фигурки sync, seating, не draughts; детали — в tasks если затронуты apply-файлы.

## Risks / Trade-offs

- [Schema scaffold → product без dual client update] → пустой UI — mitigation: один change, server contract first, затем client.
- [Несовпадение индексов стартов client/server] → фигурка «в дыре» — mitigation: зафиксировать таблицу клеток стороны в design/tasks + тест SC-PIECE-03.
- [Race двух join до start] → Colyseus `onJoin` последователен в room — ок; тесты на uniqueness.

## Migration Plan

1. Server: schema + MyRoom seating + mocha SC-PIECE-*.
2. Client: store mirror + GamePage pieces/strip; lint/typecheck.
3. Rollback: revert schema/UI; room снова без seats.

## Technical prerequisites (из explore)

- PNG `tourist1`…`tourist4` уже в client `src/assets/tourists/` — готово.
- Решения D1–D4 explore закрыты (см. proposal References).

## Open Questions

- Нет.
