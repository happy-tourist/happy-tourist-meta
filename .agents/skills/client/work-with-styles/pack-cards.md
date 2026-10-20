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

Answer/task **lists** use global wrap class `.pack-card-grid` (`display: flex;
flex-wrap: wrap; gap: 0.75rem`) in `src/css/app.scss`. Tile chrome lives in
scoped styles on `PackAnswerCardTile.vue` / `PackTaskTile.vue` (fixed px:
answer **150×200** without description / **300×200** with **vertical** splitter;
task **portrait** pack-card family W **156–180** / min-H ≈**206**, radius 12,
`--pack-card-*` surface; peek-like dots + `content.difficultyWord.*` badge;
question **24px** (peek parity); filled `.peek-slot-like--portrait` only,
**centered** row + ≈8–12px air above actions (no flex-grow void pinning slots);
Edit/Delete as bottom full-width **outline + icon** (`edit` / `delete`; Delete
MAY negative) when editable). Create/edit of answers and tasks is a
**dialog** (SC-PACK-256…260 / 279/281) — not an always-visible inline form
above the grid; when compose is allowed, expose an add control above the list.

**Catalog packs (SC-PACK-228 / 249…255):** `PackListCardTile.vue` resting size
about **180×260** (width near 180; height MAY grow with four set rows, overflow
caption, and stacked actions). Pad root **x ~12** / top chrome **~12** (+ pb ~6
for status→title air); star TL offset **~8** / icon **~16** (constrain Quasar sm
round via tile CSS); status badge TR offset **~10**. Chrome: status top (**revise**
= red outline + short «ДОРАБОТАТЬ», badge TR on mock; other statuses keep existing
catalog labels); star TL; truncated **uppercase** title (**15px** CSS px) +
description (**12px**) + revise badge (**10px**) — lock Spec type sizes as
**CSS `px`**, not `rem`/root-relative; **published-only** task-set preview rows
(`inCatalog && !neverLive`; ≤4 + i18n `packCardSetsOverflow` «ещё {k}» among
that filtered list; display ordinal `#{n}` remapped 1…N among published — not
API ordinal; lead icon = `task-set-card-tasks.svg` mask 14px; count right; row
gap ~10 in `.pack-list-tile__sets-list` only (overflow **sibling**,
`margin-top` ~5 — Spec sets→overflow ~4–6; overflow indented under label
column); soft-unpub / neverLive **omitted** from DOM — no
`pack-list-tile__set-row--muted`; rows non-navigating); set label / count /
icon ink = **title.fg** (`--pack-list-fg` → shared `--pack-card-fg`; Decision 12 —
not muted grey); pale divider **only** above actions (air ~8–10); bottom
full-width **outline + icon** actions (Edit / short «Снять»/«Вернуть» via
`taskSetCardUnpublish` / `taskSetCardRepublish` — same keys as task-set cards;
confirm / live pack header keep long `content.unpublish` / `content.republish`;
slim ~28–32 via `--pack-card-action-h`, label ~11–12px, icon ~18, nowrap;
outline = `--pack-card-border`, **not** list-only `#aeaeae`). Soft-unpublished
**pack** cards: catalog wires `:muted` when `hasLive && inCatalog === false`
(opacity **0.72** + **`border-style: dashed`**; class `pack-list-tile--muted`).
Hover: border + soft shadow like task-set (**no** scale; light `#212121` /
dark `#bdbdbd` from shared tokens — not `--q-secondary`). Do **not** fold
task-set difficulty-dot chrome into this tile.

