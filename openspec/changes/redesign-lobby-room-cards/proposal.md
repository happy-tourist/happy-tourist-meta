## Why

Список созданных игр в лобби всё ещё выглядит как плоский `q-item` ряд и расходится с уже согласованными карточками паков, наборов заданий и карт (shared chrome, outline-кнопки, структура метрик). Нужен макет-aligned редизайн карточки комнаты: превью карты, места, туристы без «+», название пака, выбранные наборы с количеством заданий, статус фазы поверх карты — с приоритетом консистентности кнопок/статусов/отступов/цветов над пикселями Gemini-макета.

## What Changes

- Карточки комнат в live lobby listing: вертикальная card-сетка (~ширина 180), не list-row.
- Структура: превью карты → места `occupied/maxSeats` → туристы (без плюса) → название пака → строки выбранных наборов с `taskCount` (≤4 + «ещё {k}») → outline «Войти».
- Статус фазы (`waiting` / `playing`) — short badge поверх превью по центру; размеры/цвета как текущие soft pills.
- Join по всей карточке и по кнопке; busy-lock join без изменения семантики.
- Lobby metadata: у каждого выбранного набора — `taskCount` (D1, из snapshot при create).
- Visual Spec из prepare-mock; **consistency > mock** (`--pack-card-*`, action-h 30, dark `#2a2a2a`).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: chrome/list presentation карточки комнаты; расширение listing metadata `taskCount`; ревизия SC-LOBBY-25/26 (+ новые сценарии card chrome / overflow / status overlay).

## Scope

- **Capability ID:** `lobby/rooms`
- **Пакеты:** client + server (metadata listing)
- Поверхность: live lobby room listing only
- Контракт: room `tourist` listing metadata (расширение полей набора); create/join/subscribe flows без смены семантики
- Mock / Visual Spec в change `assets/`
- Иконка мест: reserved SVG (пользователь добавит); Material placeholder до drop-in

## Out of scope

- Maps list / pack / task-set body redesign (уже сделаны; не трогать кроме shared tokens reuse)
- Create-modal chrome (кроме косвенного чтения тех же i18n ключей ordinals)
- Смена ACL join/spectator, maxSeats rules, densities, LobbyRoom subscribe protocol
- Удаление author из snapshot labels на сервере (карточка по-прежнему не показывает автора)
- Hover scale / enlarge
- Замена pack/task-set SVG ассетов
- Перерисовка map preview tiles

## Impact

- Client: lobby listing UI → card tile; i18n seats/tourists/status/overflow; join wiring; vitest SC-LOBBY-25/26 + новые chrome scenarios
- Server: `setMetadata` / snapshot labels + `taskCount` per selected set; mocha listing metadata
- Delta: `lobby/rooms`
- Skills: `work-with-lobby`, pack-cards note (lobby consumes `--pack-card-*`), возможно pages/stores topics
- References: prepare-mock extract 2026-10-03; explore redesign lobby room cards; prior pack/task-set/map card redesigns

## References

- Prepare-mock: lobby card light+dark; consistency > mock pixels; width 180; status center; seats + tourists + pack + sets with counts; «Войти»; overflow «ещё {k}»
- Explore 2026-10-03: D1 expand metadata taskCount; lobby-only; Material seat placeholder until user SVG
- Main `openspec/specs/lobby/rooms`
- Archive pack/task-set/map list card redesigns (chrome canon `--pack-card-*`)
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
