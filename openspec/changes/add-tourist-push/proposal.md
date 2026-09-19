## Why

На поле уже есть ход, peek, спасение, возврат с финиша и **толчок**, но после первой реализации остались UX-дыры: иконки действий сидят в углу клетки (не как return в strip), «Завершить ход» — крупная текстовая кнопка в dock, а finish на центр визуально «съедает» туриста — push уже доезжает с travel→fade, а **свой ход на центр** по-прежнему пропадает без той же анимации.

## What Changes

- Действие **push** (уже в коде): −1 step; far-side геометрия; finish/trap side-effects; auto-end; message `push`.
- Return-from-finish: зелёная иконка над strip; без модалки; слот body не стартует return (уже в коде).
- **Polish (уже):** peek / rescue / push affordances — **сверху по центру** над туристом (как return в strip).
- End-turn: убрать текстовый dock; иконка `skip_next` **справа по центру** у своего аватара; клик сразу шлёт end-turn **без** диалога и без видимого лейбла; solo без кнопки (уже).
- Push→center: цель **доежает** и исчезает (уже).
- **Fix (этот апдейт):** свой `move` на центр — тот же travel с pre-move клетки + disappear, что push→center; не исчезать с соседней клетки без кадра travel.

## Scope

- **Пакеты:** client (+ server уже для push; polish/fix — client-only).
- **Capability ID:**
  - `game/move` — push + centering + push→finish travel/fade;
  - `game/finish` — return strip UX; finish travel+disappear для **move и push**;
  - `game/board` — peek eye сверху по центру;
  - `game/presence` — end-turn icon у аватара.
- **Экраны:** Game (board affordances + strip + own presence).
- **Контракт:** room `tourist`; wire без смены shape.

## Out of scope

- Толкать пойманного (только rescue).
- Красные кольца на клетку назначения пуша.
- Confirm-dialog на end-turn; видимые подсказки/tooltips (позже).
- Смена набора Material-иконок целиком (позже).
- Auth / lobby / HTTP / reconnect.
- Server schema / finish wire (уже ок).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: push; centering rescue/push; push→finish presentation.
- `game/finish`: return strip без модалки; travel+disappear parity для **move и push** на центр.
- `game/board`: peek affordance сверху по центру.
- `game/presence`: end-turn — иконка справа по центру у аватара, без текстового dock.

## Impact

- **Server:** push validate/handler/tests (уже); fix не требует server.
- **Client:** GamePage finish-travel harden (submit capture + watch); skills при drift.
- **Docs/skills:** client board/pages; meta индекс при drift.

## References

- Explore: push D*; polish D1–D4; finish-travel bug — свой ход на центр без анимации при рабочем push→finish.
- Main specs: `game/move`, `game/finish`, `game/board`, `game/presence`.
- Sibling AGENTS; `docs/projects-map.md`.
