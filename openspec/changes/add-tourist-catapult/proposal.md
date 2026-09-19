## Why

На поле уже есть решётки, но нет второй one-shot ловушки с перемещением. **Катапульта** добавляет риск «шагнул — отбросило», пересекается с решётками на одной клетке и использует тот же паттерн плотности при создании комнаты.

## What Changes

- В create — отдельный выбор плотности катапульт: мало / средне / много (те же **12% / 22% / 35%** task-клеток, default средне), независимо от решёток.
- На `enterPlaying` сервер сеет скрытые катапульты только на коричневые task-клетки; пересечение с решётками на одной клетке разрешено.
- Любой step на клетку (ход, push, return с финиша) резолвит оставшиеся ловушки на клетке в случайном порядке; катапульта one-shot: reveal → fling (или broken) → исчезает.
- Fling: случайная свободная landable-клетка на Chebyshev-2, иначе на Chebyshev-1; иначе broken-спрайт на fade, турист остаётся; после приземления — те же эффекты, что после хода/push (в т.ч. center = финиш).
- После rescue, если на клетке ещё катапульта — сразу fling освобождённого.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `lobby/rooms` — create option плотности катапульт (мало/средне/много, default средне), отдельно от решёток;
  - `game/board` — overlay/анимация катапульты (fade in → fade out; broken при отсутствии целей);
  - `game/move` — seed; land/push/return → стек ловушек; fling rings; broken; цепочка после приземления; rescue → оставшаяся катапульта;
  - `game/finish` — fling на center завершает фишку как обычный land на center.
- **Экраны:** Lobby (create modal — второй селектор плотности); Game (board overlay + анимации).
- **Контракт:** room `tourist`; create options + sync/reveal катапульт; согласованные side-effects с существующими move/push/return/rescue.

## Out of scope

- Новые типы ловушек сверх катапульты.
- Редактор карт / смена геометрии layout.
- Тонкая настройка % плотности в UI (только три пресета).
- Публичный показ чужих budgets.
- Новые HTTP / auth / reconnect-политика.
- Изменение правил решёток (trap/rescue/all-jail/leave-clear), кроме совместного резолва на клетке с катапультой.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create выбирает плотность катапульт (12/22/35%), независимо от решёток.
- `game/board`: видимость/анимация катапульт (fade in/out; broken).
- `game/move`: seed катапульт; стек ловушек; fling / broken; триггеры move/push/return; цепочка land после fling; rescue → catapult.
- `game/finish`: fling на center = finish piece (как land на center).

## Impact

- **Server:** seed катапульт; резолв стека на land; fling; create option; mocha на SC.
- **Client:** create option + i18n; board overlay + анимации; ассеты целой и сломанной катапульты.
- **Docs/skills:** по `docs/projects-map.md` и sibling AGENTS при необходимости.

## References

- Explore (этот чат): D1 пересечение + random order + count как решётки; D2 только landable свободные; D3 после fling = как ход/push; триггеры move/push/return; center = finish; D5b те же 12/22/35 default medium отдельный селектор; D6 one-shot; D7 только task `*`; D8 broken + stay + consume; цель fling в момент выстрела; порядок ловушек рандом каждый резолв.
- Main specs: `openspec/specs/{lobby/rooms,game/board,game/move,game/finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
