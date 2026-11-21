## Why

Текущая экономика (шаги/просмотры, peek по иконке, соло-остаток, место 1/2/3) размывает цель партии. Нужна модель «один ход → открытие клетки → очки знаний на туристе → донести до финиша», с фиксированными ловушками, обрезанием последнего игрока и понятным UX (таймер хода, toast исхода ответа, разные тексты финиша/cut). После первого apply playtest показал дыры: невидимая lightbulb-иконка, пропадание туриста на финише без slide, all-jail skip без комнатного toast, copy cut-модалки.

## What Changes

- **BREAKING:** убрать budgets steps/peeks, peek-аффорданс, end-turn, push, выбор density в create, соло-остаток (∞ / 1 мин / time-expired по соло), модалку места 1/2/3 как итог.
- Один ход по карте за очередь (move **или** return); после приземления клетка открывается (задание / решётка / катапульта); задание — модалка до закрытия (или timeout), затем очередь дальше.
- **Таймер хода 60 с** + presence turn ring; при timeout открытой Q&A-модалки — закрытие как «Не успел!» + toast сверху по центру (как раньше по смыслу).
- Очки знаний = сложность задания (1|2|3) на туристе; при финише туриста — на аватар (+N анимация); недонесённые сгорают при обрезании последнего.
- Carried knowledge UI: **только** нижний strip, badge **сверху справа** + видимая иконка лампочки (Material `lightbulb`, уже в extras — не `mdi-*` без font); с поля убрать. Delivered — у аватара с той же иконкой.
- Спасение своего рядом — бесплатно и без лимита **до** хода по карте; толкания нет.
- Сид: решётки **15%**, катапульты **20%** от task-клеток, **взаимоисключающе** с заданием; после ловушки/угаданного — пустая клетка.
- Решётка: счётчик 3…1 (тик при приходе своего хода); badge поверх решётки (z-index); при приходе с 1 — снять и сразу ходить.
- **All-jail:** не телепортировать на старт; оставить за решёткой до истечения `remainingTurns`; без модалки сброса на старт. **Каждый раз**, когда у сиденья ходить некем (все незавершённые в решётке) и остаётся ≥1 другой eligible — **пропуск хода** + **toast всем** («Все туристы в решётке — ход пропущен»). Если eligible не осталось (партия окончена / cut / все финишировали) — **только** knowledge-модалка, **без** all-jail toast.
- Correct / wrong / timeout → **toast** сверху по центру (Quasar Notify), не тихий flash в header status.
- Предпоследний довёл всех → cut последнего; модалки знаний с **разным** текстом: полный финиш vs обрезание (`которые успели`, не «что успели»). `maxSeats === 1` — без cut.
- Починить анимацию ухода на финиш (клик по центру и fling катапульты → центр) — живой slide+fade, не snap-vanish.

## Scope

- **Capability ID:** `lobby/rooms`, `game/board`, `game/move`, `game/presence`, `game/finish` (по касанию `game/pieces`, `game/start` — только если требования меняются из-за экономики/таймера)
- **Server:** room `tourist` — сид 15%/20% (grille/catapult), очередь с deadline 60s, open-on-land / fling, knowledge + outcome finish|cut, grille timer без all-jail teleport, all-jail skip на become-current + room toast signal, penultimate cut, убрать push/budgets/end-turn.
- **Client:** Lobby без density; Game — knowledge HUD (strip TR + visible lightbulb), grille badge z-index, turn ring 60s, focus@3, Notify toast исходов + all-jail skip, две формулировки knowledge-модалки (cut copy), фикс finish travel; без peek/push/place-badge / solo leftover.

## Out of scope

- Контент packs/maps (редактор, модерация), auth, leave confirm, say whitelist.
- Новый рейтинг/лидерборд между игроками (только показ своих донесённых знаний).
- Смена геометрии карт / `touristsPerPlayer` / `maxSeats` выбора в lobby.
- Визуальный редизайн тайлов сверх необходимых индикаторов (knowledge / grille counter / toast / timer ring).
- Подключение пакета `mdi-v7` (используем уже загруженные material-icons).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: убрать выбор grille/catapult density; create без density options; сид фиксирован на сервере (15% grille / 20% catapult).
- `game/board`: exclusive клетки; сид 15%/20%; open-on-land; empty после clear; grille badge TR поверх overlay; carried knowledge не на поле.
- `game/move`: один ход (move|return); **таймер 60s**; free rescue; open-on-land; grille timer 3; all-jail **без** teleport + **skip каждый раз когда ходить некем** (если есть другой eligible); без push/steps/peeks/end-turn/solo ∞ leftover.
- `game/presence`: knowledge у аватара с **видимой** lightbulb; carried только strip TR + lightbulb; focus@3; **вернуть turn ring**; toast correct/wrong/timeout + **all-jail skip всем**; без end-turn / place-badge / solo leftover UX.
- `game/finish`: transfer knowledge; модалки с разным copy finish vs cut («которые успели»); penultimate cut; анимация финиша (клик/fling) без snap-vanish.

## Impact

- Контракт room `tourist`: private `knowledge` `{ delivered, pieces, outcome? }` (own only); `turnUntil` / `turnBudgetSeconds=60` снова активны в multi eligible; open-on-land + timeout → `peekClose.reason === 'timeout'`; `push`/`endTurn`/client `peek` retired; create без density; public `Piece.remainingTurns`; exclusive traps 15% grille / 20% catapult; all-jail без start reset + room broadcast skip toast (не при end-of-game); `finishPlace=1` без place UX; две i18n-формулировки по `outcome`.
- Client: Quasar `Notify`; icon names from **material-icons** (`lightbulb`); header flash исхода ответа может уступить toast.
- Client + server тесты: блоки 1–8 уже applied; блок 9 — playtest follow-up.

## References

- Explore-решения D1–D17 + polish + playtest follow-up (all-jail skip toast, material lightbulb, finish vanish, cut copy), чат 2026-11-21.
- Main specs: `openspec/specs/lobby/rooms`, `game/board`, `game/move`, `game/presence`, `game/finish`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
