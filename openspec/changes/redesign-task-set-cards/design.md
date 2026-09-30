## Context

See `proposal.md` — Why. Shipped: dedicated `PackTaskSetCardTile`, author/coauthor strip, `#{n}`, colored outline dots, total icon + «Заданий:», short «Снять»/«Вернуть», pale multi-row dividers, lead/label/count grid, slim ~28–32px actions, vertical fill. Catalog still `PackListCardTile` 150×200. Remaining gap vs light/dark mock: **внутренний vertical rhythm** (мало воздуха между title / labels и dividers) и **themed hover** (dark — светло-серая рамка, не brand secondary; без scale; иконки без перекраса).

## Goals / Non-Goals

**Goals:**
- Dedicated task-set summary card chrome on live + editor lists
- Short status badges; outline+icon bottom actions; difficulty dots 1/2/3
- Strip task-set author/coauthor from all content + lobby labels
- Mock + layout polish (hash, colors, short actions, dividers, columns, slim actions, fill)
- **Spacing polish:** larger internal gaps title↔stats / around pale dividers (mock Visual Spec)
- **Hover polish (no scale):** light — dark border + soft lift shadow; dark — light-grey border; icons/dots unchanged
- Keep catalog pack cards and answer/task playing-card tiles unchanged

**Non-Goals:**
- Dropping SQLite `coauthor_labels` / removing `authorDisplayName` from API payloads
- Staff hub card-grid redesign; map author UI
- **Hover scale / enlarge** (mock shows enlarge — product decision: **do not** implement)
- Changing **grid gutter** between cards (`.pack-card-grid` / list wrap)
- Shipping custom badge / total-row SVGs now (names reserved; Material placeholders OK)
- Changing create payload ids or soft-unpublish ACL
- Changing pack/catalog «Снять с публикации» or confirm dialog strings

## Decisions

1. **Separate task-set tile (or strong variant), keep catalog on `PackListCardTile`**
   - Rationale: catalog stays SC-PACK-228 (title+description 150×200); task-set needs denser body. Prefer a dedicated component used from live + editor; catalog keeps current tile.
   - Alternative: overload `PackListCardTile` with many slots — rejected (bloat, risk regressing catalog).

2. **Size ~150–160× taller than 200**
   - Rationale: four stat rows + slim actions + roomier internal rhythm. Width near catalog; resting height from mock ≈ **206 CSS px** at width 156 (see Visual Spec); MAY grow slightly if spacing tokens push content.
   - Alternative: force exact 150×200 — rejected (too cramped).

