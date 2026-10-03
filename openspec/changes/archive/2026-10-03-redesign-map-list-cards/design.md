## Context

See `proposal.md` — Why. Maps list cards today: `MapListCardTile` ~150×220, author + single `maps.seatConfig` (`players×tourists`), status badge in body, flat colored actions, local `--map-list-*` + hover `--q-secondary`. Pack/task-set cards already share `--pack-card-*`, outline+icon actions, short badges, muted 0.72+dashed. Prepare-mock light+dark mock attached; **consistency with pack/task-set chrome overrides mock pixels** where they diverge (published badge, long revise copy, dark bg `#2f…`, badge TL).

## Goals / Non-Goals

**Goals:**

- Redesign Maps list card: no author; two capacity rows; centered overlay status; outline actions; width **180**; consume `--pack-card-*`
- Short badge/action copy aligned with task-set / catalog pack cards
- Visual Spec measurable for apply + verify-mock
- Capacity / badge CSS masks render silhouettes (SC-MAP-68), including after Vite SVG data-URL inline

**Non-Goals:**

- Server/API contract changes; Map editor / View author meta; lobby picker; pack/task-set body redesign (except shared mask `url("${…}")` quote on PackTaskSetCardTile)
- «ОПУБЛИКОВАНО» badge; hover scale
- Stripping `authorDisplayName` (or seat/grid fields) from `GET /api/content/maps`
- Redrawing SVG assets for the solid-square mask bug

## Decisions

0. **HTTP / server unchanged (client display only)**
   - `listMaps` / `GET /api/content/maps` already returns `authorDisplayName`, `players`, `touristsPerPlayer`, `grid`, `inCatalog`, `moderationStatus`, `hasLive`, unpublish/republish routes — **aligns**; this change does **not** extend or alter them.
   - Soft-unpublish muted chrome reads existing `hasLive && inCatalog === false` (same product sense as packs).
   - Server mocha SC-MAP-06 asserting `authorDisplayName` on the **list JSON** stays valid; only client vitest drops author from the **card**.

1. **Evolve `MapListCardTile` (not reuse PackTaskSetCardTile layout)**
   - Map-specific: preview + 2 seat rows. Shared chrome via `--pack-card-*` aliases.
   - Alternative: one mega-tile — rejected (bloat).

2. **Width 180**
   - Separate catalog from packs — OK to match pack catalog width for preview size.
   - Height grows with content (preview + stats + actions).

3. **No author on list card; keep on View / staff queue**
   - Removes prop `author` from list tile only; API field and SC-MAP-47 View meta unchanged.

4. **Capacity = two stat rows (task-set grid pattern)**
   - Props `players` / `touristsPerPlayer` (numbers); i18n `maps.mapCardPlayers` / `maps.mapCardTourists` («ИГРОКОВ:» / «ТУРИСТОВ:»).
   - Lead icons: `map-card-players.svg` / `map-card-tourists.svg` (CSS mask + `currentColor`); Material placeholder OK until assets.

5. **Status overlay centered on preview**
   - Absolute center-top on preview; empty when published (SC-MAP-45).
   - Copy: `content.taskSetCardBadge.*`; soft pills + revise red outline like pack/task-set (reuse badge SVG masks).
   - Pending maps use short «НА ПРОВЕРКЕ» (maps list already has pending; chrome upgrade).

6. **Actions = catalog/task-set pattern**
   - `outline` + `icon` + `--pack-card-action-h`; labels `content.edit` / `content.taskSetCardUnpublish` / `content.taskSetCardRepublish`.
   - Confirm stays `maps.unpublish` / `maps.unpublishConfirm*`.
   - Staff Edit: same outline; label `maps.staffEdit` or `content.edit` (same RU string).

7. **Muted soft-unpub**
   - `:muted` when `hasLive && inCatalog === false` → opacity 0.72 + dashed (pack sense).

8. **Consistency > mock**
   - Colors/spacing/hover from `--pack-card-*` and PackTaskSetCardTile rhythm (lead 22, divider air ~4–6, body pad x 12, action pad).
   - Ignore mock: ОПУБЛИКОВАНО; long ТРЕБУЕТ ДОРАБОТКИ; TL badge; dark `#2f2f2f`.

9. **CSS mask URL quoting (Vite) — hotfix after apply**
   - Symptom: capacity / draft badge icons render as solid `currentColor` squares; opening the SVG file looks correct.
   - Cause: `iconMaskVars` built as `` `url(${imported})` `` without quotes; Vite may inline small SVGs as `data:image/svg+xml,…`, which makes unquoted `url()` invalid → browser drops `mask-image` → full background square.
   - Fix (Vite canon): `` `url("${imported}")` `` on `MapListCardTile` and the same pattern on `PackTaskSetCardTile`. Alternatives `?no-inline` / `assetsInlineLimit` — rejected as broader than needed.
   - Do **not** redraw or replace the SVG assets for this bug.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Vitest SC-MAP-06/55 assert `Alice` + `maps.seatConfig`; SC-MAP-31/32 assert long `content.statuses.*` / `maps.draftOnly` | Update **client** tests: no author on card; two capacity rows; short `content.taskSetCardBadge.*` (SC-MAP-55/66/67) |
