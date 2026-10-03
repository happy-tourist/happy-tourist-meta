## Context

See `proposal.md` — Why. Lobby listing today: `LobbyPage` `q-list` / `q-item` with 56px `MapGridPreview`, caption capacity (`lobby.mapCapacityCaption` / `lobby.capacity`), pack·sets one-line caption, flat primary «Войти». Metadata already has `mapGrid`, `maxSeats`, `seats`, `touristsPerPlayer`, `packTitle`, `taskSetLabels[{taskSetId, authorDisplayName}]`, `status` waiting|playing — **no `taskCount`**. Pack/map/task-set cards already share `--pack-card-*` on `.pack-card-grid`. Prepare-mock light+dark attached; **consistency with pack/task-set/map chrome overrides mock pixels**.

## Goals / Non-Goals

**Goals:**

- Redesign lobby room listing as ~180 cards: preview + seats/tourists metric rows (label left + count right, same as sets) + pack title + set rows with counts + outline Войти
- Centered phase badge on preview (ОЖИДАНИЕ / ИГРА); soft pills like existing short badges
- Extend listing metadata with per-set `taskCount` from create snapshot
- Visual Spec measurable for apply + verify-mock; overflow ≤4 + «ещё {k}»
- Join: whole card + button; keep busy-lock

**Non-Goals:**

- Maps / pack / task-set body redesign; create-modal layout redesign
- Changing join/spectator ACL, maxSeats, densities, LobbyRoom protocol
- Hover scale; mock cream/navy as new dark theme
- Requiring custom seat SVG before apply (Material placeholder OK)

## Decisions

0. **Server: add `taskCount` to listing set labels (D1)**
   - In `refreshMetadata`, for each `snap.setLabels` entry count `snap.tasks.filter(t => t.taskSetId === id).length` (create-time snapshot; static for room life).
   - Shape: `taskSetLabels: [{ taskSetId, authorDisplayName?, taskCount }]`. Keep `authorDisplayName` for compatibility; UI ignores it.
   - Alternative: client fetches pack API — rejected (lobby must not HTTP-poll packs per room).

1. **New list tile component (lobby-specific)**
   - Dedicated tile (preview + seats/tourists + pack title + set rows + join slot), consume `--pack-card-*` via host `.pack-card-grid` on Lobby listing wrap.
   - Do not reuse `MapListCardTile` body (wrong capacity copy / actions) or `PackListCardTile` (favorites/star).
   - Wire from `LobbyPage`; keep create modal as today.

2. **Width 180**
   - Match Maps / Pack catalog resting width; height grows with content.

3. **Capacity copy (metric rows = set-row rhythm)**
   - Seats and tourists use the **same full-width grid** as set rows: `lead (22) | label left | count right` — **not** a centered `icon+text` block (supersedes verify-mock centering).
   - Seats: label = RU plural of «место» by **`maxSeats`** (`Место` / `Места` / `Мест`, sentence case); count = `{seats} / {maxSeats}` (reuse/adapt `lobby.capacity` for the count cell only).
   - Tourists: label = RU plural of «турист» by **`touristsPerPlayer`** (`Турист` / `Туриста` / `Туристов`); count = bare `n` without `+`. Split former combined `roomCardTourists*` into label keys + numeric cell.
   - Drop listing reliance on `lobby.mapCapacityCaption` (`N×M`) for the card body.
   - Lead icons share the same left edge as set-row icons.

4. **Set rows**
   - Label: short «Набор #{n}» (or `content.taskSetLabel` if already hash-form; prefer short card key if long «Набор заданий #{n}» overflows 180).
   - Icon: reuse `task-set-card-tasks.svg` (CSS mask + `url("${…}")`); same glyph all rows (ignore mock checkmark variant).
   - Max 4 + `content.packCardSetsOverflow` / lobby twin «ещё {k}».

5. **Status overlay**
   - Absolute center-top on preview (`left: 50%` + `translateX(-50%)`), same geometry sense as `MapListCardTile` status.
   - Copy: short «ОЖИДАНИЕ» / «ИГРА» (new lobby keys). Soft muted / pending-style pills — **reuse existing badge size/color tokens**, not moderation SVG set unless a neutral pill fits without wrong icons.

