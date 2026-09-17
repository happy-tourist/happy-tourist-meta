## Why

На экране Game chrome размазан: выход и статус в локальной шапке страницы, усечённый room id мешает, presence сверху/снизу вокруг доски плюс отдельная полоса туристов. Нужен спокойный HUD: общая шапка приложения и нижняя панель всегда на виду — в том числе когда позже появятся карты другого размера.

## What Changes

- Убрать отображение идентификатора комнаты с экрана Game (param/reconnect без изменений).
- Перенести icon-only выход и текстовый статус партии в общую шапку приложения (выход слева, статус по центру, переключатель темы справа); на Login/Lobby выход не показывать.
- Собрать presence, собственные ресурсы/end-turn и личную полосу туристов в одну нижнюю панель, прибитую к низу viewport; доска — в пространстве над панелью.
- Для сидящего: свой маркер слева → ресурсы → слоты туристов → соперники у правого края; для зрителя: все occupied-маркеры по центру панели.
- Облака say у всех маркеров — сверху (к доске); подпись «Мои туристы» убрать.
- Affordance ready / say / budgets / end-turn — как сейчас по смыслу, меняется только место в layout.

## Scope

- Пакет: **client** only (server / room protocol / schema без изменений).
- Capability ID:
  - `game/presence` — нижняя HUD-панель, раскладка seated/spectator, статус в общей шапке
  - `game/say` — направление облаков при единой нижней панели
  - `game/leave` — размещение exit control в общей шапке на Game
  - `game/pieces` — личная полоса внутри нижней панели без текстовой подписи
- Экраны: Game (основной UX); общая шапка на Login/Lobby/Game (theme без регрессии; leave только на Game).

## Out of scope

- Отдельная адаптация узкой мобильной ширины (overflow / сжатие) — отложено; на широком layout панель собираем как задумано.
- Смена размеров/набора карт доски, серверный контракт, новые say-пресеты.
- Переименование room / публичное имя комнаты (продукт по-прежнему без display-name).
- Изменение правил leave-confirm, budgets, turn rings, reconnect grace.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/presence`: единая нижняя sticky-панель вместо top/bottom рядов вокруг доски; seated/spectator раскладка; статус партии в общей шапке; без room id в UI
- `game/say`: облака всегда сверху у маркера (к доске); убрать ветвление top-row → ниже
- `game/leave`: primary exit control в общей шапке приложения слева на экране Game
- `game/pieces`: личная полоса в нижней панели рядом с собственным chrome, без подписи «Мои туристы»

## Impact

- Client UI: общая шапка + Game screen layout / presence / strip.
- Skills client (`work-with-game-board`, `work-with-pages`, при необходимости `work-with-styles` / say) — обновить после apply.
- Server, Colyseus messages, synced state — без изменений.
- Маршрут с `roomId` и reconnect token — без изменений (только UI не показывает id).

## References

- Explore: переработка Game HUD (нижняя панель + шапка).
- Sibling: `../happy-tourist.github.io/AGENTS.md`; meta skills `.agents/skills/client/work-with-game-board`, `work-with-pages`.
- Main specs: `openspec/specs/game/presence`, `game/say`, `game/leave`, `game/pieces`, `ui/theme`.
