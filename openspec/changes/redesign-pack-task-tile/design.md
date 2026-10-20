## Context

See `proposal.md` — Why. После `redesign-peek-task-modal` game peek и `PackTaskComposeDialog` уже на dots + word-badge + portrait filled/empty slots + 4-col pool. `PackTaskTile` остался carve-out (300×200, digit badge, split вопрос|compact slots, SC-PACK-238). Compose dialog сейчас `min(360…400px)` при пуле 4×`--peek-tile-max-w` (~43px) — заметно шире ряда из четырёх mini-cards. Compose dots шарят `.peek-diff-dot` (6px) с game peek.

**Пакет:** client only (`happy-tourist.github.io`). Server / HTTP / schema — без изменений.

## Goals / Non-Goals

**Goals:**

- List task tile → portrait pack-card family; header как peek/compose; только filled mini-slots; horizontal scroll.
- Compose: max-width ≤ 4 mini-cards + padding; dots ≈ высота badge (~20px) для hit-target.
- Снять SC-PACK-238 carve-out для filled chrome на tile.

**Non-Goals:**

- Server / HTTP `/api/content/*` / Colyseus room / schema / messages / DB — **без изменений** (FE-only change; answer `content`/`description` и `difficulty` 1|2|3 уже в существующем payload).
- Менять game peek width / dot size / layout.
- Редизайн answer playing-cards / PackTaskSetCardTile / catalog.
- Новые deps; глобальная смена Quasar tokens.

## Decisions

1. **Tile size** — portrait W &lt; H; resting band **W 156–180**, `min-height` ≈**206**, radius **12**, `--pack-card-*` surface (см. Visual Spec). Apply выбирает px внутри band; без жёсткого 300×200.
2. **Header** — reuse `.peek-diff-header` / `.peek-diff-dots` / `.peek-diff-badge`: dots left (**6×6** read-only), word badge center (h **20**); keys `content.difficultyWord.*`. Снять outline digit `q-badge` и primary-copy через `content.difficulty.{1|2|3}` на list tile (digit-only badge — out).
3. **Slots on tile** — только filled; chrome `.peek-slot-like--portrait` + filled primary border (`#1976d2` 1px); row gap **8**, `overflow-x: auto`, `flex-wrap: nowrap`. Empty omit. Если filled слотов **нет** — **не** рендерить ряд слотов и **не** показывать muted placeholder `content.slotEmpty` / «пусто» на list tile.
4. **Answer text / props** — primary = answer `content`; secondary = answer `description` if any (как peek filled). **Заменить** текущий `slotLabels: string[]` (сейчас parents прокидывают и empty как `content.slotEmpty`, без description) на filled-only prop вида `filledSlots: { content: string; description?: string }[]` (или эквивалент). Parents (`ContentPackTasksPage` / `ContentPackAddTaskSetPage` / `ContentStaffRequestPage` и helpers `taskSlotLabels` / `slotLabel`): фильтровать `!answerCardId`; резолвить `content` + optional `description` из загруженных `answerCards` — не новый HTTP.
5. **Compose width** — `max-width: min(4 * var(--peek-tile-max-w) + 3 * 8px + horizontal_pad, 92vw)`; убрать min/max 360/400. Pool `repeat(4, …)` gap 8.
6. **Compose dots size** — compose-only (`.pack-task-compose-card .peek-diff-dot` или `--compose-diff-dot-size: 20px`); shared base **6px** для game peek.
7. **Actions** — Edit/Delete full-width stacked bottom — as-is.
8. **Tests** — Vitest SC-PACK-223/226/238/278 (tile), SC-PACK-279/280/276/260 (compose); обновить устаревшие 300×200 / carve-out asserts.

**Alternatives rejected:** оставить 300×200 + только перекрасить header; сужать и game peek; общие 20px dots на peek (ломает visual spec peek); показывать dashed empty на list tile.

## Risks / Trade-offs

- [Узкая portrait + длинный вопрос] → Mitigation: scroll в зоне вопроса; не растягивать ширину.
- [Много слотов + horizontal scroll] → Mitigation: явный overflow-x; Vitest SC-PACK-278.
- [Compose уже dialog на узких экранах] → Mitigation: `min(…, 92vw)` сохранить; якорь — 4 tiles, не меньше mobile.
- [Shared CSS accidentally enlarges peek dots] → Mitigation: compose-scoped selector only; Vitest SC-PACK-280 asserts peek unchanged if covered, else compose-only class check.

## Migration Plan

Client-only UI. Нет data migration. Rollback = revert tile + compose CSS/markup.

## Open Questions

_(none — explore closed)_

## Implementation touchpoints

