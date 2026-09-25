# Content pack store (`stores/content.ts`)

Read with the [core stores skill](SKILL.md) when changing pack HTTP, working
copy, add-task-set, staff lock/save, favorites, moderation take, or cascade
yellow. Pages: `Content*` under `work-with-pages`.

## Ownership

Setup store `content` owns:

- Unified packs list `listCatalog` → `GET /api/content/packs` (caller-facing
  `moderationStatus`, `openRequestType` `pack`|`task_set` for list routing
  SC-PACK-191/192, `isMine` / `isContributor` / `isFavorite`; SC-PACK-148…153)
- Favorites `starPack` / `unstarPack` → `POST …/favorite` \| `/unfavorite`
  (SC-PACK-154; registered non-anonymous; in-catalog only)
- Legacy collection HTTP (`listCollection` / add/remove) — **product UI removed**;
  route `content-collection` redirects to catalog (SC-PACK-164); do not rebuild
  collection-first UX
- `listMyModeration` / staff pending / preview (pack **and** map rows)
- Staff take / release `takeModerationRequest` / `releaseModerationRequest`
  (TTL = `CONTENT_LOCK_TTL_MS` 5 min; helper `moderationTakeHeldBy`; SC-PACK-161…163)
- Live pack detail (`isFavorite`, `moderationStatus`; legacy `inCollection` ignored for ACL)
- Creator / task-set-author working copy (`editorKind`: `creator` \|
  `task_set_author`) incl. **post-publish re-edit** via moderation (SC-PACK-156…159)
- Unified `submitPack` / add-task-set load+save+submit
- Edit lock `acquireEditLock` + heartbeat + release — **authors and staff**
  (staff blocked while open author request → `author_request_open`; SC-PACK-157)
- Staff `staffSavePack` + `isStaffEditSessionNavigation`
- Soft-unpublish pack/set (`inCatalog`; SC-PACK-120…132 / 139)
- Cascade yellow helpers (`cascadeGap*`)
- Cancel open request keeps working copy → author-facing `moderationStatus`
  `draft` on pack + catalog row (SC-PACK-175…179 / D11); hard-delete removes
  from `catalog` too (SC-PACK-177)
- Per-set `TaskSet.moderationStatus` from live/draft payloads (SC-PACK-171…174,
  187 — open marks only on request sets, not authorId fan-out)
- Never-live add-task-set ghost rows: `TaskSet.neverLive` for set author only
  on live pack (D19 / SC-PACK-188…190); row → `content-pack-add-task-set` Edit
- After Cancel of a never-live add-task-set cycle, GET add-task-set restores the
  retained revision `taskSets` (same selection as the ghost); `pendingRequestId`
  may be `null` — still bind the draft (D22 / SC-PACK-196). Put/submit reuse the
  cancelled revision id on the server (SC-PACK-197); client just forwards payload.

Open moderation = `pending`|`needs_revision`. Map API codes with
`contentErrorI18nKey` → `content.errors.*` (incl. `author_request_open`,
`moderation_taken`, `moderation_take_required`, `pack_unpublished`,
`last_published_task_set`).

**Removed product paths (do not reintroduce):** collection-first lobby/nav,
collect/uncollect on live pack, default-grant membership gates for add-task-set
(`not_in_collection`), non-staff «На модерации» header nav (SC-PACK-166 — filters
cover author pending/drafts). **Staff soft-unpublish remains in scope** — not
block.

## Cascade yellow (SC-PACK-81 / D43–D46; SC-PACK-126)

Server clears slots on card content change / delete (`cascadeNormalizeTasks`).
Client must **not** pre-clear slots before save (false `answers_dirty` under
D1′). Helpers: `cascadeGapTaskIds`, `markCascadeGaps`, `pruneCascadeGaps`,
`restoreCascadeGapsIfNeeded`, `taskHasCascadeGap` / `taskSetHasCascadeGap` →
page class `cascade-gap-outline` (not `bg-warning` row fill).

