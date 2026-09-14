## Why

Перезагрузка страницы или краткий обрыв сокета сейчас воспринимаются как полный выход: seat и фигурки снимаются, после `joinById` игрок часто становится зрителем. Нужно уметь кратко восстановить то же место и показывать остальным, что соперник временно офлайн.

## What Changes

- Неожиданный disconnect seated-игрока **не** снимает seat сразу: grace **30 секунд** с сохранением pieces и возможностью вернуться в тот же seat.
- Сознательный выход с экрана Game (кнопка в лобби) — **сразу** снимает seat (как сейчас по смыслу leave).
- После окончательного leave/timeout: если seated не осталось, а есть только зрители — комната закрывается; если кто-то seated остался — партия продолжается без отключившегося.
- На Game — кружки занятых игроков (presence) с офлайн-индикатором и круговым countdown на время grace; зрители тоже видят кружки.
- Лобби: без reconnect-hold; обрыв listing не показывает `seat reservation expired` / reconnect-шум; при необходимости тихое переподключение к списку.

## Scope

- **Capability ID:** `game/pieces` (правка leave/reconnect); `game/presence` (**новый** — UI presence вокруг стола); `lobby/rooms` (правка — лобби без reconnect / без шума reservation).
- **Пакеты:** server (room `tourist`, synced seat connectivity; **не** LobbyRoom grace); client (Game reconnect + presence; lobby — тихий resubscribe); meta skills при необходимости.
- **Контракт:** room `tourist` — reconnect; room `lobby` — только live listing, без hold seat.

## Out of scope

- Конец партии, победа/поражение, forfeit как отдельное правило сверх leave/timeout.
- Ходы, очередь ходов, задания на тайлах.
- Persist seat по userId без Colyseus reconnection (альтернативная модель).
- Grace длиннее/короче 30 с; «держать seat до конца игры» без лимита.
- **Reconnect / allowReconnection для LobbyRoom** — лобби не «охраняет» подписчика.
- Новые HTTP API / auth / theme.
- Статусы на личной strip×4 («в игре» / «прошёл») — только presence-кружки за столом.

## Capabilities

### New Capabilities

- `game/presence`: кружки занятых игроков на Game, раскладка для seated и spectator, офлайн + countdown на grace.

### Modified Capabilities

- `game/pieces`: политика leave — consented сразу; unexpected — 30 с hold + reconnect; dispose при 0 seated; sync connected/deadline.
- `lobby/rooms`: обрыв listing-подписки не держит reservation; не показывать пользователю `seat reservation expired` / reconnect-шум; при необходимости тихое переподключение к listing.

## Impact

- Server: lifecycle disconnect/reconnect **только** room `tourist`; schema seat connectivity; mocha на grace / consented / empty-seated dispose; LobbyRoom без grace.
- Client: tourist token + reconnect + presence; lobby — без token persist, без user-facing reservation/reconnect ошибок listing.
- Meta: skills rooms (tourist vs lobby policy) / board / schema.

## References

- Explore (чат): D1–D4, Q1–Q9 — grace 30 с, presence layout, QCircularProgress, sessionStorage; лобби без reconnect / без reservation-шума.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
- Main specs: `openspec/specs/game/pieces/spec.md`, `openspec/specs/game/board/spec.md`, `openspec/specs/lobby/rooms/spec.md`.