- `src/components/PackTaskTile.vue` — layout/template/styles; заменить digit badge + `slotLabels` на peek header + `filledSlots: { content; description? }[]` (filled-only; zero → no slots row).
- Parents (`ContentPackTasksPage`, `ContentPackAddTaskSetPage`, `ContentStaffRequestPage`) — переписать `taskSlotLabels` / `slotLabel` (или аналог) → filled-only resolve с `description`; обновить page-test stubs props. **Источник данных без нового HTTP:** `content` + optional `description` уже в payload answer card / task slots (`difficulty` 1|2|3, `answerCardId`) — resolve на client из уже загруженного pack/task-set state; server/API не менять.
- `src/components/PackTaskComposeDialog.vue` — card width; compose-scoped larger dots.
- `src/css/app.scss` — compose-only dot size; tile MAY reuse portrait slot classes (lift carve-out comments).
- Vitest: `PackTaskTile.test.ts`, `ContentPackTaskCompose.test.ts` (+ related page tests asserting 300×200 / SC-PACK-238).
- Skill notes (apply MUST sync — иначе агенты откатят portrait tile):
  - `.agents/skills/client/work-with-styles/pack-cards.md` — task **300×200** + SC-PACK-238 compact labeled carve-out
  - `.agents/skills/client/work-with-stores/content.md` — task 300×200 / Don’t carve-out
  - `.agents/skills/client/work-with-game-board/peek.md` — Don’t «force peek portrait AR onto PackTaskTile»
  - комментарии в `app.scss` про SC-PACK-238 carve-out

**OpenSpec scenario titles:** для SC-PACK-223 / SC-PACK-226 в delta **сохранять исторические заголовки** из main (`…vertically split…` / `…300×200`) — менять только тело GIVEN/WHEN/THEN. Rename заголовка ломает `openspec validate` / archive (scenario-loss).

Checklist: `tasks.md`.

## Visual Spec

**Source of truth:** this section. Token reuse from shipped peek/compose / pack-card (`app.scss` `.pack-card-grid` / `.peek-chrome`, `.peek-diff-*`, `.peek-slot-like--portrait`, `.peek-answer-pool`). No separate mock image — parity with existing pack-card + peek chrome. Values marked `approx` MAY land in a small band; apply MUST NOT invent a second token family.

### PackTaskTile (list) — structure

Order top → bottom (no pale mid-card dividers except actions top border):

1. Header (dots left + word badge center) — height 20px (same as `.peek-diff-header`)
2. Question region (flex grow; overflow-y auto)
3. Filled-slots row (overflow-x auto; nowrap)
4. Actions (Edit / Delete) when editable — full-width stacked text buttons

```
+---------------------------+  W 156–180; min-H ≥206; W < H
| ooo          лёгкая       |  header h=20; dots left; badge center
|      текст задания        |  pad / gap below
| [mini][mini][mini] -->    |  filled only; gap 8; overflow-x
|---------------------------|  1px --pack-card-splitter
| Edit                      |  --pack-card-action-h (~30)
| Delete                    |
+---------------------------+
```

### PackTaskTile — overall / surface

| Token | Spec |
|-------|------|
| Width | `approx` **156–180** CSS px (canon band = `PackTaskSetCardTile` 156; MAY be slightly wider for slot row) |
| Height | portrait **W &lt; H**; `min-height` **≈206** CSS px (task-set resting); MAY grow with long question scroll, not by widening past band |
| Border-radius | **12px** |
| Border | **1px** solid `var(--pack-card-border)` |
| Background / fg | `var(--pack-card-bg)` / `var(--pack-card-fg)` |
| Shadow (resting) | `var(--pack-card-shadow)` |
| Light surface | bg `#ffffff`; fg `rgba(0,0,0,0.87)`; muted `rgba(0,0,0,0.7)`; border `rgba(0,0,0,0.14)` |
| Dark surface | bg `#2a2a2a`; fg `rgba(255,255,255,0.92)`; muted `rgba(255,255,255,0.78)`; border `rgba(255,255,255,0.22)` |
| Root padding | horizontal **8–12** px; top **8–10** px; bottom **0** (actions carry own pad) — `approx` like task-set body `0 12px 6px` |

### PackTaskTile — element sizes / pads / gaps

