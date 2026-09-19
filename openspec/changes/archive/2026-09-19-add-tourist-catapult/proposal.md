## Why

На поле уже есть решётки и катапульты (seed / fling / lobby density / paced hop / deferred turn). Playtest после §7 показал ещё два дефекта:

1. **Ход не уходит после решётки** — шаги кончились, остался peek на live `*`, фишка попала за решётку → peek этой фишки запрещён, остальные на старте peek не дают; `onTrapPipelineIdle` не делает re-eval auto-end (только заранее выставленный `pendingTurnAdvance`), ход зависает.
2. **Fling на финиш без полёта** — при paced finish sync приходит **после** reveal; client enqueue не помечает deferred finish → фишка исчезает с клетки катапульты без finish travel.

Нужен follow-up §8: **re-eval auto-end на idle**; **всегда** finish travel после vanish (в т.ч. цепочка hop→…→центр).

## What Changes

- В create — отдельный выбор плотности катапульт: мало / средне / много (те же **12% / 22% / 35%**, default средне) — уже сделано.
- Seed / fling geometry / broken / overlap с решётками — уже сделано; экономика steps **без изменений** (fling не −1).
- **Paced trap resolve / deferred turn / sequential UX** — уже сделано (§7).
- **Idle re-eval (F1):** на `onTrapPipelineIdle` — если был `pendingTurnAdvance` (deadline / заранее no-actions) → `advanceTurn`; иначе заново `maybeAutoEndTurn` (+ solo exhaustion), чтобы решётка, снявшая единственный legal peek, сменила ход.
- **Finish travel always (F2):** после vanish hop’а, если piece `finished` / dest = center — всегда finish travel с клетки катапульты → центр → fade (не телепорт); каждый hop цепочки анимируется; finish travel после **последнего** vanish.

## Scope

- **Пакеты:** client + server (room `tourist`).
- **Capability ID:**
  - `lobby/rooms` — create option плотности (done);
  - `game/board` — overlay; sequential hops; grille после prior hops; board lock; finish travel после last hop;
  - `game/move` — paced + deferred + **idle re-eval auto-end** после trap (в т.ч. grille);
  - `game/finish` — fling на center; **всегда** finish travel после vanish (paced).
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
- `game/board`: sequential hop timeline; always-animated hops; finish travel after last vanish.
- `game/move`: paced; deferred pending advance; **re-eval auto-end on idle** when trap removed available actions.
- `game/finish`: fling на center = finish; **always** finish travel after catapult vanish under paced sync.

## Impact

- **Server:** `onTrapPipelineIdle` — pending → advance; else re-eval auto-end/solo; mocha SC-MOVE-93.
- **Client:** deferred finish travel когда finish sync приходит после reveal enqueue; цепочка hop→центр; SC-FINISH-20 / board.
- **Docs/skills:** idle re-eval + finish-after-vanish blurbs.

## References

- Explore §8: F1 idle re-eval after grille; F2 always finish travel; Q1 every hop animated; append to this change.
- Prior: §7 paced + deferred; §5–6 land-before-overlay / sequencing.
- Main specs: `openspec/specs/{lobby/rooms,game/board,game/move,game/finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
