## Context

См. `proposal.md` — Why. Timer/deferred pieces уже в runtime; остаётся: (1) seating gate — новые seats только в `waiting`; (2) client polish — presence row, full-width board, avatar/rings, affordances, bubbles, gap/radius 2px.

Пакеты: **server** (seating gate + mocha) затем **client** (layout). Чеклист — `tasks.md` (блоки 5–7).

Explore prerequisites (закрыты): timer D1–D7 / Q1; layout D1–D9; seating S1=B / S2 (no seats once countdown; no seats in playing).

## Goals / Non-Goals

**Goals:**

- Synced `turnUntil` (+ бюджет solo vs 60s) и `timeExpired`; room clock → advanceTurn или lock (уже в коде).
- Deferred piece spawn на `playing` (уже в коде).
- **Seating:** `onJoin` выдаёт seat только при `phase === 'waiting'` и `seats.size < maxSeats`; иначе spectator. Reconnect существующего seat без изменений. Leave/grace после `countdown`/`playing` не открывают новые seats.
- Client: dual circular progress + sibling avatar; solo modal; mirror state в Pinia (уже в коде).
- Client layout polish: top opponents row / bottom self; spectator all-top; no left/right presence gutters; board full content width; avatar sized like strip tourist; rings scale around avatar; finish/ready/say corners; bubbles toward board; tile gap/radius 2px; stable marker chrome.

**Non-Goals:**

- Новые HTTP routes / новые Colyseus messages.
- Auto-kick / finish place для time-expired.
- Sticky END-latch (all-finished / solo-started) сверх phase gate — позже при необходимости.
- Новые npm-зависимости.
- Смена maxSeats / server say protocol.

## Decisions

### D1 — Synced deadline shape

- **Выбор:** room-level `turnUntil: number` (unix ms, `0` = нет активного таймера) + `Seat.timeExpired: boolean` (default false). Solo vs multi — по числу eligible seats; sync также `turnBudgetSeconds` (60 | 300).
- **Альтернатива:** только client-local timer — отвергнуто.

### D2 — Clock lifecycle

- При каждом назначении current turn в `playing` — clear/reschedule; budget 300s если ровно один eligible, иначе 60s; mid-turn «остался один» → свежие 5:00; offline grace не паузит; waiting/countdown без turn timer.

### D3 — Solo expiry

- `seat.timeExpired = true`; `turnUntil = 0`; reject `move`; room не dispose; client modal + clear selection; leave confirm без time-expired.

### D4 — Deferred pieces

- Join **только в `waiting`** — seat+kind без pieces; на `playing` — `materializePiecesForAllSeats()`. Join в `countdown`/`playing` — без seat (см. D13).

### D5 — Presence chrome (rings + avatar)

- **Sibling** rings + sibling `<img class="presence-avatar">` поверх (не default slot progress без `show-value`; не nested progress).
- **Avatar size:** как strip tourist (~72px image box); outer/inner `q-circular-progress` sizes подстраиваются вокруг аватара (outer > avatar, inner между avatar и outer).
- Reserved outer chrome: фиксированный box маркера = outer ring size — появление active progress не меняет layout size (SC-PRESENCE-11).
- Удалить CSS `--turn` box-shadow. Max outer = `turnBudgetSeconds`; value = remaining from `turnUntil - now`.

### D6 — Точки врезки (server)

| Место | Что |
|-------|-----|
| `src/rooms/schema/MyRoomState.ts` | `turnUntil`, `turnBudgetSeconds`; `Seat.timeExpired` |
| `src/rooms/MyRoom.ts` | schedule/clear turn timer; timeout → advance or expire; defer pieces; materialize on playing; move reject if timeExpired; **`onJoin` seating gate (D13)** |
| `test/MyRoom.test.ts` | SC-MOVE-25…30; SC-PIECE deferred + seating lock (19/08/…); SC-START-13; SC-MOVE-19 spectator |