**Shared resting chrome (SC-PACK-255):** define `--pack-card-bg` / `fg` /
`muted` / `border` / `border-hover` / `shadow` / `shadow-hover` / `splitter` /
`action-h` on `.pack-card-grid` in `src/css/app.scss` (+ `body.body--dark`
overrides). Canon = former task-set surface values (dark bg `#2a2a2a`, **not**
mock `#2f2f2f`). `PackListCardTile`, `PackTaskSetCardTile`, `MapListCardTile`,
and **lobby listing** `LobbyRoomCardTile` alias these for resting surface /
hover / action outline / muted. Layout-only tokens stay local
(`--pack-list-lead-w`, `--pack-list-revise-fg`, `--pack-ts-lead-w`,
`--lobby-room-lead-w`, dots, spacing).
Visual Spec: change `redesign-pack-list-cards` / main `content/packs` after sync.

**Lobby room listing (SC-LOBBY-25/26/33/34/35/36):** `LobbyPage` wraps rooms in
`.pack-card-grid`; `LobbyRoomCardTile.vue` consumes `--pack-card-*` via
`--lobby-room-*` aliases (same host tokens as pack/map cards). Card ~180 wide;
map preview (~156) + centered short status ОЖИДАНИЕ/ИГРА; seats + tourists as
**full-width metric rows** matching set-row grid (`lead 22 | label | count` —
SC-LOBBY-36; plural seats by maxSeats, tourists by touristsPerPlayer) with pale
divider between them; uppercase pack title; set rows with listing metadata
**`taskCount`** (≤4 + `content.packCardSetsOverflow`); outline «Войти». Lead
masks `lobby-room-card-seats.svg` / `map-card-tourists.svg` /
`task-set-card-tasks.svg` via quoted `` `url("${…}")` `` (SC-MAP-68). Contracts:
meta `work-with-lobby` / `listing-cards.md`.

**Task-set lists — live + cards editor (SC-PACK-229 / 239…248):** dedicated
`PackTaskSetCardTile.vue` (not `PackListCardTile`). Mock-aligned summary chrome:

- Status top (short badges — СНЯТО / ДОРАБОТАТЬ / НА ПРОВЕРКЕ / …). Badge icons
  wired from `src/assets/content/`:
  `task-set-badge-revise.svg`, `task-set-badge-pending.svg`,
  `task-set-badge-unpublished.svg`, `task-set-badge-draft.svg` (SC-PACK-248).
  Live/editor `#status` slots use `<span class="pack-task-set-status-icon
  pack-task-set-status-icon--{revise|pending|unpublished|draft}">` (~12px CSS
  **mask** + `currentColor` ink).   Soft muted pills (not Quasar solid
  `color="grey"` / `warning` fills): reset Quasar badge chrome first
  (`background: transparent; color: inherit` on the status `q-badge`), then
  tone classes — `--muted` = pale grey bg + dark/light fg; `--pending` = soft
  amber tint + gold/amber ink (НА ПРОВЕРКЕ). ДОРАБОТАТЬ stays muted, **not**
  loud orange.
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
  height via `--pack-ts-action-h: var(--pack-card-action-h)` (= 30px) +
  `:deep(.q-btn)` min 28 / max 32 + page `dense` (SC-PACK-241). Match
  Map/PackList action type: label `font-weight: 500`, `line-height: 1.2`,
  `white-space: nowrap`; `:deep(.q-btn .q-icon)` **18px** +
  `color: currentColor`. Stacked when multiple. Soft-unpublish «Снять»
  **and** republish «Вернуть» use the same **neutral** outline chrome as Edit
  (pale `--pack-ts-border` → `--pack-card-border`; no `color="warning"` /
  no `color="primary"` tint).
  Pages gate `#actions` with a helper (`hasTaskSetCardActions` /
  `hasEditorTaskSetCardActions`) — omit the slot when no control would render
  (empty actions chrome + pale divider above actions — SC-PACK-239). Card
  soft-unpublish / republish: short keys `taskSetCardUnpublish` «Снять» /
  `taskSetCardRepublish` «Вернуть» (catalog pack cards use the **same** short
  keys). Pack confirm dialogs + live pack **header** keep long
  `content.unpublish` / `content.republish`.
