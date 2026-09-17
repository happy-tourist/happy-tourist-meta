## Why

Фаза общей шапки и sticky HUD уже в коде, но нижняя панель с chip/`q-menu` и всеми маркерами снизу не даёт удобный narrow UX и ломает привычную топологию «чужие над доской». Нужен второй проход: оппоненты сверху, свои четыре туриста снова в панели без меню, узкая ширина ≤320 (лучше 300), плюс понятный return с финиша и клик по всему финиш-блоку.

## What Changes

- **Уже сделано (фаза 1, сохранить):** leave + match status в общей шапке на Game; без room id; sticky bottom shell; grille anim ~1000 ms на поле/chrome.
- **Перестроить presence:** seated — оппоненты **над** доской, свой маркер в нижней панели; spectator — **все** occupied markers над доской; внизу у seated — свой кластер + личные туристы (без чужих markers).
- **Budgets / end-turn:** steps и peeks в **ряд над** своим аватаром; кнопка завершить ход — **над** нижней панелью справа (у нижнего правого края области доски).
- **Личные туристы:** убрать compact chip и `q-menu`; снова слоты выбора в HUD — на широкой ширине **ряд×4**, на узкой **сетка 2×2** со слотами **меньше** аватара; без подписи «Мои туристы»; влезать в **≤320** (желательно **300**) без горизонтального scroll как основного UX.
- **Finished / return:** не затемнять finished, если return сейчас доступен; затемнять, если return недоступен (некуда/нельзя). Клик по доступному finished → модалка «Вернуть на поле?»; Да → подсветка кольца **тем же цветом, что legal move targets**; Нет/Esc → ничего. Отдельной undo-кнопки на слоте нет. Клик по unfinished — select как сейчас. Переключение на другого туриста сбрасывает return-mode. Шаг списывается только при успешном return на сервер.
- **Return animation:** при успешном return все клиенты анимируют выезд с финиша на клетку кольца с **ближайшей** из четырёх center-клеток (Chebyshev; tie-break row, затем col).
- **Клик по финишу:** любой клик по визуальному 2×2-блоку финиша (не квадрант) отправляет ход на **ближайшую легальную** из четырёх center-клеток относительно текущей клетки туриста.
- Обновить client skills под новый layout / return / center-click.

## Scope

- Пакет: **client** only (server / room protocol / schema без изменений; `returnFromFinish` и finish rules уже есть).
- Capability ID:
  - `game/presence` — top opponents / spectator top; bottom own + strip; budgets над аватаром; end-turn над панелью справа
  - `game/say` — bubbles к доске: сверху markers вниз, снизу вверх
  - `game/leave` — без изменений относительно фазы 1 (уже в шапке)
  - `game/pieces` — strip ряд/2×2 без chip/menu; grille на слотах; narrow fit
  - `game/finish` — modal return; conditional dim; finish indicator на слотах
  - `game/board` — grille ~1000 ms (сохранить)
  - `game/move` — center click → nearest legal center cell; return travel anim from nearest center; return target chrome = move target color
- Экраны: Game; шапка Login/Lobby/Game без регрессии leave/status.

## Out of scope

- Server rules / messages / schema changes.
- Idle-loop grille; смена trap/rescue логики.
- Отдельный display-name комнаты.
- Изменение правил leave-confirm, budgets grants, reconnect grace.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/presence`: opponents (и spectator all) над доской; bottom = own + strip; budgets над аватаром; end-turn над панелью справа; sticky низ и header status/leave без отката
- `game/say`: ориентация bubbles к доске по вертикали маркера
- `game/leave`: (фаза 1) exit в общей шапке — без новой дельты, если уже соответствует
- `game/pieces`: полный strip без chip/menu; responsive ряд↔2×2; narrow ≤320/300
- `game/finish`: modal return; dim только когда return недоступен
- `game/board`: timing grille ~1000 ms (сохранить)
- `game/move`: nearest-center finish click; return anim; return target color = move target color

## Impact

- Client UI: `GamePage.vue` / presence layout / strip / finish modal / center click / return anim / i18n.
- Skills client (`work-with-game-board`, `work-with-pages`, styles/finish notes) — обновить после apply.
- Server — без изменений.

## References

- Explore (сессия): opponents top, strip без menu, narrow, return modal, nearest center, return anim.
- Предыдущая фаза того же change: header + sticky HUD + chip (chip отменяется этим обновлением плана).
- Sibling: `../happy-tourist.github.io/AGENTS.md`; main specs `openspec/specs/game/*`.