| Missing SVG assets | Already in sibling HEAD; Material placeholder only if absent |
| Misread delta SC-MAP-06/16 «no author on card» as «drop author from API» | Keep `authorDisplayName` in list/detail JSON; server mocha SC-MAP-06/16 stay on visibility + payload; client vitest owns card chrome |
| Mask icons look correct in `vite`/`npm run`dev` (file URL) but fail after build / Pages | Quote `url("${…}")`; assert SC-MAP-68; smoke prod build or DevTools `mask-image` not dropped |

## Migration Plan

- Client-only UI; no API migration.
- After apply: sync delta → main `content/maps`; update skills `pack-cards.md` / maps topics.

## Open Questions

- (none) Staff Edit: **keep** `maps.staffEdit` (visible RU same as `content.edit`); do not rename for this change.

---

## Visual Spec (from mock)

**Mock:** `assets/map-card-mock.jpg` (light + dark).  
**Canon priority:** pack/task-set chrome (`--pack-card-*`, short badges/actions) **over** mock pixels. Mock supplies structure, capacity copy, overlay idea, muted dashed.  
**Token host:** Maps list already wraps tiles in `.pack-card-grid` (`MapsListPage`) — tile aliases `--pack-card-*` from that grid (same as PackTaskSetCardTile). Do **not** reintroduce local `--map-list-*` surface/hover.

### Structure (top → bottom)

1. Mini map preview (square-ish, upper card; `position: relative` host for badge)
2. Status badge overlay — **absolute**, horizontally centered, near top of preview (`top` ~6–10; `left: 50%` + `translateX(-50%)`; user override of mock TL)
3. Stats body: players row → pale divider → tourists row
4. Pale divider above actions (border-top on actions region **or** explicit divider — same product sense as PackTaskSetCardTile)
5. Outline actions: Edit → Снять **or** Вернуть

**No author. No published badge.** Empty `#status` when published (no reserved body band below preview).

### Copy

| Element | Canon |
|---------|--------|
| Badge draft / pending / revise / unpublished | `ЧЕРНОВИК` / `НА ПРОВЕРКЕ` / `ДОРАБОТАТЬ` / `СНЯТО` (`content.taskSetCardBadge.*`) |
| Published | **no badge** |
| Players / tourists labels | `ИГРОКОВ:` / `ТУРИСТОВ:` (`maps.mapCardPlayers` / `maps.mapCardTourists`) |
| Edit / Unpublish / Republish (card) | `content.edit` / `content.taskSetCardUnpublish` / `content.taskSetCardRepublish` |
| Staff Edit (card) | `maps.staffEdit` (same visible string as Edit) |

### Typography

| Role | font-size | weight | line-height | Align |
|------|-----------|--------|-------------|--------|
| badge | ~10px / 0.62rem | 600–700 | ~1.2 | center |
| stats-label | ~0.68rem | 400–500 | 1.2 | left (shared edge) |
| stats-count | ~0.68rem | 600 | 1.2 | right (shared edge) |
| button | ~0.72rem | 400–500 | 1.2 | center icon+text |

### Geometry / element sizes

| Element | W | H | Pad / notes | Confidence |
|---------|---|---|-------------|------------|
| card | **180** | min grows (~240–280) | radius **12**; border **1px** solid | high (decision) |
| preview | content − pad | ~120–140 | pad ~8–12; relative host | approx |
| badge overlay | auto | ~18–20 | pad 2×6, pill; top inset ~6–10; centered | high |
| lead col | **22** | — | icon centered; column-gap ~0.3rem | high (canon) |
| stat row | full | pad-y **6** | grid: lead \| label \| count | high |
| divider | full | **1px** | air ~4–6 (`~2×3px`) between rows + above actions | high |
| body | — | — | pad **x 12**, pad-bottom ~6 (PackTaskSet body sense) | high |
| actions region | full | — | pad ~`3px 8px 8px`; gap ~0.2rem between stacked btns | high (task-set) |
| action btn | full | **30** (`--pack-card-action-h`, 28–32) | outline; btn pad `0 0.4rem` | high |
| stat icon | 14×14 | — | CSS mask + `currentColor` | high |
| badge icon | 12×12 | — | existing `task-set-badge-*.svg` | high |
| action icon | ~18 | — | Material | high |

### Цвета (canon `--pack-card-*`)

#### Light

| Token | Value |
|-------|-------|
| card.bg | `#ffffff` |
| card.border.rest | `rgba(0,0,0,0.14)` |
| card.border.hover | `#212121` |
| card.shadow.rest / hover | `0 1px 3px rgba(0,0,0,0.08)` / `0 4px 12px rgba(0,0,0,0.14)` |
| fg / count | `rgba(0,0,0,0.87)` |
| muted / stats label / icon.ink | `rgba(0,0,0,0.7)` |
| divider | `rgba(0,0,0,0.08)` |
| badge.muted bg/fg | `rgba(0,0,0,0.06)` / `rgba(0,0,0,0.72)` |
| badge.pending | `rgba(249,168,37,0.16)` / `#f9a825` |
| badge.revise | transparent + `#af5a59` outline/fg |
| action.outline / label | outline `var(--pack-card-border)`; label/icon `var(--pack-card-fg)` |
| muted opacity | **0.72** + dashed border |

