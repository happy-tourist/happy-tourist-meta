## Context

See `proposal.md` — Why. Один change: wire illustrated icons → scale/circumference/flash → clearance, focus under peeks, say-picker shift, finish travel.

Runtime после applied блоков: glyph ~36 / board hit ~52 / presence hit ~60; board/return overlap головы; focus still at 9 o’clock (to relocate); say-picker left edge clipped by HUD overflow; finish travel `MOVE_ANIM_MS=250` + `FINISH_FADE_MS=200`. Ассеты: `src/assets/board/{peek,push,return,flag,focus,end-turn,rescue,say}.png`.

## Goals / Non-Goals

**Goals**

- Illustrated icons во всех восьми слотах без solid colored circular chrome.
- ~×2 glyph/hit; peek outcome flash 10 с.
- Peek/rescue/push/return: hit целиком **над** спрайтом.
- Focus под peeks-счётчиком в own budgets; say @12 и end-turn @3 на окружности (без контроля на 9).
- Say-picker полностью видим: сдвиг **вправо**, левый край не клипается.
- Finish travel ~**550 ms** + fade ~**300 ms**; без drop без travel (в т.ч. клик по клетке финиша).

**Non-Goals**

- Budgets Material icons / ready / peek modal / say-picker copy list.
- Удлинять catapult overlay 1500 ms; flag scale; перерисовка PNG.
- `padding-left` на HUD ради focus@9 — superseded by relocating focus.

## Decisions

### D1–D4 — Wire illustrated icons (applied)

D1 transparent hit + `<img>` · D2 `src/assets/board/*.png` · D3 eight slots · D4 Vitest clock/CSS/DOM — tasks 1–2 `[x]`.

### D5–D9 — Scale, circumference, peekClose, flash (applied)

D5 ~×2 · D6 presence clock (then 12/9/3) · D7 `peekClose.reason` · D8 header flash · D9 skills — tasks 3–6 `[x]`. Circumference for **focus@9** superseded by D11 below; say@12 and end-turn@3 remain.

### D10 — Affordance над спрайтом (board + return)

Якорь = верх клетки/слота; низ hit выше спрайта:

```
top: <cellTop or -gap>;
transform: translate(-50%, -100%);
```

Gap ~**8 px**. Одинаково для peek / rescue / push / return. Flag не трогать.

### D11 — Focus под peeks (не на 9 o’clock)

Перенести `.focus-affordance` из presence-marker (left/9) **в** `.presence-budgets` / `.budget-counters`: сразу **под** ряд peeks (`visibility` + число). Сохранить ×2 glyph/hit family (~56–64) и transparent chrome. End-turn остаётся на окружности @3; say @12; **слева на маркере пусто**. Visibility/активация focus (когда есть actionable tourist) без изменений.

**Альтернатива:** padding-left + z-index при focus@9 — отвергнута (user: перенос под peeks).

### D12 — Say-picker вправо

Открытый `.say-picker` сместить **вправо** (CSS `left` / `transform` / margin), чтобы **левый край** пресетов не обрезался HUD scroll/overflow. Say affordance остаётся hit-center @12. Достаточно полного видимого пикера — без фиксированного px в spec (product: left edge fully visible + presets choosable).

### D13 — Finish travel длиннее + race

| Константа / CSS | Было | Стало |
|-----------------|------|-------|
| Finish slide | ~250 ms | ~**550 ms** |
| Finish fade | ~200 ms | ~**300 ms** (delay = slide) |

Все enter-center paths: ordinary move (в т.ч. клик по center), push→center, catapult→center. Overlay catapult не трогать.

**Race:** finished sync не роняет piece без travel+fade; audit deferred/pin/`finishAnimFrom`. FE-only для timing после vanish.

### D14 — Skills (follow-up)

Notes: clearance; focus under peeks; say-picker right; finish ~550/300.

## Risks / Trade-offs

- [Focus в budgets → тесный столбец] → Mitigation: stack under peeks only when visible; keep hit usable.
- [Picker вправо → может выйти за правый край на узком HUD] → Mitigation: prefer unclip left first; if needed, clamp within scrollport without re-clipping left.
- [Дольше finish → дольше isBoardBusy] → Mitigation: ок SC-BOARD-27.

## Migration Plan

Client (+ meta skills) deploy; server уже с `peekClose.reason`. Rollback = revert client.

## Open Questions

(нет)

## Skills / apply notes

- Client: `GamePage.vue` — affordance CSS; move focus into budgets; say-picker offset; finish timers.
- Tasks: блоки 7–9; 1–6 остаются `[x]`.