- Size tokens: width `156px`; `min-height: 206px` (resting mock; MAY grow with
  spacing); soft resting `box-shadow`; body/stats `flex: 1 1 auto` vertical fill;
  `cascade-gap-outline` (SC-PACK-126); resting surface via shared
  `--pack-card-*` (aliases `--pack-ts-bg/fg/muted/border/…`); layout tokens
  (`--pack-ts-lead-w`, dots, spacing) stay local. Soft-unpublish `--muted`:
  opacity `0.72` + **`border-style: dashed`** (mock СНЯТО). Stats **counts**
  use `--pack-ts-fg` (title.fg, weight 600); labels/icon ink stay
  `--pack-ts-muted` at **weight 400–500** (not 600).
- **Icon ink / colors** (SC-PACK-248 / Visual Spec): soft-muted badge
  light `rgba(0,0,0,0.06)` bg + `rgba(0,0,0,0.72)` fg/ink; pending light
  `rgba(249,168,37,0.16)` + `#f9a825`; dark muted `rgba(255,255,255,0.12)` /
  `0.82` fg; dark pending `rgba(255,193,7,0.14)` + `#ffc107`. Total/badge
  icons theme via **mask + `currentColor`** (not plain `<img>`). Action icons
  stay Quasar Material. JS-built mask custom properties MUST quote the Vite
  import: `` `url("${imported}")` `` on `PackListCardTile` /
  `PackTaskSetCardTile` / `MapListCardTile` (same SC-MAP-68 pitfall —
  unquoted `data:` URLs drop `mask-image`).
- **Themed hover** (SC-PACK-246 / Decision 10): clickable cards change
  **border + soft lift shadow only** — **no** `transform: scale` / enlarge.
  Light: `--pack-card-border-hover: #212121` + `--pack-card-shadow-hover` lift
  (aliased as `--pack-ts-border-hover` / `--pack-ts-shadow-hover`).
  Dark: `--pack-card-border-hover: #bdbdbd` (light grey) — **not** `--q-secondary`.
  Difficulty dots, total icon, badge icons, action icons keep semantic colors
  on hover (chrome only). Answer/task playing-card tiles stay unchanged.

**Maps list (SC-MAP-55/66/67/68):** `MapListCardTile.vue` resting width **180**
(height grows with preview + capacity + actions). Mini `MapGridPreview` top
with **centered overlay** `#status` near preview top (`left: 50%` +
`translateX(-50%)`; empty when published — no reserved body band). Preview
cells use slight rounding (`border-radius: 1px` on
`.map-grid-preview__cell`). Body: two capacity rows (lead **22** + uppercase
`maps.mapCardPlayers` / `maps.mapCardTourists` + count) with pale divider
between; **no author**. Stats **labels** weight **500**; counts **600**. Lead
icons from `map-card-players.svg` / `map-card-tourists.svg` via CSS mask.
**Vite mask `url()` quoting (SC-MAP-68):** JS-built `iconMaskVars` MUST use
`` `url("${imported}")` `` (double quotes), not `` `url(${imported})` `` —
Vite may inline small SVGs as `data:image/svg+xml,…`, and unquoted `url()` is
invalid CSS → browser drops `mask-image` → solid `currentColor` square.
Same rule on `PackTaskSetCardTile` and `PackListCardTile` `iconMaskVars`.
Do **not** redraw SVG assets to work around a broken mask URL.
Short badges via page-wired `content.taskSetCardBadge.*` (ЧЕРНОВИК / НА
ПРОВЕРКЕ / ДОРАБОТАТЬ / СНЯТО) — soft pills (reset Quasar primary fill via
`background: transparent; color: inherit` before tone) + pending amber +
revise red outline (`pack-list-status-badge--revise` catalog sense); badge
SVG masks reuse `task-set-badge-*.svg`. Soft-unpub `:muted` when `hasLive &&
inCatalog === false` (opacity **0.72** + dashed). Bottom full-width
**outline + icon** actions (Edit / short `taskSetCardUnpublish` /
`taskSetCardRepublish`; confirm keeps long `maps.unpublish*`); slim via
`--pack-card-action-h`. Resting surface / hover / action outline alias shared
`--pack-card-*` from host `.pack-card-grid` via layout-only `--map-list-*`
aliases (do not invent independent map surface/hover colors);
hover border+shadow only — **no** scale / **no** `--q-secondary`). Gate
`#status` with page `hasMapCardStatus` (omit overlay host when published) and
`#actions` with `hasMapCardActions` (omit empty chrome + pale divider).

