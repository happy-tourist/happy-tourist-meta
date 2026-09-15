## Why

Сейчас «чей ход» на Game — статичная синяя обводка presence, а время хода не ограничено: партия может зависать на AFK. У reconnect уже есть круговой countdown; нужен такой же понятный таймер хода, чтобы оба дедлайна были видны сразу и UI не скакал при появлении колец. До старта фигурки на доске лишние — рассадка без pieces до `playing`.

## What Changes

- Авторитетный таймер хода **60 с** в фазе `playing`: по истечении ход переходит следующему без хода фигурой; тикает и во время reconnect grace текущего.
- Когда остаётся один non-finished seated (остальные финишировали или вышли) — **5 мин** красный бюджет; по истечении — lock ходов + модалка «не успели довести туристов»; комната живёт, пока есть seats.
- Presence: обводка хода убирается; снаружи синий (или красный solo) ring, внутри warning reconnect; место под кольца всегда зарезервировано.
- Pieces появляются только при переходе в `playing` (в `waiting`/`countdown` — seat без фигур на доске); join уже в `playing` — сразу pieces.

## Scope

- **Пакеты:** client + server (согласованный контракт room `tourist`).
- **Capability ID:**
  - `game/move` — дедлайн хода, авто-pass 60 с, solo 5 мин, time-expired lock (reject move), клиентский clear selection;
  - `game/presence` — turn/reconnect rings, reserved chrome, убрать обводку текущего хода;
  - `game/pieces` — seat без pieces до `playing`; spawn при входе в `playing`;
  - `game/leave` — time-expired seat выходит без confirm (как finished);
  - `game/start` — при переходе в `playing` у уже сидящих появляются pieces (связь с pieces).
- **Client UX:** Game (presence rings, модалка timeout, board/strip до старта без фигур, leave).
- **Server:** room `tourist` — synced turn deadline / solo budget / time-expired; clock → advanceTurn или lock; spawn pieces на playing.

## Out of scope

- Авто-kick / dispose комнаты по истечении solo-таймера (только lock; dispose при seats = 0).
- Присвоение finish place time-expired игроку.
- Смена длительности reconnect grace (остаётся 30 с).
- Pass-кнопка для игрока; авто-ход фигуры по timeout.
- Отдельная фаза room `finished` / lobby status под timeout.
- Новые say-пресеты под timeout.
- Таблица рекордов / история партий вне комнаты.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: authoritative turn timer; 60 с → pass; solo 5 мин → lock + модалка; moves rejected when time-expired.
- `game/presence`: dual circular countdowns (turn + reconnect); reserved marker size; remove static turn outline.
- `game/pieces`: no board pieces until `playing`; spawn on phase transition; mid-`playing` join unchanged (seat + pieces).
- `game/leave`: no leave confirm for time-expired seated player.
- `game/start`: entering `playing` materializes pieces for seats that waited without them.

## Impact

- **Server:** synced turn deadline / solo vs normal budget / time-expired flag; room clock; piece spawn deferred to `playing`; mocha на timeout и deferred pieces.
- **Client:** Game presence chrome, i18n модалки, зеркало synced deadlines в game store; board/strip пустые до pieces.
- **Контракт:** room `tourist` через synced state (+ clock side-effects); без новых HTTP routes.
- **Docs/skills:** канон в OpenSpec meta; runtime в siblings per `docs/projects-map.md`.

## References

- Explore-решения: D1 pass; D2 tick during grace; D3 server deadline; D4 pieces after start; D5 lock + clear selection; D6 room like finish; D7 fresh 5:00; Q1 only `playing`; Q3–Q5 solo red / modal / normal finish.
- Main specs: `openspec/specs/game/{move,presence,pieces,leave,start,finish,board}/spec.md`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
- Карта путей: `docs/projects-map.md`.
