## Context

See `proposal.md` — Why. После `redesign-peek-task-modal` game peek и `PackTaskComposeDialog` уже на dots + word-badge + portrait filled/empty slots + 4-col pool. Первый apply этого change уже дал portrait `PackTaskTile` + filledSlots + compose width/dots. Follow-up polish (explore): outline+icon actions, question **24px**, slots centered + raised air, compose Save как peekSubmit без Cancel, width **+≈50px**.

**Пакет:** client only (`happy-tourist.github.io`). Server / HTTP / schema — без изменений.

## Goals / Non-Goals

**Goals:**

- List task tile → portrait pack-card family; header как peek/compose; только filled mini-slots; horizontal scroll; question **24px**; slots centered + not glued to actions.
- Editable actions → outline + icon parity with task-set cards (Delete MAY negative).
- Compose: width = 4 mini-cards + pad **+ ≈50px**; dots ≈ badge height; Save = peekSubmit chrome; no Cancel button.
- Снять SC-PACK-238 carve-out для filled chrome на tile.

**Non-Goals:**

- Server / HTTP `/api/content/*` / Colyseus room / schema / messages / DB — **без изменений**.
- Менять game peek width / dot size / layout (peek question 24px is the **source** for list tile; do not change peek itself).
- Редизайн answer playing-cards / PackTaskSetCardTile / catalog.
- Explicit header close (X) on compose.
- Новые deps; глобальная смена Quasar tokens.

## Decisions

1. **Tile size** — portrait W &lt; H; resting band **W 156–180**, `min-height` ≈**206**, radius **12**, `--pack-card-*` surface (см. Visual Spec). Apply выбирает px внутри band; без жёсткого 300×200.
2. **Header** — reuse `.peek-diff-header` / `.peek-diff-dots` / `.peek-diff-badge`: dots left (**6×6** read-only), word badge center (h **20**); keys `content.difficultyWord.*`. Снять outline digit `q-badge` и primary-copy через `content.difficulty.{1|2|3}` на list tile (digit-only badge — out).
3. **Slots on tile** — только filled; chrome `.peek-slot-like--portrait` + filled primary border (`#1976d2` 1px); row gap **8**, `overflow-x: auto`, `flex-wrap: nowrap`, **`justify-content: center`**. Empty omit. Zero filled → omit slots row (no `content.slotEmpty`). **Raise slots:** question region MUST NOT `flex-grow` all free height into a void that pushes slots onto the actions divider; use content-sized / scroll question (`flex: 0 1 auto` or equivalent) + **≈8–12px** air below slots above the actions divider.
4. **Answer text / props** — primary = answer `content`; secondary = answer `description` if any. Prop `filledSlots: { content: string; description?: string }[]`. Parents resolve from loaded `answerCards` — не новый HTTP. Helper `lib/taskFilledSlots.ts` already preferred (DRY).
5. **Compose width** — `min/max-width: min(4 * var(--peek-tile-max-w) + 3 * 8px + horizontal_pad + ≈50px, 92vw)`; убрать fixed 360/400. Pool `repeat(4, …)` gap 8 unchanged. The **+≈50px** is intentional slack over strict 4-tile row (SC-PACK-279 revised).
6. **Compose dots size** — compose-only (`.pack-task-compose-card .peek-diff-dot` / `--compose-diff-dot-size: 20px`); shared base **6px** для game peek.
7. **Tile actions** — Edit/Delete full-width stacked bottom: **`outline` + Material icon** (`edit` / `delete`), `dense` `no-caps`, slim `--pack-card-action-h`, chrome parity with `.pack-task-set-tile__actions` (neutral outline border via `--pack-card-border`; label weight ~500). Delete MAY use `color="negative"` (red). Labels stay `content.editTask` / `content.deleteTask`. **Not** flat text-only.
8. **Question type** — list tile question `font-size` **24px** (same as `.peek-question` / SC-BOARD-54), weight **600**, center, overflow-y auto. Supersedes former 13–15 tile-scaled band.
9. **Compose Save / Cancel** — single full-width primary **unelevated** `no-caps` submit button styled like `.peek-submit-btn` (height `--pack-card-action-h`, pad `0 0.6rem`, font ~0.78rem / 600). Labels stay `content.saveTask` / `content.addTask` (not `game.peekSubmit`). **Remove** Cancel button from UI. Dismiss without save = backdrop click / Esc (`q-dialog` not persistent); keep `@hide` / parent reset. No header close X.
10. **Tests** — Vitest SC-PACK-223/226/238/278/281 (tile), SC-PACK-279/280/276/260 (compose); update asserts for outline actions, 24px question, centered/raised slots, no Cancel, Save chrome, width +50.

**Alternatives rejected:** оставить 300×200; сужать и game peek; общие 20px dots на peek; dashed empty на list tile; question ×3 (~42px); Save label «Ответить»; явный X вместо Cancel.

## Risks / Trade-offs

- [Узкая portrait + 24px вопрос] → Mitigation: scroll в зоне вопроса; не растягивать ширину.
- [Много слотов + center + horizontal scroll] → Mitigation: `justify-content: center` + `overflow-x: auto`; flex-shrink 0 on minis; Vitest SC-PACK-278.
- [Compose +50px vs 4-tile anchor] → Mitigation: document slack in SC-PACK-279; keep `92vw` cap.
- [Shared CSS accidentally enlarges peek dots / changes peek submit] → Mitigation: compose-scoped selectors only; Vitest SC-PACK-280.
- [Removing Cancel] → Mitigation: backdrop/Esc still closes; Vitest SC-PACK-281.