**Visible CSS required on both surfaces:** `ContentPackTasksPage` (task rows)
and `ContentPackEditorPage` (task-set rows). Class alone without scoped
`outline: 2px solid var(--q-warning)` is insufficient (SC-PACK-126 / D10).

## Question list slot chips (SC-PACK-127 / 130 / 134)

Every task/question list row MUST show answer slot chips (filled / empty):
`ContentPackTasksPage` (incl. live drill-in), `ContentStaffRequestPage`,
`ContentPackAddTaskSetPage`. Live pack page (`ContentPackPage`) shows **set
summary rows** only (count + difficulty 1/2/3) — slots appear after drill-in
(SC-PACK-130). Pattern: dense `q-chip` per slot + `slotEmpty` when no slots.
A filled slot MUST show the answer card’s text. For staff `task_set` preview,
server merges live `answerCards` into `previewPending` (SC-PACK-134). Add-task-set
answer **picker** tiles MUST be rounded `q-chip` (SC-PACK-133).

## Task-set author label + Tasks back (SC-PACK-135 / 136 / 182)

`TaskSet.authorDisplayName` drives `content.taskSetLabelFrom` on live / editor /
staff hub / Tasks heading. Live drill-in **omits** page «Вернуться» when App
breadcrumbs cover the path (SC-PACK-182); editor keeps `content.backToAnswers`.
Live pack page also omits «К наборам» (SC-PACK-181 — crumbs).

## Per-set moderation marks (SC-PACK-171…174 / 187)

`TaskSet.moderationStatus` (`pending` | `needs_revision` | `draft` | `live` |
`null`) appears on live/editor payloads for **that set's author and staff
only** — pack creator MUST NOT see foreign set marks solely as `createdBy`.
Open status applies **only** to sets belonging to the open request (revision /
lineage match — not fan-out by `changeAuthorId`). `ContentPackPage` shows
badges for pending / needs_revision / draft (not `live`).

## Never-live add-task-set ghost (SC-PACK-188…190) + cancel restore (D22)

Live pack payload may include `TaskSet.neverLive === true` rows for the **set
author only** (pending / needs_revision / draft after cancel). Staff and other
users do not see ghosts on live (queue for staff). Activating the ghost row
opens add-task-set Edit (amend), not live tasks drill-in.

**Cancel restore (SC-PACK-196/197):** GET `/api/content/packs/:id/add-task-set`
after Cancel of a never-live cycle MUST return that revision’s `taskSets` (not
an empty list with only live pack title/description). `pendingRequestId` /
`moderationStatus` may be null until resubmit — still hydrate the page from
`draft`. Do not special-case “no open request → blank form”. Server put/submit
reuse the cancelled revision id; client keeps load→edit→save→submit as today.

## Packs list open routing (SC-PACK-186 / 191 / 192)

`ContentPackSummary.openRequestType`: `pack` → list row opens Edit; `task_set`
→ live first (ghost row then Edit). Never-published / pack-level draft → Edit.
Clean in-catalog → live.

## Add-task-set moderation thread (SC-PACK-128)

`ContentPackAddTaskSetPage` MUST show open-request status
(`taskSetStatusMarks.pending` | `needs_revision`) and the same thread + reply
UX as the cards editor while the author’s `task_set` request is
`pending`|`needs_revision`.

## ACL surfaces (unify-content-lists-author-edit)

