## Why

На поле уже есть решётки, но нет второй one-shot ловушки с перемещением. **Катапульта** добавляет риск «шагнул — отбросило», пересекается с решётками на одной клетке и использует тот же паттерн плотности при создании комнаты.

Базовый seed / fling / lobby density уже реализованы. Follow-up: презентация должна быть **последовательной** — сначала полная анимация катапульты, затем travel фишки (как ход / finish), иначе турист «пропадает» во время fade.

## What Changes

- В create — отдельный выбор плотности катапульт: мало / средне / много (те же **12% / 22% / 35%** task-клеток, default средне), независимо от решёток.
- На `enterPlaying` сервер сеет скрытые катапульты только на коричневые task-клетки; пересечение с решётками на одной клетке разрешено.
- Любой step на клетку (ход, push, return с финиша) резолвит оставшиеся ловушки на клетке в случайном порядке; катапульта one-shot: reveal → fling (или broken) → исчезает.
- Fling: случайная свободная landable-клетка на Chebyshev-2, иначе на Chebyshev-1; иначе broken path; после приземления — те же эффекты, что после хода/push (в т.ч. center = финиш).
- После rescue, если на клетке ещё катапульта — тот же resolve.
- **Презентация (follow-up):** client-only — пока идёт overlay катапульты, фишка визуально на клетке выстрела; **после** полного исчезновения overlay — travel на dest / finish travel; цепочка fling→catapult снова последовательно; broken: appear → 300 ms целая → break → 300 ms broken → vanish, фишка остаётся; во время любой board-анимации клики по доске запрещены.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `lobby/rooms` — create option плотности катапульт (мало/средне/много, default средне), отдельно от решёток;
  - `game/board` — overlay/анимация катапульты; sequential fling travel; broken hold; board input lock на анимациях;
  - `game/move` — seed; land/push/return → стек ловушек; fling rings; broken; цепочка после приземления; rescue → оставшаяся катапульта;
  - `game/finish` — fling на center завершает фишку; finish travel **после** vanish катапульты.
- **Экраны:** Lobby (create modal — второй селектор плотности); Game (board overlay + анимации + lock).
- **Контракт:** room `tourist`; create options + sync/reveal катапульт; согласованные side-effects с существующими move/push/return/rescue. Авторитетный relocate на server без delay; presentation delay — client-only.

## Out of scope

- Новые типы ловушек сверх катапульты.
- Редактор карт / смена геометрии layout.
- Тонкая настройка % плотности в UI (только три пресета).
- Публичный показ чужих budgets.
- Новые HTTP / auth / reconnect-политика.
- Изменение правил решёток (trap/rescue/all-jail/leave-clear), кроме совместного резолва на клетке с катапультой.
- Server-side delay coords/`finished` на время анимации (отклонено: проще client-only).
- Лимит длины цепочки fling→fling.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create выбирает плотность катапульт (12/22/35%), независимо от решёток.
- `game/board`: видимость/анимация катапульт; sequential piece travel; broken timeline; board anim lock.
- `game/move`: seed катапульт; стек ловушек; fling / broken; триггеры move/push/return; цепочка land после fling; rescue → catapult.
- `game/finish`: fling на center = finish piece; presentation after catapult vanish.

## Impact

- **Server:** seed катапульт; резолв стека на land; fling; create option; mocha на SC (уже в основном done).
- **Client:** create option + i18n; board overlay + **sequential** piece travel / broken holds; board input lock на move/grille/catapult/finish/fling anim; ассеты целой и сломанной катапульты.
- **Docs/skills:** по `docs/projects-map.md` и sibling AGENTS при необходимости.

## References

- Explore (этот чат): плотность/seed/fling D*; follow-up anim: D1 vanish→travel; D2 sequential chain; D3 client-only; broken 300+300; board lock all anims; no chain cap.
- Main specs: `openspec/specs/{lobby/rooms,game/board,game/move,game/finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
