## Why

На поле уже есть решётки и катапульты (seed / fling / lobby density / client land→overlay→fling). Playtest показал два дефекта презентации vs authority:

1. **Ход уходит раньше анимации** — после последнего шага на катапульту `maybeAutoEndTurn` / deadline сменяют `currentTurn`, а fling/overlay догоняют позже (катапульта «не видна», турист откидывается уже на чужом ходе).
2. **Цепочка «заранее»** — сервер в одном тике резолвит всю цепочку catapult→…→grille; sync сразу на финале + `holdingGrilleKeys`; решётка падает «в другом месте без кота», пока клиент ещё рисует hop’ы.

Нужен follow-up: **server-paced** только следующий hop (fling-land = новый land-sense, без −1 step), единый timeline, **переход хода только после** конца презентации; без client ack.

## What Changes

- В create — отдельный выбор плотности катапульт: мало / средне / много (те же **12% / 22% / 35%**, default средне) — уже сделано.
- Seed / fling geometry / broken / overlap с решётками — уже сделано; экономика steps **без изменений** (fling не −1).
- **Paced trap resolve:** после land (move / push / return / post-rescue / post-fling) сервер резолвит **только текущую** клетку: reveal → ждать presentation budget (общие ms с client) → применить один эффект → если fling — новая клетка как новый land; **не** считать всю цепочку в одном тике.
- **Deferred turn:** `maybeAutoEndTurn` и сработавший turn-deadline **не** `advanceTurn`, пока идёт trap-presentation pipeline; если переход уже «нужен» — выполнить **после** idle. Без `presentationDone` с клиента.
- **Презентация:** один последовательный timeline (land → catapult overlay → fling travel → … → grille drop); board lock; spectator = игрок. Client следует hop-sync, а не реконструирует финал из одного patch.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `lobby/rooms` — create option плотности (done);
  - `game/board` — overlay; land-before-overlay; sequential hops; grille после prior hops; board lock;
  - `game/move` — seed/fling (done) + **paced resolve** + **deferred auto-end / deadline advance**;
  - `game/finish` — fling на center; finish travel после vanish текущего hop.
- **Экраны:** Lobby (done); Game (board + turn chrome после анимаций).
- **Контракт:** room `tourist`; sync reveal/holding по hop; **нет** нового client→server presentation ack.

## Out of scope

- Новые типы ловушек сверх катапульты.
- Редактор карт / смена геометрии layout.
- Тонкая настройка % плотности в UI (только три пресета).
- Публичный показ чужих budgets.
- Новые HTTP / auth / reconnect-политика.
- Изменение правил решёток (trap/rescue/all-jail/leave-clear), кроме совместного резолва и paced порядка с катапультой.
- Client `presentationDone` / доверие клиенту для отпуска хода.
- Лимит длины цепочки fling→fling.
- Смена экономики steps/peeks (fling по-прежнему без лишнего step).
- Pause/extend `turnUntil` wall-clock (только отложенный `advanceTurn` после idle).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `lobby/rooms`: create выбирает плотность катапульт (12/22/35%), независимо от решёток.
- `game/board`: видимость/анимация; sequential hop timeline; grille не раньше своего land; board anim lock.
- `game/move`: seed/fling; **paced** land resolve; deferred turn advance after presentation idle.
- `game/finish`: fling на center = finish; presentation after catapult vanish того hop.

## Impact

- **Server:** заменить мгновенный full-chain `resolveCellTraps` на paced pipeline + deferred `advanceTurn` / deadline; mocha на порядок и turn-after.
- **Client:** упростить/выровнять queue под hop-sync; не стартовать grille «на финале» раньше времени; board lock на pipeline.
- **Docs/skills:** game / board / messages blurbs под paced + deferred turn.

## References

- Explore: paced hop; fling-land = land-sense; no presentationDone; deferred auto-end/deadline; append to this change.
- Prior follow-ups: Anim-D4 / D13 land-before-overlay (секции 5–6) — остаются базой UX; Anim-D3 «client-only delay / server writes immediately» **снят** для trap chain + turn.
- Main specs: `openspec/specs/{lobby/rooms,game/board,game/move,game/finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
