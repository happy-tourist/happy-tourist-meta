## Context

See `proposal.md` — Why. Catalog packs: `PackListCardTile` ~**180×260** with set preview from `listCatalog.taskSetsPreview` (already shipped). Follow-up: card shows **published-only** rows in **title.fg**; soft-unpub / never-live stay in API but not on the card. Live pack / `PackTaskSetCardTile` unchanged.

## Goals / Non-Goals

**Goals:**
- Redesign catalog pack tile to mock (~180×260): uppercase title, description, set preview ≤4 + «ещё K», outline+icon actions, red-outline revise badge
- Lightweight `taskSetsPreview` on `GET /api/content/packs` (server — done)
- **Card UI:** only published sets (`inCatalog === true` and not `neverLive`); display ordinal / overflow among that filtered list; set-row colors = `title.fg`
- Whole-card navigate; set rows non-interactive
- Update `pack-cards.md` + vitest for SC-PACK-249 revision / 254

**Non-Goals:**
- Changing `PackTaskSetCardTile` / live pack soft-unpub visibility or ordinals
- Filtering soft-unpub / never-live out of the **HTTP** `taskSetsPreview` payload
- Hover scale; grid gutter changes
- Custom SVG for Edit/Unpublish (Material)
- Changing pack unpublish confirm strings / ACL
- Staff hub card-grid

## Decisions

1. **Extend `PackListCardTile` (or strong variant) rather than reuse `PackTaskSetCardTile`**
   - Rationale: catalog needs title+description+set list, not difficulty dots; keep task-set tile untouched.
   - Alternative: one mega-tile — rejected (regress task-set).

2. **Server lightweight preview in `listCatalog`**
   - Pack row field: **`taskSetsPreview`**: array of `{ id, ordinal, taskCount, inCatalog, neverLive? }` (omit `neverLive` or set `false` for ordinary rows).
   - Return the **full** preview array for that pack (do **not** truncate to 4 on the server). Soft-unpublished and author never-live ghosts remain in the payload (SC-PACK-252/253) — **card display filter is client-only** (Decision 11).
   - Empty array when the pack has no chosen revision / no sets (`[]`), never omit the field once the contract ships (client MAY treat missing as empty during rollout).
   - Load sets + task counts for the **same revision** already chosen for title/description (`revId` in `listCatalog`: live for in-catalog / staff soft-unpub pack; working for author never-published). **Do not** call full `loadRevisionPayload` for the catalog sets path (no answer cards, no per-task slots). Prefer: select task sets for `revisionId` ordered by DB `position`, then `COUNT(*)` tasks grouped by `taskSetId`.
   - Merge author never-live ghosts with the **same visibility rules as live** (`neverLiveAuthorTaskSets` / `retainedNeverLiveAuthorTaskSetCycle`: pending | needs_revision | cancelled; author only; append after live/working sets). Ghosts need set id + task count only — reuse the cycle selection logic; load request-revision **set rows + task counts** (still no slots/cards). For matching against live author sets, a slim live set list (`id`, `authorUserId`, order) is enough — do **not** require full live slots/cards payload for list enrichment.
   - Cost: packs already capped at 200; prefer batched queries across listed `revisionId`s rather than N× full revision graphs. Ghost detection only for the calling user (same as live), not for every other author.
   - Alternative: client N+1 `getLivePack` — rejected.

3. **Ordinal `#{n}` (API vs card)**
   - API `ordinal` remains the **1-based index in full `taskSetsPreview` order** (soft-unpub and never-live stay in sequence server-side) — not the raw DB `position` column.
   - **Catalog card display ordinal** is the **1-based index among published-only** rows after client filter (Decision 11), passed to `content.taskSetLabel` / shown as `#{n}`. May differ from live pack numbering when soft-unpub sets exist between published ones — accepted (live out of scope).

4. **Overflow**
   - After published-only filter: show first **4**; caption i18n `content.packCardSetsOverflow` = `ещё {k}` where `{k}` = `publishedPreview.length - 4` when length > 4.

5. **Actions**
   - Quasar **outline** + Material `edit` / `visibility_off` / `visibility` (mirror live `PackTaskSetCardTile` / `ContentPackPage` card actions — not catalog’s current `flat` text-only). Long pack copy: `content.edit` / `content.unpublish` («Снять с публикации») / `content.republish` («Опубликовать снова»); slim ~30px via same `--pack-ts-action-h` / dense pattern; `@click.stop`; omit `#actions` when no control would render (same gate idea as `hasTaskSetCardActions`).