**Answer slots (SC-PACK-237/238 + redesign-peek-task-modal / redesign-pack-task-tile):**
Game peek and task **compose** use `.peek-slot` / `.peek-slot-like` **plus**
`--portrait` (unlabeled empty dashed; filled mini-card; AR 156∶206). Peeker
empty slots keep `--interactive` (cursor pointer drop-target affordance; click
still no-ops until filled). `PackTaskTile` list slots show **filled only** as
`.peek-slot-like--portrait` (content + optional description; gap 8;
`justify-content: center`; `overflow-x: auto`; ≈8–12px air above actions);
question **24px** / weight 600 (SC-BOARD-54 parity — **not** 13–15px);
empty slots omitted; zero filled → omit slots row (no `content.slotEmpty`
body placeholder). Edit/Delete = slim **outline + icon** (task-set action
parity; **not** flat text-only). Do **not** reintroduce dense `q-chip` as
primary slot chrome. Do **not** reintroduce list `q-chip` for answer/task
**bodies**. Peek/compose dialog cards use `--pack-card-bg` / `fg` on the card
root (not raw Quasar dark surface).

**Task compose / game peek answer pool (SC-PACK-260 / 263 / 277; SC-BOARD-49/51/52):**
pool = `.peek-answer-pool` grid of `.peek-answer-mini` cards (`--pack-card-*`,
AR 156∶206, half-size `--peek-tile-max-w` ~43, ≤4×4 + `overflow-y: auto`;
`max-height` from tile band) — **not** catalog `PackAnswerCardTile` /
`slot-picker-grid` / flat `q-chip`. Filled portrait slots: thin primary
`#1976d2` border; pool keeps neutral `--pack-card-border` (no used chrome).
Compose slots use `.peek-slot-like--portrait`; center the row
(`justify-center`). Pool cards stay reusable across slots with **no** used /
disabled chrome (SC-PACK-263; Game peek SC-BOARD-49). Difficulty header =
3 pack dots (**left edge**) + **word-only** badge (`content.difficultyWord.*`,
horizontally **centered** — not one centered cluster; compose interactive
SC-PACK-276; game read-only SC-BOARD-50). **Compose shell width** (SC-PACK-279):
`max-width` / `min-width` = `min(4×--peek-tile-max-w + 3×8px + pad + ≈50px, 92vw)`
on `.pack-task-compose-card` — **not** fixed min 360 / max 400. **Compose-only
dots** (SC-PACK-280): Ø ≈20px via `--compose-diff-dot-size` under
`.pack-task-compose-card`; shared `.peek-diff-dot` base stays **6px** for game
peek / list tile. **Compose Save** (SC-PACK-281): full-width primary unelevated
peekSubmit-like chrome (`--pack-card-action-h`); labels `content.saveTask` /
`content.addTask`; **no Cancel** btn (dismiss = backdrop/Esc). Game peek
question ~24px (SC-BOARD-54) + peek submit — unchanged by compose polish;
compose `q-input` unchanged. List grids keep playing-card tiles. Contracts: meta
`work-with-stores/content.md`; SC-PACK-222…227 / 237/238 / 256…263 / 276…281 /
239…248; SC-BOARD-46/49/50…54.