| Surface | Who | Store / route |
|---------|-----|----------------|
| Unified packs list | any auth | `listCatalog` → `content-catalog`; filters all/moderation/drafts/mine/favorites (identity-only filters disabled for guests) |
| Star / unstar | registered non-anonymous | `starPack` / `unstarPack` on in-catalog rows + detail |
| Unpublished / draft editor | creator | `loadDraft` / `saveDraft` / `submitPack` → `content-pack-edit` |
| Post-publish pack Edit | pack creator (non-staff) | acquire lock → working copy; `editorKind=creator`; submit reopens moderation |
| Post-publish task-set Edit | set `authorUserId` (not pack creator) | lock + draft; `editorKind=task_set_author`; cards read-only; only own sets |
| Add-task-set | verified non-anonymous + in-catalog | **no** collection membership (SC-PACK-164) → `content-pack-add-task-set` |
| Soft-unpublished non-staff | any | list badge / no live; trash OK for never-approved |
| Staff Edit | staff, **no** open author request | `acquireEditLock` → `loadStaffEdit` / `staffSavePack`; else disabled + `staffEditBlockedAuthorRequest` |
| Staff soft-unpublish | staff | `unpublishPack` / `republishPack` / task-set twins + confirms |
| Staff queue | staff | **take** before approve/needs_revision/cancel; take badge when held by other |

## UI contracts

- **Catalog (= primary «Наборы»):** App header / Lobby crumbs → `content-catalog`
  (not collection). Filters + status badges for draft / pending / needs_revision /
  unpublished only — **published / `in_catalog` rows show no badge** (SC-PACK-148/
  185). Star on in-catalog rows (stop click isolation). Row open: never-published
  or pack-level author draft/pending/needs_revision → edit; add-task-set-only
  (`openRequestType === 'task_set'`) → live first (SC-PACK-191); clean → live.
  Staff unpublish/republish + confirm SC-PACK-139. **No** list-chrome staff
  «Модерация» (SC-PACK-184 — App header); **no** non-staff my-moderation nav
  (SC-PACK-166).
- **Collection route:** redirect only (`content-collection` → `content-catalog`).
  Do not revive `ContentCollectionPage` as primary.
- **Live detail:** breadcrumbs replace «К наборам»; favorite star; author Edit
  (creator) / set-row Edit (task-set author); set-row `moderationStatus` badges
  for set author+staff; `neverLive` ghost → add-task-set Edit (GET restores
  cancelled never-live draft — SC-PACK-196); staff Edit gated
  by open author request; add-task-set beside «Задания» when verified+in-catalog
  (**not** `inCollection`); no Collect.
- **Editor:** `cardsReadOnly` when `editorKind === 'task_set_author'`; author may
  resubmit while own pending (incl. while staff holds take — SC-PACK-163).
  Cancel → keep working, list shows draft (not clear flags).
- **Staff hub / request / pack moderation / my-moderation:** Take / Release;
  terminal actions require held take; `moderationTakeHeldBy` + TTL. Staff
  Модерация entry is App header only. Pages omit `content.catalogNav` when
  App breadcrumbs cover Lobby / Модерация (SC-PACK-194/195).
- Support `change_pack` select: **in-catalog only** (`inCatalog !== false && hasLive`).
- Cascade / slot chips / add-task-set thread / delete-card confirms — unchanged
  SC-PACK-126…136 contracts above.
- No block/unblock UI.

## Anti-patterns

- Rebuilding collection-first nav / Collect on live / gating add-task-set on
  `inCollection` or `not_in_collection`.
- Showing non-staff «На модерации» header nav or list-chrome staff «Модерация»
  (App header + filters instead).
- Showing a green/`statusInCatalog` badge on published list rows.
- Staff Edit while author request open (must show blocked + tooltip / 409
  `author_request_open`).
- Approve / needs_revision / cancel without staff take (`moderation_take_required`).
- Letting task-set author edit answer cards or foreign sets; showing foreign
  set `moderationStatus` to pack creator alone.
- Clearing working flags on author cancel (must become draft, keep working).
- Treating add-task-set GET with `pendingRequestId === null` as a blank new set
  after never-live Cancel (must bind restored `draft.taskSets` — SC-PACK-196).
- Pre-clearing `slot.answerCardId` before save; conflating soft-unpublish with **block**.
- Releasing staff edit lock on cards↔tasks navigation (`isStaffEditSessionNavigation`).
- Jumping `v-if` caption hints (SC-PACK-119).
- Reviving page «К наборам» / live «Вернуться» / moderation `catalogNav` when App
  breadcrumbs cover the path (SC-PACK-181/182/194/195).