6. **Badge revise**
   - Mock: red **outline** + red text, **no** badge icon; place badge **top-right** (star stays TL). Copy MUST be short uppercase **`ДОРАБОТАТЬ`** — reuse `content.taskSetCardBadge.needs_revision` (or a dedicated catalog short key with the same string). MUST NOT keep catalog `content.statuses.needs_revision` («Нужна доработка») for this badge.
   - Other catalog statuses (pending / draft / unpublished / blocked): keep **existing** `ContentCatalogPage` labels and Quasar/solid colors as today — do **not** require migrating them to task-set short pills in this change.

7. **Title**
   - CSS `text-transform: uppercase` (do not require stored uppercase).

8. **Hover**
   - Not on mock → inherit task-set: border + soft shadow, no scale (light `#212121` / dark `#bdbdbd`).

9. **Star**
   - Keep current Quasar favorite control.

10. **Set-row icon**
    - Reuse `src/assets/content/task-set-card-tasks.svg` via mask + `currentColor` (14px).

11. **Published-only on catalog card (follow-up)**
    - Client filters `taskSetsPreview` to rows where `inCatalog === true` and `neverLive` is not true before slice(0, 4) / overflow / display ordinal.
    - Soft-unpublished and never-live ghosts MUST NOT render on the catalog card (no muted rows).
    - Rationale: simpler than server filter; API SC-PACK-253 stays; live pack keeps soft-unpub chrome.
    - Alternative: drop soft-unpub/neverLive from server preview — rejected for this polish (more mocha churn; not needed for UX).

12. **Set-row ink = title.fg (follow-up)**
    - `set.label.fg`, `set.count.fg`, and `set.icon.ink` MUST use the same token as `title.fg` (light `#232323` / dark near-white). Subtitle stays muted. Remove row-level muted opacity for set preview (nothing muted left on card).

## Risks / Trade-offs

- [listCatalog cost] → Mitigation: count-only / batched queries by revisionId; pack cap 200; avoid slots/cards; ghost path only for calling user with slim live set list + request set counts.
- [JPEG mock blur on button height] → Mitigation: align 28–32 with task-set token; verify-mock after apply.
- [Other status badges not on mock] → Mitigation: only revise forced to red outline; document in Visual Spec.
- [Card ordinal ≠ live ordinal when soft-unpub present] → Accepted; live out of scope (Decision 11 / Q1).

## Migration Plan

- Deploy server list preview before or with client (client treats missing preview as empty list) — already shipped.
- Follow-up: client-only; no DB / API migration.
- Rollback: revert tile filter/colors if needed.

## Open Questions

- None blocking.

## Visual Spec (from mock)

Source: change `assets/pack-card-catalog-mock.jpg` (+ `pack-card-mock-light-crop.png` / `pack-card-mock-dark-crop.png`). Anchor: Figma **180×260**.

### Структура

star TL → status badge TR → uppercase title (center) → description (center) → **published-only** set rows ≤4 (icon | label | count) → optional «ещё K» → pale divider → outline actions (Edit, Unpublish). Soft-unpublished and never-live sets are **omitted** (not muted). No pale dividers between set rows.

### Copy

| Role | Text | i18n |
|------|------|------|
| Badge revise | `ДОРАБОТАТЬ` | `content.taskSetCardBadge.needs_revision` (or equal short catalog key) — **not** `content.statuses.needs_revision` |
| Title example | `ГЕОГРАФИЯ МИРА` (CSS uppercase) | pack `title` |
| Description | pack description | pack `description` |
| Set row | `Набор заданий #{n}` | `content.taskSetLabel` (`n` = published-only index) |
| Overflow | `ещё {k}` | `content.packCardSetsOverflow` |
| Edit | `Редактировать` | `content.edit` |
| Unpublish | `Снять с публикации` | `content.unpublish` |
| Republish (not on mock) | `Опубликовать снова` | `content.republish` |

### Типографика (approx)

| Role | size | weight | line-height | align |
|------|------|--------|-------------|-------|
| title | 15–16 | 700 | ~1.2 | center, uppercase; clamp ~2–3 lines |
| subtitle | 11–12 | 400 | ~1.2 | center; clamp ~2–3 lines |
| set label | 11–12 | 400 | ~1.2 | left (shared edge after lead) |
| set count | 11–12 | 600 | ~1.2 | right (shared edge) |
| overflow | 11–12 | 400 | ~1.2 | left / muted (under rows) |
| badge | 9–10 | 600–700 | ~1.1 | center in pill |
| button | 11–12 | 400–500 | ~1.2 | center (icon+label) |

