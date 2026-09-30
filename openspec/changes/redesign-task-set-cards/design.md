## Context

See `proposal.md` — Why. First passes shipped dedicated `PackTaskSetCardTile`, author/coauthor strip, `#{n}`, colored outline dots, total icon + «Заданий:», short «Снять»/«Вернуть». Catalog still `PackListCardTile` 150×200. Remaining gap vs light/dark mock: layout rhythm — pale dividers between **every** stats row and above actions; lead/label/count column grid; vertical fill; slim action height (~28–32px vs default Quasar ~36–40).

## Goals / Non-Goals

**Goals:**
- Dedicated task-set summary card chrome on live + editor lists
- Short status badges; outline+icon bottom actions; difficulty dots 1/2/3
- Strip task-set author/coauthor from all content + lobby labels
- Mock polish: hash, colors, short card actions, temporary total icon
- Layout polish: pale multi-row dividers, column alignment, vertical fill, slim actions
- Keep catalog pack cards and answer/task playing-card tiles unchanged

**Non-Goals:**
- Dropping SQLite `coauthor_labels` / removing `authorDisplayName` from API payloads
- Staff hub card-grid redesign; map author UI; hover enlarge / new hover polish beyond existing clickable border
- Shipping custom badge / total-row SVGs now (names reserved)
- Changing create payload ids or soft-unpublish ACL
- Changing pack/catalog «Снять с публикации» or confirm dialog strings
- Neutralizing warning/primary tint on card actions (deferred)

## Decisions

1. **Separate task-set tile (or strong variant), keep catalog on `PackListCardTile`**
   - Rationale: catalog stays SC-PACK-228 (title+description 150×200); task-set needs denser body. Prefer a dedicated component used from live + editor; catalog keeps current tile.
   - Alternative: overload `PackListCardTile` with many slots — rejected (bloat, risk regressing catalog).

2. **Size ~150–160× taller than 200**
   - Rationale: four stat rows + slim actions need breathing room. Width near catalog; height tuned visually (~220–240) without locking a second magic pair in answer/task tiles.
   - Alternative: force exact 150×200 — rejected (too cramped).

3. **Author/coauthor: UI-only removal; ACL unchanged**
   - Labels: `taskSetLabel` («Набор заданий #{n}») everywhere (live, editor, staff hub, tasks header, lobby create/list).
   - On save, keep sending `coauthorLabels: []`; do not migrate/drop column.
   - `authorDisplayName` may remain in API/types unused by these labels.

4. **Short badge copy via i18n**
   - Card badges: short keys (СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …).
   - Badge leading icons: deferred; reserved under `src/assets/content/`:
     - `task-set-badge-revise.svg` — temp Material `close`
     - `task-set-badge-pending.svg` — temp Material `schedule`
     - `task-set-badge-unpublished.svg` — temp Material `visibility_off`
     - `task-set-badge-draft.svg` — temp Material `edit_note`
   - Total-row icon: `task-set-card-tasks.svg` later; until then Material `description`. Prefer `currentColor` SVGs.

5. **Actions: Quasar outline + icon + label, full width, slim**
   - Height ≈ **28–32 CSS px** (mock; ~one stats-row). Prefer `dense` and/or constrained `min-height` — default `q-btn` (~36–40) is too tall.
   - Multiple actions (Edit + Снять / Вернуть): stacked full-width slim, same ACL as today.
   - Card soft-unpublish: «Снять»; republish: «Вернуть». Pack/catalog long copy + confirms unchanged.
   - Preserve `@click.stop`. No hover scale; keep existing clickable border only.

6. **Lobby metadata**
   - Client display of `taskSetLabels` MUST not surface author strings; ordinal with `#` / pack title. Server shape can stay.

7. **Server**
   - Optional: force `coauthorLabels` to `[]` on persist. First pass skipped as UI-only.

8. **Mock polish chrome (shipped)**
   - Total row: leading icon + «Заданий:» + count; colored outline dots; short card actions; `#{n}`.

9. **Layout polish (follow-up)**
   - **Dividers (pale / low-contrast):** after total; **between** each difficulty row (Лёгкие↔Средние↔Сложные); **and** above actions. Not only two section dividers.
   - **Columns:** lead column width = width of three-dot group; total-row icon centered in that lead width; labels («Заданий:» / «Лёгкие:» / …) share one left edge; counts right-aligned.
   - **Vertical fill:** badge → title → stats → actions distribute over card height; if no status badge, empty space only in the reserved top status band (do not collapse rhythm).
   - Package: **client** — `PackTaskSetCardTile` + live/editor action `q-btn` sizing; `pack-cards.md`.

## Visual Spec (from mock)

Approx CSS px; anchor card width ~150–160.

| Token | Value | Notes |
|-------|-------|-------|
| card.width | 150–160 | ~156 today |
| card.min-height | ~220–240 | taller than catalog 200 |
| card.radius | ~12 | |
| status.top | reserved band | empty OK when no badge |
| stats.columns | lead \| label \| count | lead = 3-dot width; icon centered in lead |
| dots | 5–6 Ø, gap ~2 | filled = difficulty; 1 green / 2 amber / 3 red; empty = outline |
| divider | 1px pale | after total; between every diff row; above actions |
| action.height | ~28–32 | slim; stacked if multiple |
| hover | border/ring only | **no** scale |

Light: card `#fff`, soft border, pale grey dividers. Dark: card `#2a2a2a`, soft light border, pale dividers on dark.

## Risks / Trade-offs

- [Cramped height on small screens] → Mitigation: allow taller card; clamp title; verify light/dark.
- [Tests assert old divider count / button density] → Mitigation: update `PackTaskSetCardTile` vitest (SC-PACK-239/244).
- [Shared `PackListCardTile` accidental catalog change] → Mitigation: do not change catalog props/layout.
- [Short «Снять» confused with pack unpublish] → Mitigation: card-only i18n keys.

## Migration Plan

1. Client-only deploy of layout polish on existing tile.
2. Rollback: revert client.
3. No DB migration.
4. Later: named SVG assets for badge / total icon.

## Open Questions

(нет — explore closed: pale dividers between all stats rows + above actions; column grid; vertical fill; slim ~28–32px; multi-actions stacked as today; no scale)