6. **Actions / join**
   - Outline full-width «Войти», `--pack-card-action-h` (30px), neutral outline (not `color="primary"` fill).
   - Card body click + button → same `onJoin`; `@click.stop` on button optional if whole card handles it; busy-lock unchanged.

7. **Seat icon**
   - Reserved `src/assets/content/lobby-room-card-seats.svg` (user will add). Until present: Material `event_seat` or mask when file lands. Tourists: reuse `map-card-tourists.svg`.

8. **Consistency > mock**
   - Colors/spacing/hover from `--pack-card-*` and pack/map rhythm (lead 22, row pad-y 6, divider air ~3+3, radius 12, dark `#2a2a2a`).
   - Ignore mock: cream bg, navy `#1e252b`, radius 16–20, taller button, dual set glyphs.

9. **Divider between seats and tourists**
   - Yes (MapListCardTile capacity rhythm), even if mock omits a visible line between those two.

10. **Seats/tourists align with set metrics (explore follow-up)**
    - User locked: count seats as `occupied / maxSeats` on the right; decline seats label by maxSeats; decline tourists by touristsPerPlayer; free choice of exact noun forms within that rule.
    - Typography for capacity labels/counts → match set-row sense (~0.68rem label muted left, count tabular right); seats count MAY stay slightly stronger weight if needed for `2 / 4` readability.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Existing client tests assert `q-item` / `lobby-room-*` captions | Update vitest to card testids; keep SC-LOBBY-11/19/25/26 intent |
| Ordinal «Набор заданий #{n}» too wide at 180 | Short card key «Набор #{n}»; truncate |
| Seat SVG missing at apply | Material placeholder; drop-in path documented |
| Metadata shape change breaks typed client | Extend `GameRoomMeta` optionally; tolerate missing taskCount → show `—` or 0 only if absent (mocha asserts present on new rooms) |

## Migration Plan

- Deploy server first (or together): old clients ignore unknown `taskCount`; new clients show counts when present.
- No DB migration.
- After apply: sync delta → main `lobby/rooms`; update lobby / pack-cards skills notes.

## Open Questions

- (none) — explore defaults locked: Material seat until SVG; click card+button; width 180; overflow 4; status pills as current; seats/tourists metric rows = set-row layout (D1–D3).

---

## Visual Spec (from mock)

**Mock:** `assets/lobby-room-card-mock.jpg` (light + dark).  
**Canon priority:** pack/task-set/map chrome (`--pack-card-*`, short badges, action-h 30, radius 12) **over** mock pixels. Mock supplies structure and copy.

### Structure (top → bottom)

1. Mini map preview (`position: relative` host for badge)
2. Status badge overlay — absolute, horizontally centered, near top of preview
3. Seats row (lead | declined «Место/Места/Мест» left | `occupied / maxSeats` right) — same columns as sets
4. Pale divider
5. Tourists row (lead | declined «Турист…» left | `n` right) — same columns as sets
6. Pack title (uppercase, center, 1–2 lines truncate)
7. Set rows ≤4 (lead | label left | count right) with pale divider **between** rows
8. Optional overflow «ещё {k}»
9. Pale divider above actions (border-top on actions)
10. Outline «Войти»

### Copy

| Element | Canon |
|---------|--------|
| Badge waiting / playing | `ОЖИДАНИЕ` / `ИГРА` |
| Seats label | `Место` / `Места` / `Мест` by **maxSeats** (sentence case, not UPPER) |
| Seats count | `{seats} / {maxSeats}` right-aligned |
| Tourists label | `Турист` / `Туриста` / `Туристов` by **touristsPerPlayer** (sentence case) |
| Tourists count | bare `n` right-aligned, no `+` |
| Pack title | packTitle uppercase |
| Set label | `Набор #{n}` (1-based among selected) |
| Overflow | `ещё {k}` |
| Join | `Войти` |

### Typography

