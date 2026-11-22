## Context

See `proposal.md` — Why / Scope. Сейчас finish/return travel в client реконструируется из sync (`finished` / coords) через несколько Set/Map, seed `from`, double-`rAF` clear и defer-флаги катапульты; при раннем sync фигура выпадает из board overlay → snap-vanish. Серверный finish/return authority менять не нужно.

## Goals / Non-Goals

**Goals:**

- Один presentation-owned **travel clip** на фигуру для finish (клик→центр, catapult→центр) и return (nearest center→кольцо).
- Пока clip жив — только он задаёт visual `row/col` (+ fade phase для finish); sync не снимает фигуру с доски и не snap’ает return на кольцо.
- Упростить Game board: убрать хрупкий reconstruct-стек finish travel (D3 pinned).
- Поведение на всех зрителях доски; mid-join без observed transition — omit.

**Non-Goals:**

- Менять server schema/messages или откладывать `finished` на сервере.
- Переписывать ordinary move / non-center catapult fling pin path (кроме стыка «после vanish → start finish clip»).
- Редизайн таймингов (~550 / ~300 finish; return ≈ ordinary move travel).

## Decisions

### D1 — Presentation-owned clip вместо reconstruct-from-sync

- **Choice:** `Map<pieceKey, TravelClip>` с полями `{ kind: 'finish' | 'return', from, to, phase: 'hold' | 'travel' | 'fade' }` (или эквивалент). Старт clip’а в момент известного `from`; board overlay включает фигуру, пока clip существует. Sync `finished`/coords — confirmation / конечная истина после конца clip’а, не триггер «угадать from».
- **Why:** Убирает гонку paint/`from`/rAF; отвечает SC-FINISH-35/36/15/19/20.
- **Alt:** Допиливать `lastKnown` + `finishAnimFrom` + defer — rejected (флак). Сервер delay `finished` — rejected (proposal out of scope).

### D2 — Где стартовать clip

- **Choice:**
  - **Own click→center:** при успешной отправке `move` на center — `from` = локальная клетка до send; `to` = выбранная center-клетка.
  - **Catapult→center:** после полного vanish hop’а (существующий queue) — `from` = клетка катапульты; `to` = center dest fling’а; не параллельно overlay.
  - **Чужой finish / spectator:** при observed transition unfinished→finished на center — `from` = последняя известная non-center клетка (лёгкий `lastKnownBoardCellByKey` mirror — **не** reconstruct-стек); `to` = sync center coords; если `from` неизвестен (mid-join) — omit travel.
  - **Return:** при observed finished→unfinished на кольцо (или own submit `returnFromFinish`) — `from` = nearest center (Chebyshev к ring; ties: lower row, then lower col), `to` = ring cell.
- **Why:** `from`/`to` известны в событии; не склеиваются из гонки patch’ей.
- **Alt:** Только watch(`finished`) — rejected (корень бага).

### D3 — Удаляемый client-стек