#### Dark

| Token | Value |
|-------|-------|
| card.bg | `#2a2a2a` (not mock `#2f…`) |
| card.border.rest | `rgba(255,255,255,0.22)` |
| card.border.hover | `#bdbdbd` |
| shadow rest/hover | `0 1px 3px rgba(0,0,0,0.35)` / `0 4px 14px rgba(0,0,0,0.5)` |
| fg | `rgba(255,255,255,0.92)` |
| muted | `rgba(255,255,255,0.78)` |
| divider | `rgba(255,255,255,0.12)` |
| badge.muted | `rgba(255,255,255,0.12)` / `rgba(255,255,255,0.82)` |
| badge.pending | `rgba(255,193,7,0.14)` / `#ffc107` |
| badge.revise | `#aa4a49` outline/fg |
| action.outline / label | same as light: border token + fg |
| muted opacity | 0.72 + dashed |

### Icon ink / sizes

| Place | Size | Ink | Asset |
|-------|------|-----|-------|
| players row | 14 | stats muted | `map-card-players.svg` |
| tourists row | 14 | stats muted | `map-card-tourists.svg` |
| badge draft/pending/unpublished | 12 | badge.fg | `task-set-badge-*.svg` |
| badge revise | — | revise.fg | no icon (catalog sense) |
| Edit / Снять / Вернуть | ~18 | action.fg | Material `edit` / `visibility_off` / `visibility` |

### Missing icons

| ID | Place | Suggested file | Temp Material | Blocker? |
|----|-------|----------------|---------------|----------|
| I1 | players lead | `map-card-players.svg` | `person_outline` | no — **already in sibling** `src/assets/content/` |
| I2 | tourists lead | `map-card-tourists.svg` | `luggage` | no — **already in sibling** `src/assets/content/` |

Apply: **reuse** existing SVGs (wire CSS mask); Material only if file missing.

### Hover / focus

| Target | Property | Default → Hover |
|--------|----------|-----------------|
| card | border | `--pack-card-border` → `--pack-card-border-hover` |
| card | shadow | rest → soft lift |
| card | scale | **none** → **none** |
| card | bg / stats / icons | unchanged |

Hover not shown on mock — use pack-card canon.

### Mock deviations (do not implement)

- «ОПУБЛИКОВАНО» badge
- Long «ТРЕБУЕТ ДОРАБОТКИ»
- Badge top-left
- Dark card bg `#2f2f2f`
- Flat / brand-tinted action buttons

## Implementation notes (client)

**Package:** client (`../happy-tourist.github.io` via projects-map).

**Sibling evidence (align-front 2026-10-02):** `MapListCardTile` (~150×220, author + single `capacity`, body `#status`, hover `--q-secondary`); `MapsListPage` already on `.pack-card-grid`, wires `author`/`seatConfig`, Quasar solid badges (`content.statuses.*` / `maps.draftOnly` / `maps.unpublishedByStaff`), flat colored actions; `PackTaskSetCardTile` + `app.scss` `--pack-card-*` = chrome analogue; `MapSummary` already has `players` / `touristsPerPlayer` / `authorDisplayName` / `hasLive` / `inCatalog` / `moderationStatus` (display-only drop of author). Lead SVGs already present under `src/assets/content/`.

| Area | Touch |
|------|--------|
| Tile | `src/components/MapListCardTile.vue` — width 180; preview + centered overlay `#status`; two stat rows; drop `author`/`capacity` props; `--pack-card-*` aliases; muted; outline actions slot; hover border+shadow only; **`iconMaskVars` → `url("${…}")`** |
| Task-set tile | `src/components/PackTaskSetCardTile.vue` — same quoted `url("${…}")` for existing mask vars (drive-by; same Vite pitfall) |
| Page | `src/pages/MapsListPage.vue` — pass `players`/`touristsPerPlayer`; short badges (`taskSetCardBadge.*` + tone/icon helpers like ContentPackPage / PackTaskSet); outline+icon actions; `:muted`; keep confirm `maps.unpublish*` |
| Assets | reuse `map-card-players.svg` / `map-card-tourists.svg` (+ existing `task-set-badge-*.svg`) — **no redraw** for mask square bug |
| i18n | add `maps.mapCardPlayers`, `maps.mapCardTourists` |
| Tests | `ContentMaps.test.ts` / `MapListCardTile.test.ts`: SC-MAP-06/16/31/32/45/55 + 66/67/68 — drop Alice/`seatConfig` on **card**; assert capacity rows + short badges + outline; assert mask CSS vars use quoted `url("`… |
| Styles skill | `pack-cards.md` maps section + note: JS-built mask vars MUST use `url("${imported}")`; `work-with-pages/content-pages.md`; `work-with-stores/maps.md` |

No server changes. Checklist → `tasks.md`.
