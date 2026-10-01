## Context

See `proposal.md` — Why. Shipped: dedicated `PackTaskSetCardTile`, author/coauthor strip, `#{n}`, colored outline dots, custom SVG total/badge icons + «Заданий:», short «Снять»/«Вернуть», pale multi-row dividers, lead/label/count grid, slim ~28–32px actions, vertical fill, spacing retune (status→title ~6–8; title→stats ~7; divider air ~4–6), themed hover without scale. Catalog still `PackListCardTile` 150×200. No remaining follow-up in this change.

## Goals / Non-Goals

**Goals:**
- Dedicated task-set summary card chrome on live + editor lists
- Short status badges; outline+icon bottom actions; difficulty dots 1/2/3
- Strip task-set author/coauthor from all content + lobby labels
- Mock + layout + spacing/hover polish + **spacing retune** + **custom SVG icons** (shipped)
- Keep catalog pack cards and answer/task playing-card tiles unchanged

**Non-Goals:**
- Dropping SQLite `coauthor_labels` / removing `authorDisplayName` from API payloads
- Staff hub card-grid redesign; map author UI
- **Hover scale / enlarge** (mock shows enlarge — product decision: **do not** implement)
- Changing **grid gutter** between cards (`.pack-card-grid` / list wrap)
- Custom SVG for **action** buttons (Edit / Снять / Вернуть stay Quasar Material)
- PNG icon variants
- Changing create payload ids or soft-unpublish ACL
- Changing pack/catalog «Снять с публикации» or confirm dialog strings

## Decisions

1. **Separate task-set tile (or strong variant), keep catalog on `PackListCardTile`**
   - Rationale: catalog stays SC-PACK-228 (title+description 150×200); task-set needs denser body. Prefer a dedicated component used from live + editor; catalog keeps current tile.
   - Alternative: overload `PackListCardTile` with many slots — rejected (bloat, risk regressing catalog).

2. **Size ~150–160× taller than 200**
   - Rationale: four stat rows + slim actions + internal rhythm. Width near catalog; resting height from mock ≈ **206 CSS px** at width 156 (see Visual Spec); MAY shrink slightly if spacing retune packs content.
   - Alternative: force exact 150×200 — rejected (too cramped).