| Role | font-size | weight | line-height | Align |
|------|-----------|--------|-------------|--------|
| badge | ~10px / 0.62rem | 600–700 | ~1.2 | center |
| seats/tourists label | ~0.68rem | 400–500 | 1.2 | left shared (same edge as set labels) |
| seats/tourists count | ~0.68–0.78rem | 600–700 | 1.2 | right shared (same edge as set counts) |
| title | ~0.78–0.9rem | 700 | 1.2 | center |
| set label | ~0.68rem | 400–500 | 1.2 | left shared |
| set count | ~0.68rem | 600 | 1.2 | right |
| overflow | ~0.62rem | 400 | 1.2 | as pack catalog |
| button | ~0.72rem | 500 | 1.2 | center |

### Размеры элементов

| Элемент | W | H / min-H | Pad / gap | Notes |
|---------|---|-----------|-----------|-------|
| card | **180** | grows (~280+) | radius **12** | `--pack-card-*` |
| preview | ~140–156 | square | pad ~10 top, x ~12 | MapGridPreview |
| status | max content | ~18–22 | top ~8, center X | overlay |
| seats / tourists rows | full | ~28–32 | lead **22**, pad-y **6** | grid `lead \| 1fr \| auto` like sets; **not** centered |
| divider | full | 1px | air ~3+3 | seats↔tourists; between sets; above actions |
| set row | full | pad-y 6 | lead 22 | icon ~14 |
| Войти | full content | **30** | actions pad ~8–10 | `--pack-card-action-h` |

### Цвета (canon CSS — not mock cream/navy)

#### Light

| Токен | Значение |
|-------|----------|
| card.bg | `#ffffff` |
| card.border.rest | `rgba(0,0,0,0.14)` |
| card.border.hover | `#212121` |
| card.shadow.rest / .hover | `0 1px 3px rgba(0,0,0,0.08)` / `0 4px 12px rgba(0,0,0,0.14)` |
| title.fg / seats / set labels / counts | `rgba(0,0,0,0.87)` |
| muted (tourists labels) | `rgba(0,0,0,0.7)` |
| divider | `rgba(0,0,0,0.08)` |
| action.outline | border token |
| badge pills | soft muted / pending sense (existing short-badge tokens) |

#### Dark

| Токен | Значение |
|-------|----------|
| card.bg | `#2a2a2a` (**not** mock `#1e252b`) |
| card.border.rest | `rgba(255,255,255,0.22)` |
| card.border.hover | `#bdbdbd` |
| card.shadow.rest / .hover | `0 1px 3px rgba(0,0,0,0.35)` / `0 4px 14px rgba(0,0,0,0.5)` |
| title.fg | `rgba(255,255,255,0.92)` |
| muted | `rgba(255,255,255,0.78)` |
| divider | `rgba(255,255,255,0.12)` |

### Icon ink / sizes

| Место | Size | Ink | Asset / Material |
|-------|------|-----|------------------|
| seats | 14–16 | title.fg | `lobby-room-card-seats.svg` / temp `event_seat` |
| tourists | 14–16 | muted or fg | `map-card-tourists.svg` |
| set row | 14 | title.fg | `task-set-card-tasks.svg` |
| badge | ~12 | badge.fg | soft pill (no wrong moderation icon required) |
| Войти | optional ~18 | action ink | text-only OK |

### Missing icons

| ID | Место | Suggested file | Temp Material | Блокер apply? |
|----|-------|----------------|---------------|---------------|
| I1 | seats | `lobby-room-card-seats.svg` | `event_seat` | нет |

### Hover / focus

| Target | Theme | Property | Default → Hover |
|--------|-------|----------|-----------------|
| card | light | border | rest → `#212121` |
| card | light | shadow | rest → shadow-hover |
| card | dark | border | rest → `#bdbdbd` |
| card | dark | shadow | rest → shadow-hover |
| card | both | scale | **none** |

**Не меняется при hover:** title, counts, dividers, map tiles, badge copy.

## Implementation touchpoints

**Server:** `MyRoom.refreshMetadata` (+ types if any); mocha assert `taskCount` on listing metadata after create.

**Client:** lobby room tile (`LobbyRoomCardTile`); `LobbyPage` wrap `.pack-card-grid`; i18n seats/tourists label plurals + count cells; `GameRoomMeta` type; vitest SC-LOBBY-25/36; mask `url("${…}")` for SVG leads.

**Skills (after apply / follow-up):** `work-with-lobby` (+ listing-cards topic), `pack-cards.md` note lobby consumes shared tokens; document metric-row rhythm.
