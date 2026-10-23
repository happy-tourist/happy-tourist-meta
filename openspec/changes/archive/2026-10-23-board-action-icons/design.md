## Context

See `proposal.md` — Why. Один change: wire illustrated icons → scale/circumference/flash → clearance / say-picker / finish travel → crop assets + focus under end-turn.

Runtime (shipped): glyph/hit **36×36** (`.board-action-icon` fills hit); board/return clearance above sprite (`translate(-50%, calc(-100% + …))`); focus at avatar square corner under end-turn; say-picker shifted right; finish travel `FINISH_TRAVEL_MS` ~550 + `FINISH_FADE_MS` ~300. Ассеты: `src/assets/board/{peek,push,return,flag,focus,end-turn,rescue,say}.png` (cropped to opaque bbox).

## Goals / Non-Goals

**Goals**

- Illustrated icons во всех восьми слотах без solid colored circular chrome; PNG cropped to opaque edges.
- Glyph/hit **36×36** (glyph fills hit); peek outcome flash 10 с.
- Peek/rescue/push/return: hit целиком **над** спрайтом.
- Focus **под end-turn** на угол квадрата аватара; say @12 и end-turn @3 на окружности (без контроля на 9; не в budgets).
- Say-picker полностью видим: сдвиг **вправо**, левый край не клипается.
- Finish travel ~**550 ms** + fade ~**300 ms**; без drop без travel (в т.ч. клик по клетке финиша).

**Non-Goals**

- Budgets Material icons / ready / peek modal / say-picker copy list.
- Удлинять catapult overlay 1500 ms; flag scale; перерисовка PNG.
- Focus under peeks in budgets — superseded by D11 (under end-turn).
- `padding-left` на HUD ради focus@9 — superseded.
- Intermediate ×2-then-shrink sizes (~18 glyph / ~32 board hit) — superseded by final 36×36 glyph-fills-hit after crop.

## Decisions

### D1–D4 — Wire illustrated icons (applied)

D1 transparent hit + `<img>` · D2 `src/assets/board/*.png` · D3 eight slots · D4 Vitest clock/CSS/DOM — tasks 1–2 `[x]`.

### D5–D9 — Scale, circumference, peekClose, flash (applied)

D5 size family → final **36×36** glyph-fills-hit · D6 presence clock say@12 / end-turn@3 (focus@9 superseded by D11) · D7 `peekClose.reason` · D8 header flash · D9 skills — tasks 3–6 `[x]`.

### D10 — Affordance над спрайтом (board + return)

Якорь = верх клетки/слота; низ hit выше спрайта:

```
top: <cellTop or -gap>;
transform: translate(-50%, calc(-100% + <nudge>));
```

Gap ~**8 px** (product default). Одинаково для peek / rescue / push / return. Flag не трогать.

### D11 — Focus под end-turn (не @9, не under peeks)

`.focus-affordance` на own presence marker **ниже end-turn**, hit-center на **bottom-right углу** квадрата, в который вписан avatar (**72×72** в маркере **96×96** → `left/top: calc(50% + 36px); transform: translate(-50%,-50%)`). Transparent chrome; glyph/hit **36×36**. Say @12; end-turn @3; **слева (9) пусто**; **не** в `.presence-budgets`. Visibility/активация focus без изменений.

**Отвергнуто:** focus@9 + HUD `padding-left`; focus under peeks в budgets.

### D12 — Say-picker вправо

Открытый `.say-picker` сместить **вправо** (CSS `left` / `transform` / margin), чтобы **левый край** пресетов не обрезался HUD scroll/overflow. Say affordance остаётся hit-center @12.

### D13 — Finish travel длиннее + race

| Константа / CSS | Было | Стало |
|-----------------|------|-------|
| Finish slide | ~250 ms | ~**550 ms** (`FINISH_TRAVEL_MS`) |
| Finish fade | ~200 ms | ~**300 ms** (`FINISH_FADE_MS`, delay = slide) |

Все enter-center paths: ordinary move (в т.ч. клик по center), push→center, catapult→center. Overlay catapult не трогать.

**Race:** finished sync не роняет piece без travel+fade; audit deferred/pin/`finishAnimFrom`. FE-only для timing после vanish.

### D14 — Crop assets (applied)

Подрезать `src/assets/board/*.png` по opaque bbox (круглые → квадрат). Размер UI остаётся **36×36** glyph-fills-hit после crop (не возвращать к ~18/~32).

### D15 — Skills (follow-up)

Notes: clearance; focus under end-turn; say-picker right; finish ~550/300; cropped PNGs; 36×36.

## Risks / Trade-offs

- [Focus у угла аватара → overlap с end-turn] → Mitigation: distinct anchors (end-turn circumference @3 vs square corner); Vitest non-overlap.
- [Picker вправо → может выйти за правый край на узком HUD] → Mitigation: prefer unclip left first; if needed, clamp within scrollport without re-clipping left.
- [Дольше finish → дольше isBoardBusy] → Mitigation: ок SC-BOARD-27.

## Migration Plan

Client (+ meta skills) deploy; server с `peekClose.reason`. Rollback = revert client (+ server reason field if needed).

## Open Questions

(нет)

## Skills / apply notes

- Client: `GamePage.vue` — affordance CSS; focus under end-turn; say-picker offset; finish timers; cropped assets.
- Tasks: блоки 1–10 все `[x]`.
