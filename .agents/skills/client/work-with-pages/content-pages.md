# Content pages (packs + maps)

Read with the [core pages skill](SKILL.md) for routes/shell. Store contracts:
`work-with-stores/content.md` and `maps.md`. Tile CSS: `work-with-styles/pack-cards.md`.

## Routes

| Path | Name | Page | Notes |
|------|------|------|-------|
| `/content/packs` | `content-catalog` | `ContentCatalogPage` | Unified packs list (filters/statuses/star; staff soft-unpublish; **no** list-chrome staff «Модерация» — App header SC-PACK-184; published/`in_catalog` rows **no** status badge SC-PACK-185) |
| `/content/collection` | `content-collection` | — | **redirect** → `content-catalog` (SC-PACK-164; do not revive collection page) |
| `/content/my-moderation` | `content-my-moderation` | `ContentMyModerationPage` | Deep-link/API retained; **no** non-staff header nav; **no** `catalogNav` (SC-PACK-194) |
| `/content/maps` | `content-maps` | `MapsListPage` | List filters/statuses + Create; staff Модерация in App header (SC-MAP-43); published rows **no** badge (SC-MAP-45) |
| `/content/maps/:id/edit` | `content-map-edit` | `MapEditorPage` | View = author+seats (no paint); edit = palette; crumbs replace «К картам» (SC-MAP-44/46/47); staff `?staff=1` (blocked if author request open) |
| `/content/packs/new` | `content-pack-new` | `ContentPackCreatePage` | Create → working-copy editor; verify modal if ineligible |
| `/content/packs/:id` | `content-pack` | `ContentPackPage` | Live + star + author/staff Edit gates + add-task-set beside «Задания» (verified+in-catalog; **no** collect) |
| `/content/packs/:id/edit` | `content-pack-edit` | `ContentPackEditorPage` | Creator **or** `editorKind` author re-edit (`submitPack`) or staff (`staffSavePack` + lock) |
| `/content/packs/:id/add-task-set` | `content-pack-add-task-set` | `ContentPackAddTaskSetPage` | Post-publish add-only new task set (**no** collection gate) |
| `/content/packs/:id/tasks/:taskSetId` | `content-pack-tasks` | `ContentPackTasksPage` | Nested task-set (unpublished / author / staff session) |
| `/content/packs/:id/moderation` | `content-pack-moderation` | `ContentPackModerationPage` | Author thread (also embedded on edit); **no** `catalogNav` (SC-PACK-194) |
| `/content/staff` | `content-staff` | `ContentStaffPage` | `requiresStaff`; queue + **Take** (pack\|map); **no** `catalogNav` (SC-PACK-195) |
| `/content/staff/requests/:id` | `content-staff-request` | `ContentStaffRequestPage` | Take before Approve / needs-revision / cancel; **no** `catalogNav` |
| `/content/staff/requests/:id/tasks` | `content-staff-request-tasks` | `ContentStaffTasksPage` | Redirect → request hub (legacy); **no** `catalogNav` |

## Packs UI canon

- Catalog: ~180×260 `PackListCardTile` grid (SC-PACK-228/249…254); wire `:task-sets-preview="item.taskSetsPreview ?? []"` and soft-unpub `:muted="Boolean(item.hasLive && item.inCatalog === false)"` (whole-card opacity — not set-row muted); star TL / status top (revise short «ДОРАБОТАТЬ» red outline via `taskSetCardBadge.needs_revision` + `pack-list-status-badge--revise`); uppercase title + description + **published-only** set preview ≤4 + «ещё K» (omit soft-unpub/neverLive; display ordinal among published; set-row ink = title.fg); bottom outline+icon actions gated by `hasCatalogCardActions` (omit empty `#actions`); filters; **no** published badge; **no** list staff Модерация; row open: never-published / pack-level draft\|pending\|needs_revision → Edit; add-task-set-only → live first via `openRequestType` (SC-PACK-186/191/192).
- Live pack: task-set summary via `PackTaskSetCardTile` (SC-PACK-229/239…248; denser than catalog; width ~150–160, min-height ~206; `#{n}` ordinal title, no author; pale multi-row dividers; lead/label/count grid; retuned title→stats ~7 / divider air ~4–6 SC-PACK-247; themed hover border+shadow no scale SC-PACK-246; custom SVG total+badge icons via CSS mask SC-PACK-248; slim ~28–32px `dense` outline actions; short card Снять/Вернуть). Gate `#actions` with `hasTaskSetCardActions` (omit empty chrome for viewers). Status badges: `pack-task-set-status-badge` + `pack-task-set-status-icon--*` SVG mask + tone classes (`--pending` / `--muted`; **not** Quasar `color="warning"`/`grey`); pages use `*ModerationBadgeToneClass` / `*ModerationBadgeIconClass` helpers. Crumbs replace «К наборам»; set-row `moderationStatus` for set author+staff; `neverLive` ghost → add-task-set Edit (SC-PACK-188…190).
- Editor: answers framed CSV panel + `PackAnswerCardTile` (SC-PACK-210…227); editor task-set cards also `PackTaskSetCardTile` SC-PACK-229/239…248 (+ cascade-gap; slim `dense` outline; `hasEditorTaskSetCardActions` for `#actions`; same spacing/hover/SVG icons as live); **published + staff → `enterStaffEdit` / staff-save before creator path** (SC-PACK-231; never-published staff creator keeps Submit SC-PACK-232); author «На модерацию» requires dirty vs session baseline (`lib/editorDirty`) + minima (SC-PACK-234).
- AddTaskSet / Tasks: tasks CSV via `PackTasksCsvControls` (**hidden on live `viewOnly`** SC-PACK-235; staff Edit keeps CSV SC-PACK-236); `PackTaskTile` / compose slots use `.peek-slot-like` (SC-PACK-237/238); Tasks heading + staff request set headings use ordinal `taskSetLabel` «Набор заданий #{n}» (**no** author/coauthor — SC-PACK-135); live: no «Вернуться» — crumbs; editor keeps back-to-answers.
- Staff/author moderation pages: **no** `catalogNav` (SC-PACK-194/195).
- Store: `stores/content`; cancel→draft; `TaskSet.moderationStatus` / `neverLive`; `openRequestType`; SC-PACK-148…238.

## Maps UI canon

- List: `MapListCardTile` grid (SC-MAP-55); author draft/pending/needs_revision → Edit (SC-MAP-50); clean published → View; no published badge; staff Edit only when `hasLive` (`?staff=1`).
- Editor: title-row Edit (SC-MAP-51); board-comparable field + under-map palette labels (SC-MAP-54); centered column + usable seats (SC-MAP-56); paint block + anti-stale quiet save (SC-MAP-53); `MapGridPreview`; **published + staff → staffMode / staff-save** (SC-MAP-62; never-published keeps Submit SC-MAP-63); author Submit dirty gate (SC-MAP-65).
- Store: `stores/maps`; crumbs inside `q-page-container`; Lobby crumb → lobby SC-BRAND-20; cancel→draft; SC-MAP-41…65.
