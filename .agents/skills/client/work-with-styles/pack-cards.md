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

**Task-set lists — live + cards editor (SC-PACK-229 / 239…248):** dedicated
`PackTaskSetCardTile.vue` (not `PackListCardTile`). Mock-aligned summary chrome:

- Status top (short badges — СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …). Badge icons
  wired from `src/assets/content/`:
  `task-set-badge-revise.svg`, `task-set-badge-pending.svg`,
  `task-set-badge-unpublished.svg`, `task-set-badge-draft.svg` (SC-PACK-248).
  Live/editor `#status` slots use `<span class="pack-task-set-status-icon
  pack-task-set-status-icon--{revise|pending|unpublished|draft}">` (~12px CSS
  **mask** + `currentColor` ink). Soft muted pills (not Quasar solid
  `color="grey"` / `warning` fills):
  `--muted` = pale grey bg + dark/light fg; `--pending` = soft amber tint +
  gold/amber ink (НА ПРОВЕРКЕ). ДОРАБОТАТЬ stays muted, **not** loud orange.
  Reserved top band keeps empty space when no badge
  (`.pack-task-set-tile__status` min-height — SC-PACK-244); status→title
  pad-bottom **~6–8** CSS px (SC-PACK-247).
- Ordinal title with hash, no author/coauthor: i18n `taskSetLabel` =
  «Набор заданий #{n}» (live, editor, staff hub, tasks header, lobby create/list);
  title `font-weight: 700` (mock).
- Stats **column grid** (SC-PACK-244): CSS
  `grid-template-columns: var(--pack-ts-lead-w) minmax(0, 1fr) auto` with
  `--pack-ts-lead-w: 22px` (three 6px dots + 2×2px gaps). Each row wraps lead
  content in `.pack-task-set-tile__lead`; total-row custom SVG
  `task-set-card-tasks.svg` via CSS mask + `currentColor` (14×14;
  `stats.icon.ink` = `--pack-ts-muted`); labels
  («Заданий:» / «Лёгкие:» / «Средние:» / «Сложные:») share one left edge;
  counts right-aligned tabular.
- **Pale multi-row dividers** (SC-PACK-239): 1px low-contrast
  (`--pack-ts-splitter` ~0.08 light / ~0.12 dark) after total, **between every**
  difficulty row (`pack-task-set-divider-diff-*`), and above actions
  (`border-top` on `.pack-task-set-tile__actions`). Not only two section dividers.
- **Internal spacing** (SC-PACK-245 → retune SC-PACK-247 / Visual Spec):
  `--pack-ts-title-gap: 7px` (title→stats ~7); `--pack-ts-row-pad-y: 6px`
  (row pad-y 6–8); `--pack-ts-divider-air: 3px` → margin 3+3 = **6** CSS px air
  around pale dividers (~4–6 total). Do **not** change `.pack-card-grid` / list
  gutter between cards.
- Difficulty rows: three-dot indicator (filled count = difficulty); filled
  colors 1 green / 2 amber / 3 red (`--pack-ts-dot-1/2/3`); **unfilled =
  outline rings** (not solid muted).
- Bottom full-width **outline + icon** actions (`@click.stop`); **slim**
  height via `--pack-ts-action-h: 30px` + `:deep(.q-btn)` min 28 / max 32 +
  page `dense` (SC-PACK-241). Stacked when multiple. Soft-unpublish «Снять»
  **and** republish «Вернуть» use the same **neutral** outline chrome as Edit
  (pale `--pack-ts-border`; no `color="warning"` / no `color="primary"` tint).
  Pages gate `#actions` with a helper (`hasTaskSetCardActions` /
  `hasEditorTaskSetCardActions`) — omit the slot when no control would render
  (empty actions chrome + pale divider above actions — SC-PACK-239). Card
  soft-unpublish / republish: short keys `taskSetCardUnpublish` «Снять» /
  `taskSetCardRepublish` «Вернуть». Pack/catalog `content.unpublish` /
  `content.republish` and confirm dialogs stay long.
- Size tokens: width `156px`; `min-height: 206px` (resting mock; MAY grow with
  spacing); soft resting `box-shadow`; body/stats `flex: 1 1 auto` vertical fill;
  `cascade-gap-outline` (SC-PACK-126); explicit light/dark `--pack-ts-*`
  contrast. Soft-unpublish `--muted`: opacity `0.72` + **`border-style: dashed`**
  (mock СНЯТО). Stats **counts** use `--pack-ts-fg` (title.fg, weight 600);
  labels/icon ink stay `--pack-ts-muted`.
- **Icon ink / colors** (SC-PACK-248 / Visual Spec): soft-muted badge
  light `rgba(0,0,0,0.06)` bg + `rgba(0,0,0,0.72)` fg/ink; pending light
  `rgba(249,168,37,0.16)` + `#f9a825`; dark muted `rgba(255,255,255,0.12)` /
  `0.82` fg; dark pending `rgba(255,193,7,0.14)` + `#ffc107`. Total/badge
  icons theme via **mask + `currentColor`** (not plain `<img>`). Action icons
  stay Quasar Material.
- **Themed hover** (SC-PACK-246 / Decision 10): clickable cards change
  **border + soft lift shadow only** — **no** `transform: scale` / enlarge.
  Light: `--pack-ts-border-hover: #212121` + `--pack-ts-shadow-hover` lift.
  Dark: `--pack-ts-border-hover: #bdbdbd` (light grey) — **not** `--q-secondary`.
  Difficulty dots, total icon, badge icons, action icons keep semantic colors
  on hover (chrome only). Answer/task playing-card tiles stay unchanged.

Maps list uses `MapListCardTile.vue` (mini preview top, capacity, bottom text
actions — SC-MAP-55).

**Answer slots (SC-PACK-237/238):** compose slot rows and `PackTaskTile` slot
lists use global `.peek-slot-like` (+ `--filled` / `__label` / `__empty` /
`.peek-slot-like-row`) shared with GamePage `.peek-slot` in `app.scss`
(~72×40, dashed empty / solid filled). Do **not** reintroduce dense `q-chip`
as the primary slot chrome on these surfaces. Do **not** reintroduce list
`q-chip` for answer/task bodies or slot pickers. Contracts: meta
`work-with-stores/content.md` (Playing-card chrome); SC-PACK-222…229 / 237/238 /
239…248.
