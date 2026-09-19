## Why

На поле уже есть решётки, но нет второй one-shot ловушки с перемещением. **Катапульта** добавляет риск «шагнул — отбросило», пересекается с решётками на одной клетке и использует тот же паттерн плотности при создании комнаты.

Базовый seed / fling / lobby density и черновая presentation-очередь уже в runtime, но **overlay часто не стартует**: sync сразу ставит фишку на dest fling’а, клиент теряет reveal → визуально «просто перекидывает». Нужен follow-up: надёжный enqueue + порядок **доезд на клетку → overlay ~1000 ms → fling** одинаково у игрока и spectator.

## What Changes

- В create — отдельный выбор плотности катапульт: мало / средне / много (те же **12% / 22% / 35%** task-клеток, default средне), независимо от решёток.
- На `enterPlaying` сервер сеет скрытые катапульты только на коричневые task-клетки; пересечение с решётками на одной клетке разрешено.
- Любой step на клетку (ход, push, return с финиша) резолвит оставшиеся ловушки на клетке в случайном порядке; катапульта one-shot: reveal → fling (или broken) → исчезает.
- Fling: случайная свободная landable-клетка на Chebyshev-2, иначе на Chebyshev-1; иначе broken path; после приземления — те же эффекты, что после хода/push (в т.ч. center = финиш).
- После rescue, если на клетке ещё катапульта — тот же resolve.
- **Презентация:** у **всех** клиентов (игрок + spectator) одна цепочка: сначала визуальный доезд на клетку катапульты (как шаг / push / return land), затем overlay (successful ~**1000 ms** appear→vanish; broken: 300+300), затем fling travel / finish travel; цепочка fling→catapult снова с доезда; board lock на всю последовательность. Фикс: не терять reveal при split mirror seats→revealing.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `lobby/rooms` — create option плотности катапульт (мало/средне/много, default средне), отдельно от решёток;
  - `game/board` — overlay/анимация; **land-before-overlay**; sequential fling; broken hold; board lock; spectator = игрок;
  - `game/move` — seed; land/push/return → стек ловушек; fling rings; broken; цепочка после приземления; rescue → оставшаяся катапульта;
  - `game/finish` — fling на center завершает фишку; finish travel **после** vanish катапульты (и после land+overlay).
- **Экраны:** Lobby (create modal — второй селектор плотности); Game (board overlay + анимации + lock).
- **Контракт:** room `tourist`; create options + sync/reveal катапульт. Авторитетный relocate на server без delay; presentation delay — client-only.

## Out of scope

- Новые типы ловушек сверх катапульты.
- Редактор карт / смена геометрии layout.
- Тонкая настройка % плотности в UI (только три пресета).
- Публичный показ чужих budgets.
- Новые HTTP / auth / reconnect-политика.
- Изменение правил решёток (trap/rescue/all-jail/leave-clear), кроме совместного резолва на клетке с катапультой.
- Server-side delay coords/`finished` на время анимации (отклонено: проще client-only).
- Лимит длины цепочки fling→fling.
- Отдельные тайминги appear/vanish сверх суммарных ~1000 ms для successful.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create выбирает плотность катапульт (12/22/35%), независимо от решёток.
- `game/board`: видимость/анимация; land-before-overlay; sequential piece travel; broken timeline; board anim lock; same sequence for spectators.
- `game/move`: seed катапульт; стек ловушек; fling / broken; триггеры move/push/return; цепочка land после fling; rescue → catapult.
- `game/finish`: fling на center = finish piece; presentation after catapult vanish.

## Impact

- **Server:** seed / resolve / fling / create option (секции 1–4 done; presentation follow-up — client).
- **Client:** create option + i18n; board overlay; **fix enqueue + land→overlay→fling** для всех зрителей; board lock; ассеты.
- **Docs/skills:** game-board / AGENTS blurbs под land-before-overlay.

## References

- Explore: seed/fling D*; Anim-D1…D3; **Anim-D4** land затем overlay затем fling; spectator = player; push/return как шаг; successful ~1000 ms; root cause sync `flush:'sync'` + seats before revealing.
- Main specs: `openspec/specs/{lobby/rooms,game/board,game/move,game/finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
