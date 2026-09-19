## Why

На поле уже есть ход, peek, спасение, возврат с финиша и **толчок**, но после первой реализации остались UX-дыры: иконки действий сидят в углу клетки (не как return в strip), «Завершить ход» — крупная текстовая кнопка в dock, а push на центр визуально «съедает» цель без travel→fade как у обычного хода в финиш.

## What Changes

- Действие **push** (уже в коде): −1 step; far-side геометрия; finish/trap side-effects; auto-end; message `push`.
- Return-from-finish: зелёная иконка над strip; без модалки; слот body не стартует return (уже в коде).
- **Polish (этот апдейт):** peek / rescue / push affordances — **сверху по центру** над туристом (как return в strip).
- End-turn: убрать текстовый dock; иконка `skip_next` **справа по центру** у своего аватара (зеркало say сверху по центру); клик сразу шлёт end-turn **без** диалога и без видимого лейбла; solo по-прежнему без кнопки.
- Push (и тот же путь) на центр: цель **доежает** на финишную клетку и исчезает как при обычном ходе (`SC-FINISH-01`), не пропадает мгновенно.

## Scope

- **Пакеты:** client (+ server уже для push; polish — client-only, кроме уже закрытых server-задач).
- **Capability ID:**
  - `game/move` — push + centering push/rescue + push→finish travel/fade;
  - `game/finish` — return strip UX (уже); finish disappear parity для push;
  - `game/board` — peek eye сверху по центру;
  - `game/presence` — end-turn icon у аватара.
- **Экраны:** Game (board affordances + strip + own presence).
- **Контракт:** room `tourist`; wire push / endTurn / returnFromFinish без смены shape.

## Out of scope

- Толкать пойманного (только rescue).
- Красные кольца на клетку назначения пуша.
- Confirm-dialog на end-turn; видимые подсказки/tooltips (позже).
- Смена набора Material-иконок целиком (позже).
- Auth / lobby / HTTP / reconnect.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: push; centering rescue/push; push→finish presentation.
- `game/finish`: return strip без модалки; disappear parity при finish от push.
- `game/board`: peek affordance сверху по центру.
- `game/presence`: end-turn — иконка справа по центру у аватара, без текстового dock.

## Impact

- **Server:** push validate/handler/tests (уже); polish не требует server.
- **Client:** GamePage affordance CSS/layout; end-turn у presence; finish travel для push; skills/i18n по необходимости.
- **Docs/skills:** client board/pages/presence; meta индекс при drift.

## References

- Explore (этот чат): push D*; polish D1=center peek/rescue/push; D2=end-turn без dialog; D3=`skip_next`; D4=push→finish как ход; мелочи: icon справа по центру как say сверху; без видимого текста; solo без end-turn.
- Main specs: `game/move`, `game/finish`, `game/board`, `game/presence`.
- Sibling AGENTS; `docs/projects-map.md`.
