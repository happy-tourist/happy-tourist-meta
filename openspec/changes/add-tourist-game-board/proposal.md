## Why

Продукт «Счастливый турист» больше не является онлайн-шашками: на экране игры нужно показать новое статическое игровое поле, а все продуктовые и агентные упоминания шашек — убрать или заменить, чтобы лобби и комната по-прежнему стыковались при ручной проверке.

## What Changes

- На экране игры появляется поле «Счастливый турист» заданной формы (старты, тайлы заданий, сплошной центр) без фигурок, кликов и правил хода.
- **BREAKING:** игровой room type переименовывается с `checkers` на `tourist` на клиенте и сервере (create / join / lobby filter / тесты / loadtest), чтобы список комнат и вход в партию продолжали работать.
- Продуктовый язык и канон агентов (skills, AGENTS, docs, OpenSpec context) переводятся с шашек на настольную игру «Счастливый турист»; skill правил сервера переименовывается в `work-with-game`.
- Шашечная логика выбора клетки / подсказок хода / отправки `move` с игрового экрана убирается до следующих change с правилами.

## Scope

- **Capability ID:** `game/board` (новый); `lobby/rooms`; `ui/theme`; `auth/login` (имя комнаты в сценарии join).
- **Пакеты:** client (поле + контракт room `tourist`); server (регистрация room `tourist` и согласование listing/тестов/loadtest без правил игры); meta (docs / skills / AGENTS / openspec context).
- **Client:** экран Game — статичное поле; лобби/game store — имя комнаты `tourist`; без UX хода шашек.
- **Server:** имя playable room `tourist` + realtime listing; без новой механики board/move.
- **Контракт:** room name `tourist` (вместо `checkers`); schema/messages правил — без расширения в этом change.

## Out of scope

- Правила хода, валидация, переворот тайлов с заданиями, контент заданий.
- Фигурки игроков и расстановка (в т.ч. рандом по сторонам) — позже; максимум 4×4 зафиксирован только как продуктовый ориентир.
- Клики по тайлам и интерактив выбора.
- Отдельная dark-палитра тайлов (в dark остаются те же цвета поля).
- Полноценная замена/удаление sync schema `board` / message `move` на сервере сверх переименования room и зачистки шашечной лексики в skills/AGENTS.
- Новые HTTP API и изменения auth flows (кроме упоминания имени комнаты в сценариях).

## Capabilities

### New Capabilities

- `game/board`: статическое игровое поле «Счастливый турист» на экране игры (геометрия, типы тайлов, цвета, адаптив, сплошной центр) без фигурок, кликов и правил.

### Modified Capabilities

- `lobby/rooms`: каноническое имя playable room и lobby filter — `tourist` вместо `checkers`.
- `ui/theme`: требование «тема chrome не меняет доску шашек» заменяется на совместимость с новым полем (тайлы поля не зависят от light/dark preference).
- `auth/login`: сценарий join после Google JWT использует room `tourist` вместо `checkers`.

## Impact

- Client: экран игры, game/lobby wiring имени комнаты, удаление шашечного move UX.
- Server: регистрация room `tourist`, тесты и loadtest на новое имя; без новых правил.
- Meta: ребренд «шашки» → «Счастливый турист» / настольная игра в AGENTS, docs, openspec config context, client/server skills; `work-with-checkers` → `work-with-game`.
- Main specs `lobby/rooms`, `ui/theme`, `auth/login` потребуют sync после archive; появится `game/board`.

## References

- `docs/projects-map.md` — пути client/server.
- `../happy-tourist.github.io/AGENTS.md` — client UI / game.
- `../happy-tourist-server/AGENTS.md` — rooms / listing.
- Explore: поле 10×10 sparse; `1` старт зелёный; `*` задание коричневый; центр жёлтый сплошной 2×2; radius 12; gap 6; max tile 60px; mobile edge-to-edge; фон как страница; room `tourist`; D1 UI-only; D8 rename room на server тоже.