| Element | Spec |
|---------|------|
| Header | height **20px**; width 100%; layout = `.peek-diff-header` (relative; dots absolute left; badge centered) |
| Dots | **3×** read-only; each **6×6** px; gap **2** px; border-radius 50%; fill count = difficulty; color = `--pack-ts-dot-{N}` via `--peek-diff-dot-color` (light: `#43a047` / `#f9a825` / `#e53935`; dark: `#66bb6a` / `#ffca28` / `#ef5350`) |
| Badge | height **20px**; pad **2×8** px; radius **999px**; font-size **11px**; weight **600**; line-height **1.2**; word-only (`content.difficultyWord.{1\|2\|3}`); tint `--peek-badge-{N}-bg` / `-fg` |
| Header → question gap | **8–12** px (`approx`; peek header `margin-bottom: 12`) |
| Question | `font-size` **13–15** px (`approx` 0.85rem) — **tile-scaled**, NOT game peek 24px (SC-BOARD-54); weight **600**; line-height **1.25–1.35**; color `--pack-card-fg`; **text-align: center**; overflow-y **auto**; word-break break-word; min-height **≈40–56** px flex region |
| Question → slots gap | **8–10** px |
| Slot mini-card | width `var(--peek-tile-max-w)` **43px**; AR **156∶206**; radius **12px**; pad **4×5** px; border **1px** solid `#1976d2` (filled primary thin — same as `.peek-slot-like--portrait.peek-slot-like--filled`); bg `--pack-card-bg` |
| Slot primary text | font-size **12px**; weight **600**; line-height **1.2**; center; clamp ≤3 lines |
| Slot secondary (description) | font-size **9px**; weight **400**; line-height **1.2**; color `--pack-card-muted`; clamp ≤2 lines; omit node when empty |
| Slots row | `display: flex`; `flex-wrap: nowrap`; gap **8** px; `overflow-x: auto`; `overflow-y: hidden` |
| Empty slots | **not rendered** |
| Zero filled slots | omit entire slots row; **no** muted `content.slotEmpty` body placeholder |
| Actions divider | border-top **1px** `--pack-card-splitter`; air above **≈3** px (`--pack-card` / task-set divider air) |
| Edit / Delete | full-width text; height `var(--pack-card-action-h)` **≈30** px (min 28 / max 32); font-size **≈0.78rem**; as current stacked flat dense — no Material corner icons |
| Icons | none (dots are CSS circles; no Material icons on tile body) |

### PackTaskTile — alignment

| Element | Align |
|---------|-------|
| Dots | left edge of content |
| Badge | horizontal center of header |
| Question | center |
| Slot texts | center |
| Edit / Delete labels | center (Quasar full-width text) |

### PackTaskTile — hover / focus

List tile is **read-only chrome** (no click-to-edit on body). Hover chrome **not required** (not shown on parity reference for list tiles). Editable actions: Quasar flat button hover as today. Cascade outline unchanged (`2px` warning).

### PackTaskTile — copy / i18n

| Surface | Key / copy |
|---------|------------|
| Badge words | `content.difficultyWord.1` / `.2` / `.3` (лёгкая / средняя / сложная) |
| Edit | `content.editTask` |
| Delete | `content.deleteTask` |
| Fallback blank question | parent `content.taskN` (as today) |

### Slot data binding (FE)

Props for filled slots MUST carry `{ content: string; description?: string }` (or equivalent dual arrays) resolved on client from loaded pack `answerCards` by `answerCardId` — **no new HTTP**. Empty `answerCardId` slots omitted before render. When the resulting array is empty, omit the slots row entirely (do **not** show `content.slotEmpty` / generic empty copy in the tile body). Do **not** keep passing a parallel `slotLabels: string[]` of empty placeholders once `filledSlots` (or equivalent) is wired.

### Compose dialog polish

| Token | Spec |
|-------|------|
| Width formula | `max-width: min(4 * var(--peek-tile-max-w) + 3 * 8px + horizontal_pad, 92vw)` where pool gap = **8px** (`.peek-answer-pool`); `horizontal_pad` = dialog section pad **≈32** px total (16+16 Quasar section) **or** explicit card pad if restructured — dialog MUST NOT stay at fixed min **360** / max **400** |
| Min-width | drop inflated `min(360px, 92vw)`; MAY use `min(formula, 92vw)` for both min and max, or min ≈ content |
| Dots (compose-only) | visual Ø **≈20** px (= `.peek-diff-badge` height); gap MAY stay **2–4** px; selector scoped under `.pack-task-compose-card` (or `--compose-diff-dot-size: 20px`); shared `.peek-diff-dot` base remains **6×6** for game peek |
| Pool | still `repeat(4, minmax(0, var(--peek-tile-max-w)))`; gap 8 — unchanged |
| Game peek | width + 6px dots **unchanged** |

### CSS scope rule

Larger dots and narrower width apply **only** under compose dialog root (`.pack-task-compose-card` / equivalent). Shared `.peek-diff-dot` base stays 6px for game peek / any other consumer. Tile MAY reuse `.peek-diff-*` + `.peek-slot-like--portrait` without compose-only enlargements.
