## Context

See `proposal.md` — Why. Client-only visual swap on Game affordances. Server untouched — no new `onMessage` / schema / HTTP; activation keeps existing tourist-room messages (`peek`, `rescue`, `push`, `returnFromFinish`, `endTurn`, `say`) and local focus.

Текущее состояние: `GamePage` рисует Material `q-icon` внутри цветных кругов (blue/green) для peek / rescue / push / return / focus / end-turn / say и Material `flag` на finished strip. Готовые RGBA PNG уже лежат в client `src/assets/board/` (`peek`, `push`, `return`, `flag`, `focus`, `end-turn`, `rescue`, `say`) — explore: crop + rename; raw `*-removebg*` удалены.

## Goals / Non-Goals

**Goals**

- Подключить illustrated icons во все восемь affordance-слотов.
- Убрать solid colored circular chrome; сохранить hit-area ~32–36px и clock placement (12 / 9 / 3).
- Простой hover (`opacity` / `brightness`).

**Non-Goals**

- Budgets HUD, ready, peek modal, say-picker list chrome.
- Server / protocol: **no BE work** — no new/changed `onMessage`, schema fields, HTTP routes, or DB; activation continues to use existing tourist-room messages listed in Context (focus stays local).
- Перерисовка или повторный crop ассетов (уже готовы).

## Decisions

### D1 — Без цветного фона (chrome → transparent hit)

Заменить визуал кнопки: transparent/`background: transparent` + `<img>` illustrated icon; убрать solid `rgba(…)` / hex fill **и** hover-fill у `.peek-affordance` / `.rescue-affordance` / `.push-affordance` / `.return-affordance` / `.focus-affordance` / `.end-turn-affordance` / `.say-affordance`. Убрать medallion-like `box-shadow` на этих affordances (он читается как цветной круг вместе с fill). Hit-box размеры (~32 board/strip, ~36 presence) и absolute offsets оставить. Glyph: `<img>` с `object-fit: contain`, ориентир ~18 CSS px как у текущего `q-icon` (внутри hit, не раздувать hit). Hover: лёгкий `opacity` или `filter: brightness` — без новой рамки / нового цветного fill. FE-default: `opacity` на `:hover`/`:active` (без нового fill).

**Finished-flag (не кнопка):** слот `.my-tourist-finish-icon` на strip — заменить Material `q-icon name="flag"` на `<img src=…/flag.png>` (тот же class / top-right). У него **нет** solid circular chrome — только glyph swap. Не полагаться на CSS `color: #2e7d32` (Material tint): PNG as-is; существующий лёгкий `drop-shadow` для контраста можно оставить. Не делать flag кликабельным.

**A11y glyph:** на кнопках-affordance сохранить существующий `:aria-label` / `$t(…)` на `<button>`; у `<img>` внутри — `alt=""` (декоративный). У finish-flag — перенести `:aria-label="$t('game.finishStripAria')"` на `<img>` (или wrapping span), как сейчас на `q-icon`.

**Альтернатива:** оставить цветной круг + img внутри → отброшена (двойная «медалька»).

**Visual Spec:** n/a — change asset-driven (готовые PNG + reuse существующих hit CSS), не mock-driven pixel Spec.

### D2 — Каталог ассетов

Канон: `src/assets/board/{peek,push,return,flag,focus,end-turn,rescue,say}.png`. Import в `GamePage` (или тонкий helper рядом, если DRY). Не класть board-action PNG в `assets/content/` (там catalog SVG).

### D3 — Scope слотов

| Slot | Asset | Was |
|------|-------|-----|
| peek | `peek.png` | `visibility` |
| rescue | `rescue.png` | `lock_open` |
| push | `push.png` | `swipe` |
| return | `return.png` | `undo` |
| finished strip | `flag.png` | `flag` |
| focus | `focus.png` | `center_focus_strong` |
| end-turn | `end-turn.png` | `skip_next` |
| say | `say.png` | `chat_bubble_outline` |

Out: budgets, ready.

### D4 — Тесты

Текущие suites (`GamePresenceFocus`, `GameBoardWire`, strip/finish) в основном проверяют **наличие классов** и **clock/CSS offsets** (`cssRuleBody('say-affordance'|…)`), а не Material `q-icon name=…`. После wire:

- геометрию clock / offsets **сохранить**;
- в CSS-правилах affordances — assert **отсутствие** solid `rgba(…)` / hex fill и medallion-like `box-shadow` (transparent hit);
- glyph: `<img>` (src / class) вместо Material name, где suite уже смотрит DOM;
- budgets / ready chrome **не** трогать в assertions.

## Risks / Trade-offs

- [Мелкие детали PNG на 32–36px] → Mitigation: `object-fit: contain`; при необходимости чуть уменьшить padding, не раздувать hit-area.
- [Say pin / flag не круг] → Mitigation: hit-area всё ещё ~квадрат/круг; SC-PRESENCE-33 говорит про hit overlap, не про форму глифа.
- [Тесты на solid fill / Material glyph] → Mitigation: D4 — CSS assert transparent hit; DOM `<img>` где suite смотрит glyph; clock offsets не ломать.

## Migration Plan

Client-only deploy. Rollback = revert GamePage chrome + assets folder. Нет data migration.

## Open Questions

(нет — D1–D4 закрыты в explore)

## Skills / apply notes

- `.agents/skills/client/work-with-game-board` (+ `focus.md` / `peek.md`): affordance visuals; strip flag/return; presence clock icons — illustrated PNG, no Material / solid colored circle.
- `work-with-styles/board.md` + `work-with-pages/game-page.md`: CSS class notes for transparent hit chrome.
- Без обязательного нового token set.
- Tasks: см. `tasks.md`.
