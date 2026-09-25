# Content maps store (`stores/maps.ts`)

Read with the [core stores skill](SKILL.md) when changing UGC map HTTP, paint
helpers, staff lock/save, soft-unpublish, or list statuses/filters. Pages:
`MapsListPage` / `MapEditorPage`; shared staff hub still uses `content` store for
queue/preview/take (type `map`). Component: `MapGridPreview.vue`.

## Ownership

Setup store `maps` owns:

- `listMaps` / `createMap` → `GET|POST /api/content/maps`
  (row `moderationStatus`, `authorRequestOpen` — SC-MAP-31…36)
- live + draft `loadLiveMap` / `loadDraft` / `saveDraft` (quiet save while
  never-published; **author post-publish re-edit** via working copy + moderation)
- `submitMap` / `loadModeration` / `postModerationMessage`
- staff `acquireEditLock` / `refreshEditLock` / `releaseEditLock` /
  `loadStaffEdit` / `staffSaveMap` — staff blocked while `authorRequestOpen`
  (`author_request_open`; SC-MAP-36)
- `unpublishMap` / `republishMap` (soft-hide `inCatalog`; cascade-cancel + RU mail)
- `deleteUnpublishedMap` → `POST /api/content/map/delete` (never-approved only)
- `cancelRequest` → shared cancel; keeps working → author-facing
  `moderationStatus` `draft` + list row update (SC-MAP-41/42 / D11)
- Known errors include `author_request_open`, `moderation_taken`,
  `moderation_take_required` (take HTTP lives on `content` store)

Open moderation = `pending`|`needs_revision`. Map API codes with
`mapsErrorI18nKey` → `maps.errors.*` (not `content.errors.*`).

**Out of scope (do not reintroduce):** map collection / add-to-collection;
wiring map id into tourist-room create (SC-MAP-28); pack cascade yellow / slots;
duplicating staff take/release HTTP here (use `content.takeModerationRequest`).

## Shared queue vs maps store

| Surface | Store |
|---------|--------|
| Author open requests (list filters) / staff pending / staff take\|release\|approve\|needs_revision\|cancel\|preview | `stores/content.ts` (`type: 'map'`, `mapId`, optional `grid` / seats / `takenBy`; `StaffPreview.map`) |
| Maps list filters + status badges / paint editor / map draft HTTP / map staff Edit / soft-unpublish / author delete | `stores/maps.ts` |

Do **not** fold map CRUD into `content.ts`. Do **not** duplicate staff queue
HTTP in `maps` beyond cancel/preview helpers the editor already needs.

## Grid + paint (SC-MAP-04)

- Size: `MAP_SIZE=10`, `MAP_CELLS=100`; alphabet `MAP_CELL`: `.` hole, `1` start,
  `*` task, `7` finish.
- Pure helpers: `emptyMapGrid`, `countStarts`, `paintCell`, `paintToolToCell`,
  `clampSeatCount` (players / touristsPerPlayer **1…4**).
- Palette tools: `start` | `task` | `finish`. Click same type → clear to hole;
  other type → replace.
- Submit gate: starts ≥ players × touristsPerPlayer (client UX + server reject).

## UI contracts

- **List:** filters all / moderation / drafts / mine (identity-only disabled for
  guests); badges for draft / pending / needs_revision / unpublished only —
  **published / `in_catalog` rows show no badge** (SC-MAP-45); mini
  `MapGridPreview` + author + `players×tourists`; Create at top; **no** collect;
  **no** list-chrome staff «Модерация» (SC-MAP-43 — App header).
  Row open: never-published → Edit without `?edit` (SC-MAP-49); author
  draft/pending/needs_revision on a live map → Edit with `query.edit=1`
  (SC-MAP-50); clean published → View-first (SC-MAP-46). Staff stays View-first.
- **Editor view:** author display + seat config; Enter Edit control in the
  **title row** (`map-view-edit`), not inside `map-view-meta` (SC-MAP-51);
  **no** paint tools / seat selects (SC-MAP-46/47). App breadcrumbs replace
  «К картам» and sit **inside** `q-page-container` (SC-MAP-44/52 / SC-BRAND-17).
- **Editor boot:** never-published or creator author-work
  (draft/pending/needs_revision) → Edit + lock; clean published → View (SC-MAP-46/49/50).
- **Editor edit:** interactive preview + palette + seats; quiet autosave while
  creator-editable; **author may re-edit published** via lock + draft → moderation
  (mirrors packs); staff `?staff=1` lock session when no open author request
  (else blocked tooltip `maps.staffEditBlockedAuthorRequest`). Cancel → draft,
  keep working (do not `clearWorkingFlags`).
- Unpublish confirm warns open requests cancelled (`maps.unpublishConfirm`).
- Errors: page `q-banner` on `maps.error`; prefer `mapsErrorI18nKey` when set.

## Tests

- Paint pure: `src/stores/__tests__/maps.paint.test.ts` (SC-MAP-04).
- Pages/UI: `src/pages/__tests__/ContentMaps.test.ts` (list filters/statuses +
  view meta / title-row Edit / author pending→Edit / crumbs /
  cancel→draft SC-MAP-41…52 + SC-MAP-06…08, 14, 17, 21, 24–25, 29–36 + submit
  starts gate). App crumbs inside `q-page-container`: `AppHeaderChrome` SC-MAP-52 /
  SC-BRAND-17.
- Server twin: `test/zz-contentMaps.test.ts` (mocha) — do not mix stacks.

See meta `work-with-test` / `work-with-pages` / `work-with-localization` (`maps.*`).
