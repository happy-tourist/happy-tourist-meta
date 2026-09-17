## Why

На экране Game chrome размазан: выход и статус в локальной шапке страницы, усечённый room id мешает, presence сверху/снизу вокруг доски плюс отдельная полоса туристов. Нужен спокойный HUD: общая шапка приложения и нижняя панель всегда на виду. После сборки нижней панели четыре полноразмерных слота strip (~4×72px) всё ещё ломают раскладку на desktop и mobile; статусы (финиш / решётка) и выбор туриста нужно уместить в один квадрат размера аватара с picker’ом.

## What Changes

- Убрать отображение идентификатора комнаты с экрана Game (param/reconnect без изменений).
- Перенести icon-only выход и текстовый статус партии в общую шапку приложения (выход слева, статус по центру, переключатель темы справа); на Login/Lobby выход не показывать.
- Собрать presence, собственные ресурсы/end-turn и личную полосу туристов в одну нижнюю панель, прибитую к низу viewport; доска — в пространстве над панелью.
- Для сидящего: свой маркер слева → ресурсы → слоты туристов → соперники у правого края; для зрителя: все occupied-маркеры по центру панели.
- Облака say у всех маркеров — сверху (к доске); подпись «Мои туристы» убрать.
- Affordance ready / say / budgets / end-turn — как сейчас по смыслу, меняется только место в layout.
- Сжать личный strip в HUD до одного compact chip (~размер аватара) с сеткой 2×2 (N/E / W/S): статусы финиша и решётки видны на мини-слотах; клик по chip **только** открывает picker (не select).
- Picker: `q-menu` вверх к доске (desktop и mobile), без заголовка; внутри — один ряд из четырёх полноразмерных слотов (как прежний strip); клик по доступному туристу = select и закрытие меню; клик вне / Esc = закрыть без select.
- Return-from-finish control — **только** в меню рядом с finished-слотом; на compact chip — только индикатор финиша (без return).
- На chip и в меню показывать решётку, когда piece `trapped` (зеркало поля); анимация drop/rise решётки **~1000 ms** везде (поле + chrome strip).
- Клик по туристу на доске по-прежнему select без меню. На чужом ходе / без move-interact — меню только просмотр статусов (без select).

## Scope

- Пакет: **client** only (server / room protocol / schema без изменений).
- Capability ID:
  - `game/presence` — нижняя HUD-панель, раскладка seated/spectator, статус в общей шапке
  - `game/say` — направление облаков при единой нижней панели
  - `game/leave` — размещение exit control в общей шапке на Game
  - `game/pieces` — compact chip + menu picker вместо ряда из четырёх 72px в HUD; статусы trapped/finished на chrome
  - `game/finish` — return affordance только в picker-меню; finish indicator остаётся на chip и в меню
  - `game/board` — длительность анимации revealed grille drop/rise ~1000 ms (вместо ~1500 ms)
- Экраны: Game (основной UX); общая шапка на Login/Lobby/Game (theme без регрессии; leave только на Game).

## Out of scope

- Отдельная адаптация узкой мобильной ширины для всего HUD (горизонтальный scroll панели допустим) — кроме сжатия strip в chip.
- Смена размеров/набора карт доски, серверный контракт, новые say-пресеты.
- Переименование room / публичное имя комнаты (продукт по-прежнему без display-name).
- Изменение правил leave-confirm, budgets, turn rings, reconnect grace, trap/rescue/return **логики** на server.
- Idle-loop анимации решётки (только one-shot drop/rise ~1000 ms).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/presence`: единая нижняя sticky-панель вместо top/bottom рядов вокруг доски; seated/spectator раскладка; статус партии в общей шапке; без room id в UI
- `game/say`: облака всегда сверху у маркера (к доске); убрать ветвление top-row → ниже
- `game/leave`: primary exit control в общей шапке приложения слева на экране Game
- `game/pieces`: личная полоса в нижней панели как compact 2×2 chip + `q-menu` ряд×4; без подписи «Мои туристы»; grille/finish статусы на chrome; select из меню или с доски
- `game/finish`: return control только в picker-меню; finish indicator на chip и в меню
- `game/board`: grille drop/rise animation duration ~1000 ms on board (and same timing for strip chrome overlays)

## Impact

- Client UI: общая шапка + Game screen layout / presence / compact strip + menu / grille anim timing.
- Skills client (`work-with-game-board`, `work-with-pages`, при необходимости `work-with-styles` / finish notes) — обновить после apply.
- Server, Colyseus messages, synced state — без изменений (`piece.trapped` / `holdingGrilleKeys` уже есть).
- Маршрут с `roomId` и reconnect token — без изменений (только UI не показывает id).

## References

- Explore: переработка Game HUD + compact tourist strip / grille на статусах.
- Sibling: `../happy-tourist.github.io/AGENTS.md`; meta skills `.agents/skills/client/work-with-game-board`, `work-with-pages`.
- Main specs: `openspec/specs/game/presence`, `game/say`, `game/leave`, `game/pieces`, `game/finish`, `game/board`, `ui/theme`.