3. **Author/coauthor: UI-only removal; ACL unchanged**
   - Labels: `taskSetLabel` («Набор заданий #{n}») everywhere (live, editor, staff hub, tasks header, lobby create/list).
   - On save, keep sending `coauthorLabels: []`; do not migrate/drop column.
   - `authorDisplayName` may remain in API/types unused by these labels.

4. **Short badge copy via i18n + soft muted pills**
   - Card badges: short keys (СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / ЧЕРНОВИК) + leading icons.
   - Chrome: **soft muted pills** (not Quasar solid `grey`/`warning` fills) —
     `--muted` pale grey bg + dark/light fg; `--pending` soft amber tint + gold/amber ink.
   - **Assets wired** (client): `src/assets/content/task-set-badge-{revise,pending,unpublished,draft}.svg`, `task-set-card-tasks.svg` via mask/`currentColor` (Decision 12).

5. **Actions: Quasar outline + icon + label, full width, slim**
   - Height ≈ **28–32 CSS px**. Prefer `dense` and/or constrained `min-height`.
   - Multiple actions (Edit + Снять / Вернуть): stacked full-width slim, same ACL as today.
   - Card soft-unpublish: «Снять»; republish: «Вернуть». Pack/catalog long copy + confirms unchanged.
   - Preserve `@click.stop`. Neutral pale outline for Edit / «Снять» / «Вернуть» (no warning/primary tint).
   - Action icons stay Quasar Material (`edit`, `visibility_off`, …) — no custom SVG for actions.

6. **Lobby metadata**
   - Client display of `taskSetLabels` MUST not surface author strings; ordinal with `#` / pack title. Server shape can stay.

7. **Server**
   - Optional: force `coauthorLabels` to `[]` on persist. First pass skipped as UI-only.

8. **Mock polish chrome (shipped)**
   - Total row: leading icon + «Заданий:» + count; colored outline dots; short card actions; `#{n}`.

9. **Layout polish (shipped)**
   - Pale dividers after total; between each difficulty row; above actions.
   - Columns: lead = 3-dot width; icon centered; labels shared left edge; counts right.
   - Vertical fill; reserved top status band when no badge.

10. **Spacing + hover polish (shipped)**
    - First pass tokens: `--pack-ts-title-gap: 14px`; `--pack-ts-divider-air: 5px` (10 total); themed hover light `#212121` / dark `#bdbdbd`; no scale; icons unchanged on hover.
    - Package: **client** — `PackTaskSetCardTile` CSS; `pack-cards.md`.

11. **Spacing retune (shipped)**
    - **status→title:** ~**6–8 CSS px** (`padding-bottom` on `__status` and/or `margin-top` on title).
    - **title→stats:** `--pack-ts-title-gap` **7** (was 14).
    - **divider air:** `--pack-ts-divider-air` **2–3** (total air ~**4–6**).
    - Left `--pack-ts-row-pad-y: 6`; did **not** change `.pack-card-grid` / list gutter.
    - Package: **client** — CSS tokens + vitest SC-PACK-247; `pack-cards.md`.

12. **Wire custom SVG icons (shipped)**
    - Assets under `{client}/src/assets/content/` (I1–I5) wired via CSS **`mask-image` + `background: currentColor`** (theme ink from Visual Spec).
    - Sizes: badge icon **~12** CSS px; total-row icon **14** CSS px.
    - Map: revise→ДОРАБОТАТЬ; pending→НА ПРОВЕРКЕ; unpublished→СНЯТО; draft→ЧЕРНОВИК; tasks→«Заданий:».
    - Soft muted / pending hex/rgba from Visual Spec `### Цвета` (not Quasar solid fills).
    - Package: **client** — `PackTaskSetCardTile` + live/editor badge slots; vitest SC-PACK-248; `pack-cards.md`.

Макеты: `openspec/changes/redesign-task-set-cards/assets/task-set-cards-mock.png` (+ `task-set-cards-mock-v2.jpg`).

## Visual Spec (from mock)

Источник: light/dark скрин «Мои наборы заданий».  
**Якорь:** resting card width на макете **127 px** → CSS **156 px** (scale **×1.228**). Остальные CSS px = mock_px × 1.228, округление до целых.  
Hover-карточка на макете визуально крупнее — **enlarge не переносим** (Decision 10); для resting размеров берём **не**-hover карточку (левый столбец light / resting dark).

### Overall

| Token | Mock px | CSS px | Notes |
|-------|---------|--------|-------|
| card.width | 127 | **156** | якорь |
| card.height (resting) | 168 | **206** | min-height ≈ 206; MAY −few after retune |
| card.radius | ~10–12 | **12** | established product radius |
| card.pad.top | ~7 | **8–10** | status band |
| card.pad.x (body) | ~9–11 | **10–12** | content inset |
| card.pad.bottom | ~5–8 | **6–8** | below actions |
| grid.gap (cards) | ~15–16 | ~20 | **out of scope** — не менять list gutter |

### Размеры элементов

| Элемент | W (CSS) | H / min-H (CSS) | Pad / gap (CSS) | Notes |
|---------|---------|-----------------|-----------------|-------|
| card (overall) | 156 | min-height **206** | radius 12 | resting |
| status band | full | min ~22 (badge ~10–12 content) | pad-top 8–10; **pad-bottom / gap→title 6–8** | center badge; empty OK |
| badge pill | auto | ~18–22 | pad ~2×6; gap icon–text ~2–3 | icon+label |
| title | full content | ~14–16 (1–2 lines) | **margin-bottom → stats: ~7** (retune; was 12–16) | center |
| total-row | full | ~14–16 | row pad-y **6–8** | icon 14 in lead 22 |
| diff-row ×3 | full | ~14–16 | row pad-y **6–8** | dots 6Ø gap 2 |
| divider | full content | **1** | **air row↔divider: ~4–6** total (retune; was 8–12) | pale; after total; between diffs; above actions |
| action button | full | **28–32** | pad-x ~6–8 | slim outline; stacked if many |
| lead column | **22** | — | — | 3×6 dots + 2×2 gaps |
| total icon | 14×14 | — | centered in lead | custom SVG |
| difficulty dots | Ø **6** | — | gap **2** | filled green/amber/red; empty outline |

### Структура + copy + align

1. Badge (center) → 2. Title center → 3. Total → **divider** → 4. Diff1 → **divider** → 5. Diff2 → **divider** → 6. Diff3 → **divider** → 7. Actions  
Labels left shared edge; counts right shared edge; button label center.

| Элемент | Copy | Align H |
|---------|------|---------|
| Badge | ДОРАБОТАТЬ / СНЯТО / НА ПРОВЕРКЕ / ЧЕРНОВИК / … | center |
| Title | Набор заданий #{n} | center |
| Stats labels | Заданий: / Лёгкие: / Средние: / Сложные: | left |
| Counts | numbers | right |
| Actions | Редактировать / Снять / Вернуть | center |

### Цвета

Канон для apply / verify-mock (hex/rgba). Icon ink = `currentColor` родителя (см. Icon ink). Soft muted pills — **не** Quasar solid `grey`/`warning`.

#### Light

| Token | Value |
|-------|-------|
| card.bg | `#ffffff` |
| card.border.rest | `rgba(0,0,0,0.14)` (~`#e1e3e6`) |
| card.border.hover | `#212121` |
| card.shadow.rest | `0 1px 3px rgba(0,0,0,0.08)` |
| card.shadow.hover | `0 4px 12px rgba(0,0,0,0.14)` (lift; **no** scale) |
| title.fg | `rgba(0,0,0,0.87)` |
| stats.label.fg / stats.icon.ink | `rgba(0,0,0,0.7)` (`--pack-ts-muted`) |
| stats.count.fg | `rgba(0,0,0,0.87)` (`--pack-ts-fg` / title.fg; weight 600) |
| divider | `rgba(0,0,0,0.08)` |
| dots.filled.1 / .2 / .3 | `#43a047` / `#f9a825` / `#e53935` |
| dots.empty | transparent fill + `1px` border = filled color |
| badge.muted.bg (ДОРАБОТАТЬ / СНЯТО / ЧЕРНОВИК) | `rgba(0,0,0,0.06)` |
| badge.muted.fg / .icon.ink | `rgba(0,0,0,0.72)` |
| badge.pending.bg (НА ПРОВЕРКЕ) | `rgba(249,168,37,0.16)` |
| badge.pending.fg / .icon.ink | `#f9a825` |
| action.outline | `rgba(0,0,0,0.14)` (= card.border.rest) |
| action.label.fg / .icon.ink | `rgba(0,0,0,0.87)` (= title.fg) |
| card.muted.opacity (СНЯТО soft-unpublish) | `0.72` |
| card.muted.border-style | `dashed` (mock; rest cards stay `solid`) |

#### Dark

| Token | Value |
|-------|-------|
| card.bg | `#2a2a2a` |
| card.border.rest | `rgba(255,255,255,0.22)` |
| card.border.hover | `#bdbdbd` (**not** `--q-secondary`) |
| card.shadow.rest | `0 1px 3px rgba(0,0,0,0.35)` |
| card.shadow.hover | `0 4px 14px rgba(0,0,0,0.5)` |
| title.fg | `rgba(255,255,255,0.92)` |
| stats.label.fg / stats.icon.ink | `rgba(255,255,255,0.78)` |
| stats.count.fg | `rgba(255,255,255,0.92)` (`--pack-ts-fg` / title.fg; weight 600) |
| divider | `rgba(255,255,255,0.12)` |
| dots.filled.1 / .2 / .3 | `#66bb6a` / `#ffca28` / `#ef5350` |
| dots.empty | transparent fill + `1px` border = filled color |
| badge.muted.bg | `rgba(255,255,255,0.12)` |
| badge.muted.fg / .icon.ink | `rgba(255,255,255,0.82)` |
| badge.pending.bg | `rgba(255,193,7,0.14)` |
| badge.pending.fg / .icon.ink | `#ffc107` |
| action.outline | `rgba(255,255,255,0.22)` |
| action.label.fg / .icon.ink | `rgba(255,255,255,0.92)` |
| card.muted.opacity | `0.72` |
| card.muted.border-style | `dashed` |

### Typography (from mock / shipped rem @ 16px root)

| Role | font-size | weight | line-height | Notes |
|------|-----------|--------|-------------|-------|
| badge | `0.62rem` (~10) | 600 | 1.2 | uppercase short copy |
| title | `0.78rem` (~12.5) | 700 | 1.25 | center; clamp 2 lines |
| stats label / count | `0.68rem` (~11) | 400 / **600** count | 1.2 | label muted; count title.fg |
| action | `0.72rem` (~11.5) | Quasar default | — | slim outline |

### Icon ink / sizes

| Место | Size CSS px | Ink | Asset |
|-------|-------------|-----|-------|
| total-row («Заданий:») | **14×14** | `stats.icon.ink` via `currentColor` / CSS mask | `task-set-card-tasks.svg` |
| badge ДОРАБОТАТЬ | **~12** | `badge.muted.icon.ink` | `task-set-badge-revise.svg` |
| badge НА ПРОВЕРКЕ | **~12** | `badge.pending.icon.ink` | `task-set-badge-pending.svg` |
| badge СНЯТО | **~12** | `badge.muted.icon.ink` | `task-set-badge-unpublished.svg` |
| badge ЧЕРНОВИК | **~12** | `badge.muted.icon.ink` | `task-set-badge-draft.svg` |
| action Edit / Снять / Вернуть | Quasar dense (~18) | `action.icon.ink` | Material (no custom SVG) |

Assets: path-only SVGs (no baked fill); themed via mask/`currentColor` (Decision 12). Иконки **не** перекрашивать на card hover (только border/shadow chrome).

### Custom icons (wired)

| ID | Место | File under `src/assets/content/` | Copy рядом |
|----|-------|----------------------------------|------------|
| I1 | total-row | `task-set-card-tasks.svg` | Заданий: |
| I2 | badge revise | `task-set-badge-revise.svg` | ДОРАБОТАТЬ |
| I3 | badge pending | `task-set-badge-pending.svg` | НА ПРОВЕРКЕ |
| I4 | badge unpublished | `task-set-badge-unpublished.svg` | СНЯТО |
| I5 | badge draft | `task-set-badge-draft.svg` | ЧЕРНОВИК |

### Hover / focus

| Target | Theme | Property | Default | Hover | Notes |
|--------|-------|----------|---------|-------|-------|
| card | light | border-color | soft grey | **dark / near-black** | |
| card | light | shadow | soft rest | **lift** | |
| card | light | scale | none | **none** | mock enlarge ignored |
| card | dark | border-color | soft light | **light grey** | Decision 10 |
| card | dark | shadow | soft rest | lift OK | |
| card | dark | scale | none | **none** | |
| icons / dots / badge icons | both | color | semantic | **unchanged** | chrome only |
| action outline | both | — | neutral | may track card border slightly | optional |

**Muted / soft-unpublished:** lower opacity / muted fg — not hover.

## Risks / Trade-offs

- [Cramped height after retune] → Mitigation: keep min-height flexible; verify light/dark.
- [Tests assert old 14/10 spacing tokens] → Mitigation: update vitest SC-PACK-245/247.
- [SVG black default under `<img>`] → Mitigation: mask/`currentColor` (Decision 12); not plain `<img>` without theming.
- [Shared `PackListCardTile` accidental catalog change] → Mitigation: do not change catalog props/layout.
- [Short «Снять» confused with pack unpublish] → Mitigation: card-only i18n keys.
- [Mock enlarge vs no-scale] → Mitigation: documented Decision 10; verify-mock must not flag missing scale as Violation.

## Migration Plan

1. Client-only change shipped (no DB migration).
2. Rollback: revert client (assets may stay).

## Open Questions

(нет)