3. **Author/coauthor: UI-only removal; ACL unchanged**
   - Labels: `taskSetLabel` («Набор заданий #{n}») everywhere (live, editor, staff hub, tasks header, lobby create/list).
   - On save, keep sending `coauthorLabels: []`; do not migrate/drop column.
   - `authorDisplayName` may remain in API/types unused by these labels.

4. **Short badge copy via i18n**
   - Card badges: short keys (СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …) + leading Material placeholders until SVG:
     - `task-set-badge-revise.svg` — temp Material `close`
     - `task-set-badge-pending.svg` — temp Material `schedule`
     - `task-set-badge-unpublished.svg` — temp Material `visibility_off`
     - `task-set-badge-draft.svg` — temp Material `edit_note`
   - Chrome: **soft muted pills** (not Quasar solid `grey`/`warning` fills) —
     `--muted` pale grey bg + dark/light fg; `--pending` soft amber tint + gold/amber ink.
   - Total-row icon: `task-set-card-tasks.svg` later; until then Material `description`. Prefer `currentColor` SVGs.

5. **Actions: Quasar outline + icon + label, full width, slim**
   - Height ≈ **28–32 CSS px** (mock measured ~28–30 for button chrome). Prefer `dense` and/or constrained `min-height`.
   - Multiple actions (Edit + Снять / Вернуть): stacked full-width slim, same ACL as today.
   - Card soft-unpublish: «Снять»; republish: «Вернуть». Pack/catalog long copy + confirms unchanged.
   - Preserve `@click.stop`. Neutral pale outline for Edit / «Снять» / «Вернуть» (no warning/primary tint).

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

10. **Spacing + hover polish (follow-up)**
    - **Internal spacing only** (not grid gutter): increase air between title and stats block, and between each stats label-row and its neighbouring pale dividers — match Visual Spec tokens (`title→stats`, `row↔divider`).
    - **Hover (clickable cards): no `transform: scale` / enlarge.**
      - Light: border → near-black / dark grey; soft lift `box-shadow`.
      - Dark: border → **light grey** (~`#bdbdbd` / `#c0c4c6` product sense); soft lift shadow OK.
      - **Do not** recolor difficulty dots, total icon, badge icons, or action icons on hover (chrome/border/shadow only).
      - Do **not** use brand `--q-secondary` as the dark-theme hover border.
    - Package: **client** — `PackTaskSetCardTile` CSS; `pack-cards.md`.

## Visual Spec (from mock)

Источник: light/dark скрин «Мои наборы заданий».  
**Якорь:** resting card width на макете **127 px** → CSS **156 px** (scale **×1.228**). Остальные CSS px = mock_px × 1.228, округление до целых.  
Hover-карточка на макете визуально крупнее — **enlarge не переносим** (Decision 10); для resting размеров берём **не**-hover карточку (левый столбец light / resting dark).

### Overall

| Token | Mock px | CSS px | Notes |
|-------|---------|--------|-------|
| card.width | 127 | **156** | якорь |
| card.height (resting) | 168 | **206** | min-height ≈ 206; MAY +few if spacing grows |
| card.radius | ~10–12 | **12** | established product radius |
| card.pad.top | ~7 | **8–10** | status band |
| card.pad.x (body) | ~9–11 | **10–12** | content inset |
| card.pad.bottom | ~5–8 | **6–8** | below actions |
| grid.gap (cards) | ~15–16 | ~20 | **out of scope** — не менять list gutter |

### Размеры элементов

| Элемент | W (CSS) | H / min-H (CSS) | Pad / gap (CSS) | Notes |
|---------|---------|-----------------|-----------------|-------|
| card (overall) | 156 | min-height **206** | radius 12 | resting |
| status band | full | min ~22 (badge ~10–12 content) | pad-top 8–10 | center badge; empty OK |
| badge pill | auto | ~18–22 | pad ~2×6; gap icon–text ~2–3 | icon+label |
| title | full content | ~14–16 (1–2 lines) | **margin-bottom → stats: 12–16** | center; was too tight (~5) |
| total-row | full | ~14–16 | row pad-y **6–8** | icon 14 in lead 22 |
| diff-row ×3 | full | ~14–16 | row pad-y **6–8** | dots 6Ø gap 2 |
| divider | full content | **1** | **air row↔divider: 8–12** total (split above+below) | pale; after total; between diffs; above actions |
| action button | full | **28–32** | pad-x ~6–8 | slim outline; stacked if many |
| lead column | **22** | — | — | 3×6 dots + 2×2 gaps |
| total icon | 14×14 | — | centered in lead | Material `description` |
| difficulty dots | Ø **6** | — | gap **2** | filled green/amber/red; empty outline |

### Структура + copy + align

1. Badge (center) → 2. Title center → 3. Total → **divider** → 4. Diff1 → **divider** → 5. Diff2 → **divider** → 6. Diff3 → **divider** → 7. Actions  
Labels left shared edge; counts right shared edge; button label center.

| Элемент | Copy | Align H |
|---------|------|---------|
| Badge | ДОРАБОТАТЬ / СНЯТО / НА ПРОВЕРКЕ / … | center |
| Title | Набор заданий #{n} | center |
| Stats labels | Заданий: / Лёгкие: / Средние: / Сложные: | left |
| Counts | numbers | right |
| Actions | Редактировать / Снять / Вернуть | center |

### Цвета

#### Light

| Token | Value |
|-------|-------|
| card.bg | `#ffffff` |
| card.border.rest | soft grey ~`#e1e3e6` / rgba(0,0,0,0.12–0.14) |
| card.border.hover | **near-black / dark grey** (stronger than rest) |
| card.shadow.rest | soft `0 1px 3px` rgba(0,0,0,~0.08) |
| card.shadow.hover | soft **lift** (larger blur/y-offset; **no** scale) |
| divider | pale ~rgba(0,0,0,0.08) |
| dots 1/2/3 | green / amber / red |

#### Dark

| Token | Value |
|-------|-------|
| card.bg | `#2a2a2a` |
| card.border.rest | soft light ~rgba(255,255,255,0.22) |
| card.border.hover | **light grey** ~`#bdbdbd` / `#c0c4c6` (mock sampled ~186–208 RGB grey) — **not** `--q-secondary` |
| card.shadow.rest / hover | soft dark lift; hover may strengthen |
| divider | pale ~rgba(255,255,255,0.12) |
| dots 1/2/3 | green / amber / red (unchanged on hover) |

### Missing icons

| ID | Место | Suggested file | Temp Material | Блокер apply? |
|----|-------|----------------|---------------|---------------|
| I1 | total-row | `task-set-card-tasks.svg` | `description` | нет |
| I2 | badge revise | `task-set-badge-revise.svg` | `close` | нет |
| I3 | badge pending | `task-set-badge-pending.svg` | `schedule` | нет |
| I4 | badge unpublished | `task-set-badge-unpublished.svg` | `visibility_off` | нет |
| I5 | badge draft | `task-set-badge-draft.svg` | `edit_note` | нет |

### Hover / focus

| Target | Theme | Property | Default | Hover | Notes |
|--------|-------|----------|---------|-------|-------|
| card | light | border-color | soft grey | **dark / near-black** | |
| card | light | shadow | soft rest | **lift** | |
| card | light | scale | none | **none** | mock enlarge ignored |
| card | dark | border-color | soft light | **light grey** | Decision 10 |
| card | dark | shadow | soft rest | lift OK | |
| card | dark | scale | none | **none** | |
| icons / dots / badge icons | both | color | semantic | **unchanged** | «всё кроме иконок» = chrome only |
| action outline | both | — | neutral | may track card border slightly | optional |

**Muted / soft-unpublished:** lower opacity / muted fg — not hover.

## Risks / Trade-offs

- [Cramped height on small screens] → Mitigation: allow taller card; clamp title; verify light/dark.
- [Tests assert tight spacing / secondary hover] → Mitigation: update `PackTaskSetCardTile` vitest (SC-PACK-245/246).
- [Shared `PackListCardTile` accidental catalog change] → Mitigation: do not change catalog props/layout.
- [Short «Снять» confused with pack unpublish] → Mitigation: card-only i18n keys.
- [Mock enlarge vs no-scale] → Mitigation: documented Decision 10; verify-mock must not flag missing scale as Violation.

## Migration Plan

1. Client-only deploy of spacing + hover polish on existing tile.
2. Rollback: revert client.
3. No DB migration.
4. Later: named SVG assets for badge / total icon.

## Open Questions

(нет — D1 no scale; D2 internal spacing only / grid gap untouched)
