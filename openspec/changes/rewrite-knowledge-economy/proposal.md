## Why

Текущая экономика (шаги/просмотры, peek по иконке, соло-остаток, место 1/2/3) размывает цель партии. Нужна модель «один ход → открытие клетки → очки знаний на туристе → донести до финиша», с фиксированными ловушками, обрезанием последнего игрока и понятным UX (таймер хода, toast исхода ответа, разные тексты финиша/cut).

## What Changes

- **BREAKING:** убрать budgets steps/peeks, peek-аффорданс, end-turn, push, выбор density в create, соло-остаток (∞ / 1 мин / time-expired по соло), модалку места 1/2/3 как итог.
- Один ход по карте за очередь (move **или** return); после приземления клетка открывается (задание / решётка / катапульта); задание — модалка до закрытия (или timeout), затем очередь дальше.
- **Таймер хода 60 с** + presence turn ring; при timeout открытой Q&A-модалки — закрытие как «Не успел!» + toast сверху по центру (как раньше по смыслу).
- Очки знаний = сложность задания (1|2|3) на туристе; при финише туриста — на аватар (+N анимация); недонесённые сгорают при обрезании последнего.
- Carried knowledge UI: **только** нижний strip, badge **сверху справа** + иконка `mdi-lightbulb-on`; с поля убрать. Delivered — у аватара.
- Спасение своего рядом — бесплатно и без лимита **до** хода по карте; толкания нет.
- Сид: решётки **15%**, катапульты **15%** от task-клеток, **взаимоисключающе** с заданием; после ловушки/угаданного — пустая клетка.
- Решётка: счётчик 3…1 (тик при приходе своего хода); badge поверх решётки (z-index); при приходе с 1 — снять и сразу ходить.
- **All-jail:** не телепортировать на старт; оставить за решёткой до истечения `remainingTurns`; без модалки сброса на старт; ход advance после ловушки как обычно.
- Correct / wrong / timeout → **toast** сверху по центру (Quasar Notify), не тихий flash в header status.
- Предпоследний довёл всех → cut последнего; модалки знаний с **разным** текстом: полный финиш vs обрезание. `maxSeats === 1` — без cut.
- Починить анимацию ухода на финиш (клик по центру и fling катапульты → центр).

## Scope

- **Capability ID:** `lobby/rooms`, `game/board`, `game/move`, `game/presence`, `game/finish` (по касанию `game/pieces`, `game/start` — только если требования меняются из-за экономики/таймера)
- **Server:** room `tourist` — сид 15%/15%, очередь с deadline 60s, open-on-land / fling, knowledge + outcome finish|cut, grille timer без all-jail teleport, penultimate cut, убрать push/budgets/end-turn.
- **Client:** Lobby без density; Game — knowledge HUD (strip TR + lightbulb), grille badge z-index, turn ring 60s, focus@3, Notify toast исходов, две формулировки knowledge-модалки, фикс finish travel; без peek/push/place-badge / solo leftover.

## Out of scope

- Контент packs/maps (редактор, модерация), auth, leave confirm, say whitelist.
- Новый рейтинг/лидерборд между игроками (только показ своих донесённых знаний).
- Смена геометрии карт / `touristsPerPlayer` / `maxSeats` выбора в lobby.
- Визуальный редизайн тайлов сверх необходимых индикаторов (knowledge / grille counter / toast / timer ring).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: убрать выбор grille/catapult density; create без density options; сид фиксирован на сервере (15%/15%).
- `game/board`: exclusive клетки; open-on-land; empty после clear; grille badge TR поверх overlay; carried knowledge не на поле.
- `game/move`: один ход (move|return); **таймер 60s**; free rescue; open-on-land; grille timer 3; all-jail **без** teleport; без push/steps/peeks/end-turn/solo ∞ leftover.
- `game/presence`: knowledge у аватара; carried только strip TR + lightbulb; focus@3; **вернуть turn ring**; toast correct/wrong/timeout; без end-turn / place-badge / solo leftover UX.
- `game/finish`: transfer knowledge; модалки с разным copy finish vs cut; penultimate cut; анимация финиша (клик/fling).

## Impact

- Контракт room `tourist`: private `knowledge` `{ delivered, pieces, outcome? }` (own only); `turnUntil` / `turnBudgetSeconds=60` снова активны в multi eligible; open-on-land + timeout → `peekClose.reason === 'timeout'`; `push`/`endTurn`/client `peek` retired; create без density; public `Piece.remainingTurns`; exclusive traps 15%/15%; all-jail без start reset; `finishPlace=1` без place UX; две i18n-формулировки по `outcome`.
- Client: включить Quasar `Notify`; header flash исхода ответа может уступить toast.
- Client + server тесты: дописать polish поверх уже применённых блоков 1–6.

## References

- Explore-решения D1–D17 + polish (timer 60s / toast / strip knowledge / all-jail wait / grille 15% / finish anim / dual modal copy), чат 2026-11-21.
- Main specs: `openspec/specs/lobby/rooms`, `game/board`, `game/move`, `game/presence`, `game/finish`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
