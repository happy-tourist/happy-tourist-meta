## Context

См. `proposal.md` — Why. Timer, deferred pieces, seating gate и presence/board layout уже в runtime. Остаётся client polish: (1) compact leave icon; (2) починка клика say (overflow / hit-area).

Пакеты: **client** only для остатка. Чеклист — `tasks.md` (блок 8).

Explore prerequisites (закрыты): timer D1–D7 / Q1; layout D1–D9; seating S1=B / S2; polish H1 icon-only / H2 `logout` / S1 leave+say.

## Goals / Non-Goals

**Goals:**

- Synced `turnUntil` (+ бюджет solo vs 60s) и `timeExpired`; room clock → advanceTurn или lock (уже в коде).
- Deferred piece spawn на `playing` (уже в коде).
- **Seating:** `onJoin` выдаёт seat только при `phase === 'waiting'` и `seats.size < maxSeats`; иначе spectator. Reconnect существующего seat без изменений. Leave/grace после `countdown`/`playing` не открывают новые seats.
- Client: dual circular progress + sibling avatar; solo modal; mirror state в Pinia (уже в коде).
- Client layout polish: top opponents row / bottom self; spectator all-top; no left/right presence gutters; board full content width; avatar sized like strip tourist; rings scale around avatar; finish/ready/say corners; bubbles toward board; tile gap/radius 2px; stable marker chrome (уже в коде).
- **Leave:** Game header exit — Material icon `logout` without visible label; accessible name «Выход из игры» (стабильная одна строка хедера на мобилке).
- **Say:** own-marker speech affordance and preset picker remain pointer/touch activatable; presence row overflow MUST NOT clip them; hit-area usable on touch.

**Non-Goals:**

- Новые HTTP routes / новые Colyseus messages.
- Auto-kick / finish place для time-expired.
- Sticky END-latch (all-finished / solo-started) сверх phase gate — позже при необходимости.
- Новые npm-зависимости.
- Смена maxSeats / server say protocol.
- Скрытие roomId; смена статуса; другие иконки leave.

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

Server: `work-with-schema`, `work-with-game`, `work-with-rooms`, `server-work-with-test` (seating gate + mocha). Client polish: `work-with-game-board`, `work-with-styles`, `work-with-pages`, `work-with-localization`, `client-align-code` / `client-verify-code` (leave icon + say hit).

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

### D14 — Compact leave control (client)

- **Выбор (H1/H2):** на Game header `q-btn` exit — `icon="logout"`, **без** `:label`; accessible name через `aria-label` / i18n `game.leave` («Выход из игры»). Тот же Material set, что Lobby logout.
- **Альтернативы:** `exit_to_app` / `meeting_room` — out of scope; короткий текст «Выход» — отвергнуто (H1 = icon only).
- Точка врезки: `GamePage.vue` header; skills `work-with-pages` / `work-with-localization` при упоминании label leave.

### D15 — Say affordance hit + no overflow clip (client)

- **Проблема:** `.presence-row { overflow-x: auto }` часто форсирует clip по Y → affordance/picker над/под маркером не принимают тач или невидимы.
- **Выбор:** ряд не клипает chrome маркера (overflow visible на ряду; при необходимости горизонтальный scroll на отдельной обёртке, не на том же боксе что клипает Y). Affordance и picker: `pointer-events: auto`, z-index выше rings/avatar; hit-area ≥ ~32–44 CSS px на тач (визуал иконки может остаться ~22px).
- Точка врезки: `GamePage.vue` presence CSS + say button/picker; skill `work-with-game-board`.

## Risks / Trade-offs

- [Долгие mocha на 60s/300s] → ускорять константы / fake clock (уже в тестах).
- [Рассинхрон wall clock client] → remaining от synced `turnUntil`.
- [Пустые presence без avatar] → sibling img; Axis C hard defect.
- [4 крупных маркера на узком телефоне] → gap anti-overlap + optional horizontal scroll ряда; не уменьшать avatar ниже strip.
- [Прыжок layout] → убрать side columns; reserved marker box = outer ring size always; leave icon-only снижает wrap хедера.
- [Mid-game join expectations] → seating only in waiting; обновить mocha SC-PIECE-08/19 и связанные.
- [Say overflow clip] → D15: не совмещать overflow-x scroll и visible Y на одном элементе.

## Migration Plan

- Server schema defaults backward-compatible (уже выкатано).
- Client layout / leave / say — согласованный деплой с обновлёнными specs/skills.
- Rollback: revert client GamePage/CSS (+ meta specs если нужно).

## Open Questions

Нет.
