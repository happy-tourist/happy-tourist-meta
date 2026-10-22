## Why

Иногда после подтверждения создания игры диалог лобби зависает в состоянии «создаём», хотя `tourist`-комната на сервере уже появилась и создатель уже занял место. После обновления страницы игрок видит комнату в списке и может войти как второй участник (первое место удерживается grace reconnect). Причина: UI ждёт завершения отписки от lobby listing после успешного входа в игру; если `leave` лобби не резолвится, успех create не доходит до закрытия диалога и навигации.

## What Changes

- Успешный create/join в игру больше не блокируется ожиданием завершения отписки от lobby listing.
- После успешного входа в `tourist` клиент сразу считает create/join завершённым для UX (закрытие диалога создания, переход на игровой экран); отписка от лобби инициируется без блокирующего ожидания сети и ограничивается коротким timeout.
- Поведение распространяется на общий путь входа в игру (create и join), чтобы hang leave не воспроизводился на join.

## Scope

- **Capability ID:** `lobby/rooms`
- **Пакет:** client only (`happy-tourist.github.io`)
- **UX:** Lobby (диалог создания / вход в listed room) → переход на Game после успешного входа в `tourist`
- **Контракт room:** без изменений — reuse as-is: `lobby` (`LobbyRoom`, без reconnect hold), `tourist` (`MyRoom` + `enableRealtimeListing`), seating + tourist reconnect grace; leave listing = обычный Colyseus `leave()` (без новых messages/schema/HTTP)
- **Server / BE delta:** нет. Create/join `tourist`, live listing и leave лобби уже в main `openspec/specs/lobby/rooms/spec.md` (SC-LOBBY-04, SC-LOBBY-08) и в sibling server — этот change только меняет **когда** client перестаёт ждать завершения leave (background + short bound; orphan WS → disconnect / next subscribe)

## Out of scope

- Server seating / reconnect grace / realtime listing / LobbyRoom lifecycle
- Любые новые room messages, schema fields, HTTP routes
- Reclaim места создателя после refresh (reconnect token UX)
- Изменение правил create options (map/pack/sets/densities/maxSeats)
- Новый внешний сервис или npm-зависимость

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: уточнить SC-LOBBY-05 — успешный вход в игру и навигация не MUST ждать завершения close lobby subscription; отписка MUST быть инициирована при входе и не должна блокировать UX при зависшем leave

## Impact

- Client: store-путь входа в `tourist` + UX диалога создания / join на Lobby (закрытие loading и навигация на Game без ожидания leave лобби)
- Server: без изменений (reuse существующего create/join/listing/leave; apply только в `happy-tourist.github.io`)
- Спеки: delta `lobby/rooms` (client leave timing); после sync — main `openspec/specs/lobby/rooms/spec.md`
- Skills (meta): обновить client-канон leave-on-enter (`work-with-rooms`, `work-with-lobby`, `colyseus-client`, …) — иначе apply останется на blocking ожидании leave лобби

## References

- Main spec: `openspec/specs/lobby/rooms/spec.md` (SC-LOBBY-05)
- Client AGENTS: `../happy-tourist.github.io/AGENTS.md`
- Explore: intermittent create hang + join-as-second после refresh; выбран background leave + short timeout (см. `design.md` D1/D3)
