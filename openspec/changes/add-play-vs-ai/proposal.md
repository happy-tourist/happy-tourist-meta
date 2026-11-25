## Why

Игроки хотят начать партию сразу, без ожидания живых соперников в лобби. Нужен режим «против AI»: один человек и остальные места за столом заполняются компьютерными соперниками, которые ходят и отвечают на задания в темпе, похожем на живого игрока.

## What Changes

- В окне создания игры — выбор соперников: живые люди или AI (радио с иконками); create option `opponentMode: 'people'|'ai'` (omit → people); при `maxSeats=1` сервер coerce в effective people/solo (без ботов / unlist / AI join-deny).
- В **effective AI** (`ai` ∧ `maxSeats≥2`) сервер сразу заполняет свободные места ботами (`Seat.isBot`); партия стартует как полный стол; `setPrivate` — комната не в live-листинг; чужой join — hard deny (не spectator).
- Боты играют как эгоистичные Easy-соперники (спасение, возврат с финиша, знания, cut), задания решают верно, с human-like задержками (отдельный move-delay на каждое действие); в presence все боты — «AI».
- Уход единственного человека из effective-AI комнаты (любая фаза) сразу dispose — боты комнату не держат.

## Scope

**Пакеты:** client (Lobby create UX, Game presence/имена) и server (room `tourist`, seats, start, bot turn/peek, lobby metadata visibility).

**Capability ID:**

- `game/ai` (новый) — режим против AI, bot seats, политика Easy, задержки, always-correct задания
- `lobby/rooms` — create opponent-mode; AI-комнаты не в listing
- `game/start` — заполнение ботами → full table → countdown
- `game/presence` — отображение имени «AI» для bot seats
- `game/leave` — dispose AI-комнаты при уходе человека
- `game/pieces` — bot seats в ёмкости maxSeats; без human/spectator join в AI-режиме

**Контракт:** room name `tourist`; create `opponentMode`; synced `Seat.isBot`; JWT `onAuth` для человека без изменений; боты — synthetic seats (не отдельные клиенты); HTTP/БД под ботов нет.

## Out of scope

- Отдельные процессы / `@colyseus/loadtest` как прод-боты
- Несколько уровней сложности (Hard) и полный оптимальный solver
- Смешанный стол «люди + AI» в одной комнате
- Разные отображаемые имена у ботов («AI 2»…)
- Изменение правил знаний / cut / turn economy для людей
- Новый HTTP API / БД под ботов

## Capabilities

### New Capabilities

- `game/ai`: создание и жизненный цикл партии против AI; synthetic bot seats; Easy-эвристики хода; human-like задержки; корректное авто-решение заданий ботом

### Modified Capabilities

- `lobby/rooms`: radio люди/AI; create `opponentMode`; maxSeats=1 coerce people; effective-AI unlist (`setPrivate`)
- `game/start`: после seat человека в effective AI боты добивают maxSeats → countdown; ready не нужен
- `game/presence`: `isBot` → подпись «AI»
- `game/leave`: уход человека из effective AI (любая фаза) → немедленный dispose
- `game/pieces`: bot seats + `isBot`; hard deny чужой join/spectator в effective AI

## Impact

- Client: Lobby create radio после maxSeats (`person`/`smart_toy`, default people) + i18n; `createGame({ opponentMode })`; Game presence caption «AI» под аватаром bot seats (`isBot`)
- Server: room `tourist` create/join/listing/seating/start/leave + bot controller на ходе и open-on-land peek
- Без новых npm-зависимостей и без внешних AI-сервисов

## References

- Explore-решения (серверные seats, Easy, unlist, no spectators, dispose on leave, имя «AI», задержки-массивы)
- `docs/projects-map.md`; sibling `AGENTS.md` (client / server)
- Main specs: `openspec/specs/lobby/rooms`, `game/start`, `game/move`, `game/presence`, `game/leave`, `game/pieces`, `game/finish` (cut / knowledge — поведение людей без изменения правил)
