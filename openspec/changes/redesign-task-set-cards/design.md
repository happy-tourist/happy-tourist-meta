## Context

See `proposal.md` — Why. First pass shipped dedicated `PackTaskSetCardTile` on live + editor, author/coauthor strip, short badges, outline actions. Catalog still `PackListCardTile` 150×200. Gap vs agreed light/dark mock: no `#` in ordinal, monochrome dots, no total-row icon / «Заданий:», missing mid divider, card actions still use long pack unpublish / republish copy.

## Goals / Non-Goals

**Goals:**
- Dedicated task-set summary card chrome on live + editor lists
- Short status badges; outline+icon bottom actions; difficulty dots 1/2/3
- Strip task-set author/coauthor from all content + lobby labels
- Align remaining visual chrome to mock (hash, colors, dividers, short card actions, temporary total icon)
- Keep catalog pack cards and answer/task playing-card tiles unchanged

**Non-Goals:**
- Dropping SQLite `coauthor_labels` / removing `authorDisplayName` from API payloads
- Staff hub card-grid redesign; map author UI; hover enlarge / new hover polish
- Shipping custom badge SVGs now (names reserved for later)
- Changing create payload ids or soft-unpublish ACL
- Changing pack/catalog «Снять с публикации» or confirm dialog strings

## Decisions

1. **Separate task-set tile (or strong variant), keep catalog on `PackListCardTile`**
   - Rationale: catalog stays SC-PACK-228 (title+description 150×200); task-set needs denser body. Prefer a dedicated component (e.g. task-set list tile) used from live + editor; catalog keeps current tile.
   - Alternative: overload `PackListCardTile` with many slots — rejected (bloat, risk regressing catalog).

2. **Size ~150–160× taller than 200**
   - Rationale: mock ~130×170 at screenshot scale; four stat rows need breathing room. Width near catalog; height tuned visually (~220–240) without locking a second magic pair in answer/task tiles.
   - Alternative: force exact 150×200 — rejected (too cramped for mock content).

3. **Author/coauthor: UI-only removal; ACL unchanged**
   - Switch labels to `taskSetLabel` («Набор заданий #{n}») on live, editor, staff hub headings, tasks page header, lobby create options, lobby room list metadata display — **one string everywhere**.
   - On save, keep sending `coauthorLabels: []`; do not migrate/drop column.
   - `authorDisplayName` may remain in API/types unused by these labels.
   - Alternative: strip fields from server payloads now — deferred (out of scope).

4. **Short badge copy via i18n**
   - Card badges: short keys (СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …). Page subtitles may keep longer three-phase phrases where already required; cards MUST use short forms.
   - Badge leading icons: deferred; reserved asset names (client, e.g. under `src/assets/content/`):
     - `task-set-badge-revise.svg` (ДОРАБОТАТЬ / needs_revision) — temp Material `close`
     - `task-set-badge-pending.svg` (НА ПРОВЕРКЕ / pending) — temp Material `schedule`
     - `task-set-badge-unpublished.svg` (СНЯТО) — temp Material `visibility_off`
     - `task-set-badge-draft.svg` (ЧЕРНОВИК, if shown) — temp Material `edit_note`
   - Total-row document icon: `task-set-card-tasks.svg` later; until then Material `description` (or equivalent). Prefer `currentColor` SVGs.

5. **Actions: Quasar outline + icon + label, full width**
   - Match mock sizing (not overly `dense`); preserve `@click.stop` so body open ≠ action.
   - Card soft-unpublish label: «Снять» (dedicated i18n key). Card republish: «Вернуть».
   - Pack/catalog `content.unpublish` «Снять с публикации» and confirm dialogs unchanged.
   - No hover scale CSS; **no new hover-polish** in this follow-up (leave existing clickable border as-is).

6. **Lobby metadata**
   - Client display of `taskSetLabels` MUST not surface author strings; prefer ordinal with `#` / pack title. Server metadata shape can stay; client stops rendering author.

7. **Server**
   - Optional: force `coauthorLabels` to `[]` on persist. No new HTTP. No schema migration. First pass skipped as UI-only.

8. **Mock polish chrome (follow-up)**
   - Total row: leading icon + «Заданий:» + count; horizontal divider under total; second divider already above actions.
   - Difficulty dots: filled count = difficulty; filled colors difficulty 1 green / 2 amber / 3 red; unfilled = outline rings (not solid muted).
   - Difficulty row labels: «Лёгкие:» / «Средние:» / «Сложные:» (capitalized + colon).
   - Package: **client** — `PackTaskSetCardTile`, i18n, live/editor action labels; skills `pack-cards.md` (+ related notes).

## Risks / Trade-offs

- [Cramped height on small screens] → Mitigation: allow taller card; clamp title; verify light/dark.
- [Tests still assert old «Набор заданий {n}» / long unpublish] → Mitigation: update vitest (tile, list-cards, FollowUp, LobbyCreateWire, AuthorEditTake).
- [Shared `PackListCardTile` accidental catalog change] → Mitigation: do not change catalog props/layout in this change.
- [Short «Снять» confused with pack unpublish] → Mitigation: card-only i18n keys; pack surfaces keep long copy.

## Migration Plan

1. Client-only deploy of tile + i18n + lobby label changes (+ polish pass).
2. Rollback: revert client; server optional `[]` persist is harmless.
3. No DB migration.
4. Later: drop in named SVG assets and swap Material placeholders.

## Open Questions

(нет — explore decisions closed: `#{n}` everywhere; no new hover; outline empty dots; card «Снять»/«Вернуть»; confirms unchanged; badge SVGs later)
