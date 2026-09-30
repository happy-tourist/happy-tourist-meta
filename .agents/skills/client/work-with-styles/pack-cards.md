# Pack / map card CSS

Read with the [core styles skill](SKILL.md). Contracts: `work-with-stores/content.md`.

## Content pack cascade yellow (`ContentPackEditorPage` / `ContentPackTasksPage`)

Class `cascade-gap-outline` marks cleared answer slots after cascade normalize.
**Same visible outline on both pages** (SC-PACK-126 / D10):

```css
.cascade-gap-outline {
  outline: 2px solid var(--q-warning);
  outline-offset: -2px;
}
```

Do not use `bg-warning` row fill. Applying the class without this scoped CSS
is a hard defect. Contracts: meta `work-with-stores/content.md`.

## Pack playing-card grid (`app.scss` + tile components)

Answer/task lists use global wrap class `.pack-card-grid` (`display: flex;
flex-wrap: wrap; gap: 0.75rem`) in `src/css/app.scss`. Tile chrome lives in
scoped styles on `PackAnswerCardTile.vue` / `PackTaskTile.vue` (fixed px:
answer **150×200** without description / **300×200** with **vertical** splitter;
task **300×200**; ~×2 type; explicit light/dark `--pack-tile-*` contrast — not
`--q-card-background` alone; difficulty top-left on tasks; Edit/Delete as
bottom full-width **text** buttons when editable).

**Catalog packs (SC-PACK-228):** `PackListCardTile.vue` stays **150×200**
(status top; star TL; truncated title + description; bottom full-width **text**
actions). Do **not** fold task-set summary chrome into this tile.

**Task-set lists — live + cards editor (SC-PACK-229 / 239…244):** dedicated
`PackTaskSetCardTile.vue` (not `PackListCardTile`). Mock-aligned summary chrome:

- Status top (short badges — СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …). Badge SVGs
  deferred; reserved names under `src/assets/content/`:
  `task-set-badge-revise.svg`, `task-set-badge-pending.svg`,
  `task-set-badge-unpublished.svg`, `task-set-badge-draft.svg`. Live/editor
  `#status` slots use temp Material `close` / `schedule` / `visibility_off` /
  `edit_note` inside `q-badge.pack-task-set-status-badge` (mock: icon+label).
  Colors: pending = Quasar `warning` (yellow/gold); needs_revision / draft /
  unpublished = `grey` (ДОРАБОТАТЬ is muted, **not** `color="warning"` /
  loud orange). Reserved top band keeps empty space when no badge
  (`.pack-task-set-tile__status` min-height — SC-PACK-244).
- Ordinal title with hash, no author/coauthor: i18n `taskSetLabel` =
  «Набор заданий #{n}» (live, editor, staff hub, tasks header, lobby create/list).
- Stats **column grid** (SC-PACK-244): CSS
  `grid-template-columns: var(--pack-ts-lead-w) minmax(0, 1fr) auto` with
  `--pack-ts-lead-w: 22px` (three 6px dots + 2×2px gaps). Each row wraps lead
  content in `.pack-task-set-tile__lead`; total-row Material `description`
  (placeholder until `task-set-card-tasks.svg`) centered in lead; labels
  («Заданий:» / «Лёгкие:» / «Средние:» / «Сложные:») share one left edge;
  counts right-aligned tabular.
- **Pale multi-row dividers** (SC-PACK-239): 1px low-contrast
  (`--pack-ts-splitter` ~0.08 light / ~0.12 dark) after total, **between every**
  difficulty row (`pack-task-set-divider-diff-*`), and above actions
  (`border-top` on `.pack-task-set-tile__actions`). Not only two section dividers.
- Difficulty rows: three-dot indicator (filled count = difficulty); filled
  colors 1 green / 2 amber / 3 red (`--pack-ts-dot-1/2/3`); **unfilled =
  outline rings** (not solid muted).
- Bottom full-width **outline + icon** actions (`@click.stop`); **slim**
  height via `--pack-ts-action-h: 30px` + `:deep(.q-btn)` min 28 / max 32 +
  page `dense` (SC-PACK-241). Stacked when multiple. Soft-unpublish «Снять»
  uses the same neutral outline chrome as Edit (no `color="warning"`).
  Pages gate `#actions` with a helper (`hasTaskSetCardActions` /
  `hasEditorTaskSetCardActions`) — omit the slot when no control would render
  (empty actions chrome + pale divider above actions — SC-PACK-239). Card
  soft-unpublish / republish: short keys `taskSetCardUnpublish` «Снять» /
  `taskSetCardRepublish` «Вернуть». Pack/catalog `content.unpublish` /
  `content.republish` and confirm dialogs stay long.
- Size tokens today: width `156px`; `min-height: 232px` (MAY exceed 200);
  soft resting `box-shadow`; body/stats `flex: 1 1 auto` vertical fill;
  `cascade-gap-outline` (SC-PACK-126); explicit light/dark `--pack-ts-*`
  contrast; **no hover scale / no new hover-polish** (existing clickable
  border only). Answer/task playing-card tiles stay unchanged.

Maps list uses `MapListCardTile.vue` (mini preview top, capacity, bottom text
actions — SC-MAP-55).

**Answer slots (SC-PACK-237/238):** compose slot rows and `PackTaskTile` slot
lists use global `.peek-slot-like` (+ `--filled` / `__label` / `__empty` /
`.peek-slot-like-row`) shared with GamePage `.peek-slot` in `app.scss`
(~72×40, dashed empty / solid filled). Do **not** reintroduce dense `q-chip`
as the primary slot chrome on these surfaces. Do **not** reintroduce list
`q-chip` for answer/task bodies or slot pickers. Contracts: meta
`work-with-stores/content.md` (Playing-card chrome); SC-PACK-222…229 / 237/238 /
239…244.
