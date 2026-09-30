## Context

See `proposal.md` — Why. Today live pack and cards editor reuse `PackListCardTile` (150×200) for task sets; stats are a cramped caption; labels use `taskSetLabelFrom` with `authorDisplayName` and optional `coauthorLabels`. Catalog packs share the same tile component. Lobby create/list surfaces author on set options and room metadata. Target chrome is the agreed light/dark mock (no hover scale).

## Goals / Non-Goals

**Goals:**
- Dedicated task-set summary card chrome on live + editor lists
- Short status badges; outline+icon bottom actions; difficulty dots 1/2/3
- Strip task-set author/coauthor from all content + lobby labels
- Keep catalog pack cards and answer/task playing-card tiles unchanged

**Non-Goals:**
- Dropping SQLite `coauthor_labels` / removing `authorDisplayName` from API payloads
- Staff hub card-grid redesign; map author UI; hover enlarge
- Changing create payload ids or soft-unpublish ACL

## Decisions

1. **Separate task-set tile (or strong variant), keep catalog on `PackListCardTile`**
   - Rationale: catalog stays SC-PACK-228 (title+description 150×200); task-set needs denser body. Prefer a dedicated component (e.g. task-set list tile) used from live + editor; catalog keeps current tile.
   - Alternative: overload `PackListCardTile` with many slots — rejected (bloat, risk regressing catalog).

2. **Size ~150–160× taller than 200**
   - Rationale: mock ~130×170 at screenshot scale; four stat rows need breathing room. Width near catalog; height tuned visually (~220–240) without locking a second magic pair in answer/task tiles.
   - Alternative: force exact 150×200 — rejected (too cramped for mock content).

3. **Author/coauthor: UI-only removal; ACL unchanged**
   - Switch labels to `taskSetLabel` («Набор заданий {n}») on live, editor, staff hub headings, tasks page header, lobby create options, lobby room list metadata display.
   - On save, keep sending `coauthorLabels: []`; do not migrate/drop column.
   - `authorDisplayName` may remain in API/types unused by these labels.
   - Alternative: strip fields from server payloads now — deferred (out of scope).

4. **Short badge copy via i18n**
   - Card badges: short keys (СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …). Page subtitles may keep longer three-phase phrases where already required; cards MUST use short forms.

5. **Actions: Quasar outline + icon + label, full width**
   - Match mock; preserve `@click.stop` so body open ≠ action. No hover scale CSS.

6. **Lobby metadata**
   - Client display of `taskSetLabels` MUST not surface author strings; prefer ordinal/pack title. Server metadata shape can stay; client stops rendering author.

7. **Server**
   - Optional: force `coauthorLabels` to `[]` on persist. No new HTTP. No schema migration.

## Risks / Trade-offs

- [Cramped height on small screens] → Mitigation: allow taller card; clamp title; verify light/dark.
- [Tests still assert «от {name}» / SC-PACK-135 old text] → Mitigation: update vitest (ContentFollowUp5, list-cards, LobbyCreateWire, AuthorEditTake) with scenarios.
- [Shared `PackListCardTile` accidental catalog change] → Mitigation: do not change catalog props/layout in this change.
- [Staff hub still shows old From-label until touched] → Mitigation: include staff heading + tasks header in author-strip pass (no card-grid there).

## Migration Plan

1. Client-only deploy of tile + i18n + lobby label changes.
2. Rollback: revert client; server optional `[]` persist is harmless.
3. No DB migration.

## Open Questions

(нет — D1 и product choices закрыты в explore)