### D13 — Seating only while waiting

- **Выбор (S1=B, S2):** новый seat iff `phase === 'waiting' && seats.size < maxSeats`. Иначе join — spectator (без kind/pieces).
- Reconnect в grace — как сейчас (не новый seat).
- Leave / grace timeout в `countdown` или `playing` освобождают kind/cells и occupancy, но **не** позволяют последующему join получить seat, пока phase не `waiting` (комната обычно не возвращается в waiting).
- Отдельный synced `seatingClosed` не нужен — достаточно `phase`.
- Альтернатива sticky END-latch — out of scope этой итерации.

### D7 — Точки врезки (client) — timer UX

| Место | Что |
|-------|-----|
| `src/stores/game.ts` | mirror `turnUntil`, `turnBudgetSeconds`, `timeExpired`; helpers |
| `src/pages/GamePage.vue` | rings + avatar; timeout modal; leave confirm gate |
| `src/i18n/*` | copy модалки timeout |

### D8 — Skills при apply

Server: `work-with-schema`, `work-with-game`, `work-with-rooms`, `server-work-with-test` (seating gate + mocha). Client polish: `work-with-game-board`, `work-with-styles`, `work-with-pages`, `client-align-code` / `client-verify-code`.

### D9 — Presence row layout (client)

- **Seated:** свой маркер — bottom row (одна позиция); соперники — одна top row, L→R по join order среди others; **нет** left/right presence slots.
- **Spectator:** все occupied seats — одна top row, L→R по join order; bottom пуст.
- **Board:** вне side gutters — `tourist-board` занимает полную ширину game content area на узком viewport (SC-BOARD-03); presence rows выше/ниже доски, не в колонках слева/справа.
- **Gap между маркерами:** горизонтальный зазор достаточный, чтобы say-bubbles соседних seats не перекрывались (flex + min-gap; при нехватке места — допустим горизонтальный scroll ряда, не сжатие аватара ниже strip size).

```
TOP:  [Opp…] [Opp…] [Opp…]     (spectator: all seats)
BOARD: full width, gap 2, radius 2
BOTTOM: [Me]                   (seated only)
```

### D10 — Affordance / badge corners

- Finish place badge: **top-left** на каждом маркере с `finishPlace > 0`.
- Ready affordance: **top-left** только на своём маркере (ожидание ready / countdown UX как сейчас); с finish по фазам не пересекается.
- Say affordance: **top-right** только на своём маркере; чужие markers без send affordance.

### D11 — Say bubble orientation

- Bubbles всегда **в сторону доски**: top-row markers → stack below avatar (toward board); bottom self → stack above avatar (toward board).
- Newer closer to avatar; не viewport toasts.
- Заменяет прежнюю ориентацию по left/right side slots.

### D12 — Board tile chrome

- `--gap: 2px`, `--radius: 2px` (было 6 / 12); max tile side 60px на wide viewport без изменений.
- Пересчитать `max-width` формулы доски под новый gap: `10 * 60px + 9 * 2px`.

## Risks / Trade-offs

- [Долгие mocha на 60s/300s] → ускорять константы / fake clock (уже в тестах).
- [Рассинхрон wall clock client] → remaining от synced `turnUntil`.
- [Пустые presence без avatar] → sibling img; Axis C hard defect.
- [4 крупных маркера на узком телефоне] → gap anti-overlap + optional horizontal scroll ряда; не уменьшать avatar ниже strip.
- [Прыжок layout] → убрать side columns; reserved marker box = outer ring size always.
- [Mid-game join expectations] → seating only in waiting; обновить mocha SC-PIECE-08/19 и связанные.

## Migration Plan

- Server schema defaults backward-compatible (уже выкатано).
- Client layout — согласованный деплой с обновлёнными specs/skills.
- Rollback: revert client GamePage/CSS (+ meta specs если нужно).

## Open Questions

Нет.