### Геометрия (approx CSS px)

| Token | Value |
|-------|-------|
| card.w × h | 180 × 260 (h MAY grow with 4 rows + overflow + 2 actions) |
| radius | 12 |
| pad root | x 12–16; y ~10–14 (chrome band + body + actions) |
| star | ~16; TL offset ~8–10 |
| badge | h ~18–20 pill; TR offset ~8–10; **no** leading icon |
| title→subtitle gap | ~4–6 |
| subtitle→sets gap | ~10–14 |
| set row h / gap | ~18–20 / ~8–12 |
| sets→overflow gap | ~4–6 |
| overflow→divider gap | ~8–10 |
| max visible sets | 4 (of **published-only** list) |
| lead col | ~22 (icon centered; same product sense as `--pack-ts-lead-w`) |
| set icon | 14 |
| divider | 1px above actions only (no between rows) |
| action h | 28–32; stacked gap ~4–6; full-width |

### Цвета

#### Light

| Token | Value | Conf |
|-------|-------|------|
| card.bg | `#ffffff` | high |
| card.border.rest | `rgba(0,0,0,0.12)` | med |
| title.fg | `#232323` / `#242424` | high |
| subtitle.fg | `#717171` | high |
| set.label.fg | **= title.fg** | high (follow-up) |
| set.count.fg | **= title.fg** | high (follow-up) |
| set.icon.ink | **= title.fg** (`currentColor`) | high (follow-up) |
| badge.revise.bg | transparent | high |
| badge.revise.fg/border | `#af5a59`–`#b26b65` | high |
| divider | `rgba(0,0,0,0.08)` | med |
| action.outline | `rgba(0,0,0,0.22)` | low |
| action.label.fg / action.icon.ink | title.fg / `currentColor` → title.fg | med |

#### Dark

| Token | Value | Conf |
|-------|-------|------|
| card.bg | `#2f2f2f` | high |
| card.border.rest | `rgba(255,255,255,0.18)`–`0.22` | med |
| title.fg | `#d4d4d4`–`#fff` | med |
| subtitle.fg | `#979797` | high |
| set.label.fg | **= title.fg** | high (follow-up) |
| set.count.fg | **= title.fg** | high (follow-up) |
| set.icon.ink | **= title.fg** | high (follow-up) |
| badge.revise.bg | transparent | high |
| badge.revise.fg/border | `#aa4a49`–`#9e4f4b` | high |
| divider | `#5d5d5d` / rgba white 0.16 | med |
| action.outline | `#aeaeae` | high |
| action.label.fg / action.icon.ink | title.fg / `currentColor` → title.fg | med |

### Icon ink / sizes

| Place | Size | Ink | Asset |
|-------|------|-----|-------|
| star | ~16 | existing Quasar amber when favorited | Quasar `star` / `star_border` |
| set-row | 14 | set.icon.ink (= title.fg) | `src/assets/content/task-set-card-tasks.svg` via CSS mask + `currentColor` |
| badge | — | — | **no icon** |
| actions | Quasar ~18 | action.icon.ink | Material `edit` / `visibility_off` / `visibility` |

### Missing icons

- none

### Hover / focus

Hover not on mock. Implement like task-set (`PackTaskSetCardTile`): border + soft shadow only; **no** scale; light hover border `#212121`; dark `#bdbdbd`. Do **not** keep current catalog `--q-secondary` ring.

### Client type sketch (store)

```ts
/** Lightweight set row on GET /api/content/packs (SC-PACK-252/253). */
export interface PackTaskSetPreview {
  id: string;
  /** 1-based index in full taskSetsPreview order (API). Card remaps among published. */
  ordinal: number;
  taskCount: number;
  inCatalog: boolean;
  neverLive?: boolean;
}

export interface ContentPackSummary {
  // …existing fields…
  /** Full preview from API; card shows published-only ≤4 + overflow. Missing → []. */
  taskSetsPreview?: PackTaskSetPreview[];
}
```

## Implementation notes (files)

**Server:** `src/lib/content.ts` — `listCatalog` / pack summary enrichment; mocha SC-PACK-252/253 (unchanged for follow-up).

**Client (follow-up):** `PackListCardTile.vue` — filter published-only; display ordinal; set-row CSS → title.fg; drop muted set-row classes; vitest SC-PACK-249 revision + SC-PACK-254; `pack-cards.md`.

See `tasks.md` for checklist.