## Migration Plan

Client-only UI. Нет data migration. Rollback = revert tile + compose CSS/markup.

## Open Questions

_(none — explore closed; question size = 24px)_

## Implementation touchpoints

- `src/components/PackTaskTile.vue` — portrait layout; question 24px; slots center + raised air; Edit/Delete → outline+icon; reuse `.peek-diff-*` / `.peek-slot-like--portrait`.
- Parents + `lib/taskFilledSlots.ts` — already filled-only resolve (keep).
- `src/components/PackTaskComposeDialog.vue` — width +≈50; Save → peekSubmit-like chrome; remove Cancel btn; keep hide/cancel emit path for parents.
- `src/css/app.scss` — only if shared helpers needed; prefer scoped compose/tile.
- Vitest: `PackTaskTile.test.ts`, `ContentPackTaskCompose.test.ts` (+ related).
- Skill notes: pack-cards / content / peek — outline actions, 24px question, compose Save/no Cancel / +50 width.

**OpenSpec scenario titles:** для SC-PACK-223 / SC-PACK-226 в delta **сохранять исторические заголовки** из main — менять только тело. Rename ломает validate/archive.

Checklist: `tasks.md`.

## Visual Spec

**Source of truth:** this section. Token reuse from shipped peek/compose / pack-card. No separate mock image. Values marked `approx` MAY land in a small band.

### PackTaskTile (list) — structure

Order top → bottom (no pale mid-card dividers except actions top border):

1. Header (dots left + word badge center) — height 20px
2. Question region (content-sized / scroll; **not** flex-grow void that pins slots to bottom)
3. Filled-slots row (centered; overflow-x auto; nowrap) + air **≈8–12** below before actions
4. Actions (Edit / Delete) when editable — outline + icon stacked

```
+---------------------------+  W 156–180; min-H ≥206; W < H
| ooo          лёгкая       |  header h=20
|      текст задания        |  24px center
|    [mini][mini] -->       |  filled only; centered; gap 8
|         (air)             |  ~8–12 above divider
|---------------------------|
| [edit] Изменить вопрос    |  outline + icon; --pack-card-action-h
| [del]  Удалить            |  outline + icon; MAY negative
+---------------------------+
```

### PackTaskTile — overall / surface

| Token | Spec |
|-------|------|
| Width | `approx` **156–180** CSS px |
| Height | portrait **W &lt; H**; `min-height` **≈206** |
| Border-radius | **12px** |
| Border / bg / fg / shadow | `--pack-card-*` (light/dark as before) |
| Root padding | horizontal **8–12** px; top **8–10** px; bottom **0** |

### PackTaskTile — element sizes / pads / gaps

| Element | Spec |
|---------|------|
| Header | height **20px**; `.peek-diff-header` |
| Dots | **6×6**; gap **2**; fill = difficulty |
| Badge | h **20**; word-only `content.difficultyWord.*` |
| Header → question gap | **8–12** px |
| Question | `font-size` **24px** (parity `.peek-question`); weight **600**; line-height **1.25–1.35**; center; overflow-y **auto**; word-break; min-height `approx` **40–56**; **flex-grow MUST NOT** create a large empty void that pins slots to the actions edge |
| Question → slots gap | **8–10** px |
| Slot mini-card | `--peek-tile-max-w` **43px**; AR **156∶206**; filled primary border `#1976d2` |
| Slots row | flex nowrap; gap **8**; `justify-content: center`; `overflow-x: auto`; minis `flex-shrink: 0` |
| Slots → actions air | **≈8–12** px below slots row (or equivalent padding before divider) |
| Empty / zero filled | omit slot / omit row |
| Actions | outline + icon; height `--pack-card-action-h` **≈30**; font ~0.72–0.78rem weight **500**; icon **18px**; Delete MAY `color="negative"` |
| Icons on body | none except action icons |

### PackTaskTile — alignment

| Element | Align |
|---------|-------|
| Dots | left |
| Badge | center header |
| Question | center |
| Slots row | **center** (horizontal) |
| Slot texts | center |
| Edit / Delete | center (full-width outline) |

### PackTaskTile — copy / i18n

| Surface | Key |
|---------|-----|
| Badge | `content.difficultyWord.{1\|2\|3}` |
| Edit | `content.editTask` |
| Delete | `content.deleteTask` |

### Slot data binding (FE)

`filledSlots: { content; description? }[]` from loaded `answerCards` — no new HTTP. Zero → omit row.

### Compose dialog polish

| Token | Spec |
|-------|------|
| Width | `min(4 * var(--peek-tile-max-w) + 3 * 8px + horizontal_pad + ≈50px, 92vw)` — slack **≈50** over strict 4-tile+pad |
| Dots | compose-only Ø **≈20**; shared base **6×6** for peek |
| Pool | `repeat(4, …)` gap 8 — unchanged |
| Save | full-width primary unelevated no-caps; chrome parity `.peek-submit-btn` (`--pack-card-action-h`, pad `0 0.6rem`, font ~0.78rem / 600); labels `content.saveTask` / `content.addTask` |
| Cancel | **not shown**; dismiss = backdrop / Esc |
| Game peek | width + dots + submit **unchanged** by this change |

### CSS scope rule

Larger dots, +50 width, Save chrome apply **only** under compose (`.pack-task-compose-card`). Shared `.peek-diff-dot` base stays 6px. Tile MAY reuse peek header/slot classes without compose-only enlargements.
