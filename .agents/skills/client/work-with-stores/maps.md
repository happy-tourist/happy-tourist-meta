# Content maps store (`stores/maps.ts`)

Read with the [core stores skill](SKILL.md) when changing UGC map HTTP, paint
helpers, staff lock/save, or soft-unpublish. Pages: `MapsListPage` /
`MapEditorPage`; shared staff hub still uses `content` store for queue/preview
(type `map`). Component: `MapGridPreview.vue` (list + editor + staff preview).

## Ownership

Setup store `maps` owns:

- `listMaps` / `createMap` → `GET|POST /api/content/maps`
- live + draft `loadLiveMap` / `loadDraft` / `saveDraft` (quiet save while
  never-published)
- `submitMap` / `loadModeration` / `postModerationMessage`
- staff `acquireEditLock` / `refreshEditLock` / `releaseEditLock` /
  `loadStaffEdit` / `staffSaveMap`
- `unpublishMap` / `republishMap` (soft-hide `inCatalog`; cascade-cancel open
  map requests + RU mail on server — SC-MAP-21…27)
- `deleteUnpublishedMap` → `POST /api/content/map/delete` (never-approved only)
- `cancelRequest` → shared staff cancel endpoint (same as packs)
- `loadStaffPreview` when editor needs request preview by id

Open moderation = `pending`|`needs_revision` (no hard-reject). Map API codes
with `mapsErrorI18nKey` → `maps.errors.*` (not `content.errors.*`).

**Out of scope (do not reintroduce):** map collection / add-to-collection;
wiring map id into tourist-room create (SC-MAP-28); pack cascade yellow / slots.

## Shared queue vs maps store

| Surface | Store |
|---------|--------|
| Author «На модерации» list / staff pending list / staff request approve|needs_revision|cancel|preview | `stores/content.ts` (`type: 'map'`, `mapId`, optional `grid` / seats on rows; `StaffPreview.map`) |
| Maps list / paint editor / map draft HTTP / map staff Edit session / map soft-unpublish / author delete | `stores/maps.ts` |

Do **not** fold map CRUD into `content.ts`. Do **not** duplicate staff queue
HTTP in `maps` beyond cancel/preview helpers the editor already needs.

## Grid + paint (SC-MAP-04)

- Size: `MAP_SIZE=10`, `MAP_CELLS=100`; alphabet `MAP_CELL`: `.` hole, `1` start,
  `*` task, `7` finish.
- Pure helpers next to the store: `emptyMapGrid`, `countStarts`, `paintCell`,
  `paintToolToCell`, `clampSeatCount` (players / touristsPerPlayer **1…4**).
- Palette tools: `start` | `task` | `finish`. Click same type → clear to hole;
  other type → replace.
- Submit gate: starts ≥ players × touristsPerPlayer (client UX + server reject).

## UI contracts

- List: mini `MapGridPreview` + author + `players×tourists`; Create at top; **no**
  collect/uncollect (SC-MAP-06…08).
- Editor: interactive preview + bottom palette + seat controls; quiet autosave
  while creator-editable; after first approve non-staff is view-only (SC-MAP-17);
  staff `?staff=1` lock session mirrors packs.
- Unpublish confirm must warn open requests cancelled (`maps.unpublishConfirm`).
- Errors: page `q-banner` on `maps.error`; prefer `mapsErrorI18nKey` when set.

## Tests

- Paint pure: `src/stores/__tests__/maps.paint.test.ts` (SC-MAP-04).
- Pages/UI: `src/pages/__tests__/ContentMaps.test.ts` (SC-MAP-06…08, 14, 17,
  21, 24–25, 29–30 + submit starts gate).
- Server twin: `test/zz-contentMaps.test.ts` (mocha) — do not mix stacks.

See meta `work-with-test` / `work-with-pages` / `work-with-localization` (`maps.*`).
