## Why

Сейчас «чей ход» на Game — статичная синяя обводка presence, а время хода не ограничено: партия может зависать на AFK. У reconnect уже есть круговой countdown; нужен такой же понятный таймер хода, чтобы оба дедлайна были видны сразу и UI не скакал при появлении колец. До старта фигурки на доске лишние — рассадка без pieces до `playing`.

После внедрения dual rings presence всё ещё вокруг доски (left/right gutters): на мобилке доска сужается по ширине/высоте, layout прыгает, avatar мелкий, affordances и say-bubbles привязаны к старым слотам. Нужна верхняя полоса соперников, full-width доска и согласованный chrome маркеров.

Mid-game / mid-countdown подсадка ломает «закрытый стол» и solo 5:00 — новые seats только пока фаза `waiting`.

## What Changes

- Авторитетный таймер хода **60 с** в фазе `playing`: по истечении ход переходит следующему без хода фигурой; тикает и во время reconnect grace текущего.
- Когда остаётся один non-finished seated (остальные финишировали или вышли) — **5 мин** красный бюджет; по истечении — lock ходов + модалка «не успели довести туристов»; комната живёт, пока есть seats.
- Presence: обводка хода убирается; снаружи синий (или красный solo) ring, внутри warning reconnect; место под кольца всегда зарезервировано.
- Presence avatar: tourist PNG **всегда** виден на occupied marker — не прятать в слоте `q-circular-progress` без `show-value` и не вкладывать progress в progress.
- Pieces появляются только при переходе в `playing` (в `waiting` — seat без фигур; на materialize — всем сидящим).
- **Seating lock:** новый seat только в фазе `waiting` при `seats < maxSeats`. С начала `countdown` и во всём `playing` join → spectator (reconnect существующего seat без изменений). Leave после countdown/playing **не** открывает стол для новых seats.
- **Presence layout:** у seated — свой маркер снизу; все соперники одной полосой сверху (L→R по join order); у зрителя — все occupied markers одной полосой сверху; без left/right колонок вокруг доски.
- **Marker chrome:** avatar по размеру как strip-турист; turn/reconnect rings вокруг аватара; finish badge top-left у всех; ready affordance top-left только у себя; say affordance top-right только у себя.
- **Say bubbles:** всегда в сторону доски; зазор между маркерами достаточный, чтобы bubbles не перекрывались.
- **Board tile chrome:** gap **2px**, corner radius **2px**; full-width board на узком viewport.

## Scope

- **Пакеты:** client + server (согласованный контракт room `tourist`).
- **Capability ID:**
  - `game/move` — дедлайн хода; solo 5 мин; time-expired; turn order без mid-game append seats;
  - `game/presence` — rings; row layout; avatar/affordance chrome;
  - `game/pieces` — deferred pieces; **seating только в `waiting`**; leave/grace не reopen после countdown/playing;
  - `game/leave` — time-expired без confirm;
  - `game/start` — materialize на `playing`; countdown закрывает рассадку;
  - `game/finish` — finished seats + seating lock (нет mid-game seat после leave в playing);
  - `game/say` — bubbles к доске; say affordance top-right;
  - `game/board` — gap/radius 2px; full-width.
- **Client UX:** Game layout/chrome/timeout/leave.
- **Server:** timer/pieces + `onJoin` seating gate по phase.

## Out of scope

- Авто-kick / dispose по solo-timeout (только lock; dispose при seats = 0).
- Finish place для time-expired.
- Смена reconnect grace (30 с).
- Pass-кнопка; авто-ход по timeout.
- Отдельная фаза `finished` / lobby status под timeout.
- Новые say-пресеты под timeout.
- Sticky END-latch сверх phase gate (все finished / solo started) — отложено; сейчас достаточно `phase !== waiting`.
- Таблица рекордов; смена maxSeats.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: turn timer; solo budget; time-expired; no mid-playing seat append to turn order.
- `game/presence`: dual rings; row layout; avatar≈strip; affordance corners.
- `game/pieces`: deferred pieces until `playing`; **new seats only while `waiting`**; leave/grace after countdown/playing do not reopen seating.
- `game/leave`: no leave confirm for time-expired.
- `game/start`: materialize on `playing`; countdown implies seating closed for newcomers.
- `game/finish`: finished occupancy; no mid-game seat after leave once past waiting.
- `game/say`: bubbles toward board; say affordance top-right.
- `game/board`: gap/radius 2px; edge-to-edge board width.

## Impact

- **Server:** turn deadline; deferred pieces; `onJoin` rejects new seats when phase is `countdown` or `playing`; mocha seating + timer.
- **Client:** presence row/chrome; board gap/radius; timeout modal; skills/docs.
- **Контракт:** room `tourist` synced state; без новых HTTP/say messages.
- **Docs/skills:** OpenSpec meta + sibling AGENTS per `docs/projects-map.md`.

## References

- Explore (timer): D1–D7 / Q1; solo red / modal.
- Explore (layout): opponents top; self bottom; spectator all-top; avatar≈strip; corners; bubbles toward board; gap/radius 2.
- Explore (seating): S1=B — no new seats after start; S2 — no new seats once countdown started; reconnect OK.
- Main specs: `openspec/specs/game/{move,presence,pieces,leave,start,finish,board,say}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