- **Choice (pinned):** Удалить как истину finish/return presentation: `finishAnimFromByKey` + clear после double-`rAF`, `disappearingKeys` как единственный visibility gate finished-фигур, `catapultDeferFinishKeys` как finish-fade gate, `resolveFinishTravelFrom` / `lib/finishTravelFrom.ts` (+ unit-тесты helper’а), finish-seed через `boardActionPreCell`. **Оставить** лёгкий `lastKnownBoardCellByKey` mirror только как источник `from` для D2 observed чужой finish (не paint override). **Visibility finished на доске** (обязательно для SC-FINISH-19/20/36/**37**): (a) **активный finish clip** (hold|travel|fade) — SC-FINISH-36, **или** (b) **catapult presentation ownership** pin/visual/queue на эту фигуру **до** старта finish clip после vanish (ранний sync `finished` во время overlay **не** снимает фигуру) — SC-FINISH-37, **или** (c) mid-join omit (уже finished без observed transition — без travel). Visibility return travel = **активный return clip**. Catapult pin/visual оставить и для center-fling hop (pre-clip), и для **не**-center fling path. SC-FINISH-36/37 — **новые** scenario ID (в main отсутствуют; в delta Traceability = added).
- **Why:** Один механизм вместо пяти флагов; lastKnown ≠ reconstruct; pre-clip catapult window не создаёт snap-absent при early `finished`.
- **Alt:** «Удалить или сузить» без выбора / обернуть clip поверх старых флагов — rejected. «Visibility = только finish clip» без catapult-owned pre-clip — rejected (ломает SC-FINISH-20 / SC-FINISH-37).

### D4 — CSS / фазы

- **Choice:** Явные фазы hold→travel→fade для `kind: 'finish'` (hold = один committed paint на `from`, затем travel на `to` ~550 ms = product `FINISH_TRAVEL_MS`, затем fade ~300 ms = `FINISH_FADE_MS`). Для `kind: 'return'`: hold→travel с timing ordinary move travel (~`MOVE_ANIM_MS`); без finish-fade (фигура остаётся на кольце). **Binding к существующему board chrome (extends, не новый stack):** `pieceStyle` / `--prow`/`--pcol` читают clip `from` на hold и clip `to` на travel|fade; fade-фаза finish выставляет класс `.piece--disappearing` (уже ~550 slide + ~300 fade delay); hold может использовать `piece--no-transition` / suppress на один paint, затем снять suppress чтобы travel стартовал **по фазе clip**, не по double-`rAF` clear override. `boardPieces` включает finished, пока (a)|(b) из D3; `hasVisualBoardPresentation` / busy учитывают живой travel clip (вместо size checks `finishAnimFrom`/`returnAnimFrom`). Не полагаться на «снять override чтобы CSS сам догадался» как на единственный триггер. Sync `finished`/coords — confirmation после конца clip’а.
- **Why:** Тот же visual contract (~550+~300 / move travel), без rAF-гонки; явный стык с текущими CSS-классами GamePage.
- **Alt:** Web Animations API — optional later; не обязателен, если фазы стабильны на CSS transition.

### D5 — Тесты

- **Choice:** Behavioral vitest: clip/`from` обязателен при finish с non-center origin (SC-FINISH-35/36); early `finished` во время catapult overlay не drop до старта clip (SC-FINISH-37); return clip `from` = nearest center (SC-FINISH-15); source-only SFC regex не считать достаточным для SC-FINISH-35. Обновить client board skill / AGENTS при apply, если канон ещё ссылается на `finishTravelFrom`.
- **Why:** Спека уже требует behavioral path.

## Risks / Trade-offs

- **[Risk] Чужой клиент без last-known non-center** → Mitigation: omit travel (mid-join); не snap-vanish с «пустого» from — либо clip с валидным from, либо omit.
- **[Risk] Двойной старт clip (own submit + sync watch)** → Mitigation: start idempotent per pieceKey (второй start ignored while clip alive).
- **[Risk] Регрессия catapult multi-hop** → Mitigation: finish clip только после last vanish; промежуточные hops — существующий fling path.
- **[Risk] Return timing vs move** → Mitigation: kind `'return'` использует ordinary move travel ms (~`MOVE_ANIM_MS`); finish — ~550 travel + ~300 fade.

## Migration Plan

1. Client-only apply в `happy-tourist.github.io`; server не деплоить ради этого change.
2. Rollback = revert client commit.
3. После verify — sync/archive OpenSpec как обычно.

## Technical prerequisites

None open — Vue/CSS/Colyseus mirror уже в стеке; новых npm/deps нет.

## Key touch points (indicative)

**Client:** `GamePage` — `boardPieces` visibility (D3 a|b), `pieceStyle` / `--prow`/`--pcol`, `.piece--disappearing` / suppress, `hasVisualBoardPresentation` / busy, catapult queue release→finish clip (D2); optional thin `lib` helper for clip types; remove/replace `finishTravelFrom` + reconstruct Maps; vitest board/finish travel (**behavioral** + обновить `GameBoardWire` asserts старого reconstruct-стека); meta skill `work-with-game-board` при apply.

### Server contract (reuse-only)

Новой server-работы **нет**. Client presentation binds **existing** room `tourist` surface (не менять, не откладывать `finished`):

| Surface | Role for this change |
|---------|----------------------|
| `onMessage('move')` `{ pieceId, row, col }` | own click→center start clip; authority finish on center |
| `onMessage('returnFromFinish')` `{ pieceId, row, col }` | own return submit; authority unfinished on ring |
| Schema piece `finished` + `row`/`col` | sync confirmation after clip; observed finish/return transitions |
| Paced catapult relocate → `applyCenterFinish` | `finished` MAY land after reveal enqueue; client still owns post-vanish finish travel |

No new schema fields, messages, HTTP, auth, or DB. Nearest-center `from` for return travel is **client** geometry (Chebyshev), not a server field.

Чеклист реализации — `tasks.md`.
