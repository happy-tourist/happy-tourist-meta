## Why

На поле ещё нет ловушек: партия сводится к ходам, peek и финишу. Первая ловушка — **решётка** — добавляет риск захода на task-клетки, кооперативное спасение своих туристов и аварийный выход (reset на старт / возврат с финиша), без новых типов ловушек и без редактора карт.

## What Changes

- При создании комнаты — выбор плотности решёток: мало / средне / много (процент от task-клеток на seed; default средне).
- На `enterPlaying` сервер сеет скрытые решётки только на task-клетки; до захода их никто не видит.
- Заход на клетку с решёткой: анимация опускания (видят все) → турист `trapped` (нельзя ходить и peek).
- Спасение: свой свободный турист на Chebyshev-1 (включая диагональ), при steps ≥ 1 — иконка над trapped; клик = −1 step, анимация подхода/назад, решётка поднимается и исчезает (spent).
- Все 4 своих trapped → сразу расстановка на свободные старты своей стороны (по одному на сторону), державшие решётки пропадают; модалка-предупреждение только себе; ход продолжается при остатке steps/времени.
- Finished-туриста можно вернуть на кольцо вокруг центра (Chebyshev-1 от блока 2×2, с углами): иконка у flag при steps ≥ 1 и свободной клетке без дыры; −1 step; снова в игре.
- Solo = те же правила, что multi.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `lobby/rooms` — create option плотности решёток (мало/средне/много, default средне);
  - `game/board` — overlay/анимация решётки; spent-клетка обычная task (peek после спасения возможен);
  - `game/move` — land→trap; rescue/return (−1 step); all-jail reset; auto-end учитывает rescue/return; lock move+peek у trapped;
  - `game/pieces` — sync `trapped`; расстановка на старты при all-jail; unfinish при return;
  - `game/finish` — return снимает `finished` у piece (пока seat без полного place / на практике при наличии хода и steps).
- **Экраны:** Lobby (create modal); Game (board, strip, модалка all-jail только себе).
- **Контракт:** room `tourist`; create options + sync состояния ловушек/trapped; messages rescue / returnFromFinish (имена в design).

## Out of scope

- Другие типы ловушек / несколько ловушек на одной клетке.
- Редактор своей карты / смена геометрии layout.
- Спасение чужих туристов.
- Покупка/продажа решёток, магазин тайлов.
- Публичный показ чужих budgets.
- Новые HTTP / auth / reconnect-политика.
- Тонкая настройка % плотности в UI (только три пресета).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create выбирает плотность решёток.
- `game/board`: видимость/анимация решёток; task после spent доступен для peek.
- `game/move`: trap / rescue / return / all-jail; стоимость в steps; auto-end.
- `game/pieces`: `trapped`; reset на старты; return → again on board.
- `game/finish`: piece может снова стать unfinished через return.

## Impact

- **Server:** seed решёток (% task); schema/private maps; handlers move side-effect + rescue/return; mocha на SC.
- **Client:** create option; board overlay + анимации; strip return; all-jail warning modal; i18n.
- **Ассет:** картинка решётки в client assets (путь в design).
- **Docs/skills:** по `docs/projects-map.md` и sibling AGENTS при необходимости.

## References

- Explore (этот чат): D1 task-only; D2 own rescue 1 step; D3 all-4 own → start per side; D4 holding grilles spent; D5 return = in play; D6 ring w/ corners; D7 grille goes, tile peekable; density 25/45/65 default medium; Q8 same turn; Q9 moot; Q12 auto-reset + warning modal self-only; return always when steps+legal cell; trapped locks peek; rescue Chebyshev-1 diagonal OK.
- Main specs: `openspec/specs/{lobby/rooms,game/board,game/move,game/pieces,game/finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
