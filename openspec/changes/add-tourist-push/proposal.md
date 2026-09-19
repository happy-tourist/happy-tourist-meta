## Why

На поле уже есть ход, peek, спасение и возврат с финиша, но нет действия **толкнуть** соседа по прямой — полезный тактический ход своим или чужим туристом. Параллельно возврат с финиша всё ещё открывает лишнюю модалку подтверждения; нужен такой же зелёный affordance над головой, как у спасения на поле.

## What Changes

- Новое действие **push**: выбранный свободный свой турист может толкать соседнего (свой/чужой, не trapped) на клетку «с другой стороны» по прямой (Chebyshev-вектор), если посадка туда — как обычный ход (включая финиш и решётку; дыры нельзя).
- Стоимость **1 шаг**; ход не передаётся; при остатке шагов можно толкать снова (в т.ч. другого соседа).
- UX: иконки push над **целями**; глаз peek — над выбранным; после пуша selection остаётся на толкающем; анимация подхода/отскока толкающего + travel жертвы.
- Legal push учитывается в auto-end-turn (как rescue/return).
- Return-from-finish: убрать модалку; зелёная кнопка над головой finished-туриста в нижней панели; клик по слоту strip сам по себе return не стартует.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `game/move` — push (−1 step), геометрия, trap/finish side-effects посадки, reject trapped target, auto-end;
  - `game/finish` — return affordance без модалки (иконка над strip).
- **Экраны:** Game (board affordances + strip return).
- **Контракт:** room `tourist`; новое message push (имя в design); returnFromFinish без смены wire shape.

## Out of scope

- Толкать пойманного (только rescue).
- Красные кольца на клетку назначения пуша (только иконка).
- Новые типы ловушек / смена layout.
- Публичные чужие budgets.
- Auth / lobby / HTTP / reconnect.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: действие push; стоимость; side-effects посадки; auto-end.
- `game/finish`: UX return — иконка вместо модалки; слот strip не активирует return.

## Impact

- **Server:** validate/apply push; handler; mocha SC; auto-end включает legal push.
- **Client:** board push icons + anim; strip return icon; удаление return-confirm dialog; i18n; keep-focus после push.
- **Docs/skills:** при необходимости по `docs/projects-map.md` и sibling AGENTS.

## References

- Explore (этот чат): D1=1 step; D2=no push trapped; D3=grille like move; D4=return green icon over strip head, no modal, slot not the control; Q1=icons per pushable neighbor of selected; Q2=eye on selected, push on target; Q3=keep selection; auto-end as current steps logic.
- Main specs: `openspec/specs/game/move/spec.md`, `openspec/specs/game/finish/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
