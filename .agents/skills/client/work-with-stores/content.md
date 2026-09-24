# Content pack store (`stores/content.ts`)

Read with the [core stores skill](SKILL.md) when changing pack HTTP, working
copy, add-task-set, staff lock/save, or cascade yellow. Pages: `Content*` under
`work-with-pages`.

## Ownership

Setup store `content` owns catalog / collection (trash confirm) /
`listMyModeration` / live (`inCollection`; patch membership on collect/remove) /
creator working-copy draft (`GET|PUT /api/content/packs/:id/draft`) / unified
`submitPack` / add-task-set load+save+submit / staff `acquireEditLock` +
`staffSavePack` + release / helper `isStaffEditSessionNavigation` /
`needsRevisionRequest` + `cancelRequest` +
`approveRequest` / author delete unpublished / staff `unpublishPack` +
`republishPack` (pack soft-hide `inCatalog`; SC-PACK-120…125 / 129) /
`unpublishTaskSet` + `republishTaskSet` (set soft-hide; SC-PACK-131/132; reject
last published set → `last_published_task_set`). Open moderation =
`pending`|`needs_revision` (no hard-reject). Map API codes with
`contentErrorI18nKey` → `content.errors.*` (incl. `pack_unpublished`,
`last_published_task_set`).

**Removed (do not reintroduce):** personal drafts API, dual `submitAnswers` /
`submitTasks`, `draftStale` / `rebaseDraft`, foreign-pending co-edit of full
pack. **Staff soft-unpublish/republish of pack and task set is in scope**
(D9 / D13) — not the same as product **block**; not hard-delete of sets.

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
A filled slot MUST show the answer card’s text (not a generic «заполнен» when
the card resolves). For staff `task_set` preview, server merges live
`answerCards` into `previewPending` (SC-PACK-134). Add-task-set answer
**picker** tiles MUST be rounded `q-chip` like Tasks (SC-PACK-133), not
rectangular `q-btn`.

## Task-set author label + Tasks back (SC-PACK-135 / 136)

`TaskSet.authorDisplayName` (displayName else email local-part) drives
`content.taskSetLabelFrom` on live / editor / staff hub / Tasks heading.
Live drill-in back uses `content.back` («Вернуться»), not pack title;
editor keeps `content.backToAnswers`.

## Add-task-set moderation thread (SC-PACK-128)

`ContentPackAddTaskSetPage` MUST show open-request status
(`taskSetStatusMarks.pending` | `needs_revision`) and the same thread + reply
UX as the cards editor while the author’s `task_set` request is
`pending`|`needs_revision`. Load via `loadModeration` / `postModerationMessage`
(prefer not inventing a parallel messages payload). Hide the thread block when
there is no open own request (fresh create / foreign pending).

## ACL surfaces (simplify-content-pack-editing)

| Surface | Who | Store / route |
|---------|-----|----------------|
| Unpublished editor | creator | `loadDraft` / `saveDraft` / `submitPack` → `content-pack-edit` |
| Published non-staff | verified + inCollection + in-catalog | add-task-set only → `content-pack-add-task-set` |
| Soft-unpublished non-staff | any | collection gray row only; no live/Edit; trash OK |
| Staff Edit | moderator\|admin | `acquireEditLock` → `loadStaffEdit` / `staffSavePack` (no Submit); includes soft-unpublished |
| Staff soft-unpublish pack | staff | `unpublishPack` / `republishPack` (catalog + **collection** + live; confirm on unpublish; SC-PACK-129) |
| Staff soft-unpublish set | staff | `unpublishTaskSet` / `republishTaskSet` (Editor/live row + inside Tasks; confirm on unpublish; disable last published; SC-PACK-131/132) |
| Staff queue | staff | `approveRequest` / `needsRevisionRequest` (`/needs-revision`); **no** pack/set unpublish |

## UI contracts

- Collection: Edit for unpublished creator or staff; **no** row add-task-set (SC-PACK-118); trash + click isolation (no row `:to`); «На модерации» nav hidden for staff (SC-PACK-116); soft-unpublished (`hasLive && inCatalog === false`) row gray + «Снято с публикации», non-navigating for non-staff, trash remains (SC-PACK-121/125); **staff** pack unpublish/republish on row with confirm (SC-PACK-129).
- Catalog: staff see soft-unpublished with badge + «Опубликовать снова»; public list hides them (server); staff «Снять с публикации» + confirm on in-catalog rows (SC-PACK-120/129).
- Live: staff Edit (+ lock) including soft-unpublished pack; staff pack + **task-set** unpublish/republish (confirm on unpublish); set **summary rows** + drill-in (SC-PACK-130); soft-unpublished set gray / no enter for non-staff (SC-PACK-132); non-staff add-task-set **beside «Задания»** only while in-catalog (SC-PACK-117); no full Edit for non-staff after publish (SC-PACK-53/106); non-staff deep-link → `pack_unpublished` / empty (SC-PACK-122).
- Staff Edit session: cards↔tasks keep lock / `staffEditTarget`; tasks persist → `staffSavePack`; unlock only on leave Edit (SC-PACK-115). Helper: `isStaffEditSessionNavigation`.
- Support `change_pack` select: **in-catalog only** (`inCatalog !== false && hasLive`).
- Editor hints: tooltips / reserved space — no jumping `v-if` captions (SC-PACK-119).
- Cascade yellow: `cascade-gap-outline` CSS on **Editor task-set rows** and Tasks task rows (SC-PACK-126).
- Slot chips on every question list (staff hub, add-task-set, tasks / live drill-in) (SC-PACK-127); not inline on live summary (SC-PACK-130).
- Add-task-set: status + moderation thread + reply while open request (SC-PACK-128); answer picker = rounded chips (SC-PACK-133).
- Delete-card confirm: `deleteCardConfirmPublished` iff `pack.hasLive`.
- Status labels: prefer `content.statuses.needs_revision` / `taskSetStatusMarks.needs_revision` (keep `rejected` alias in i18n for legacy rows).
- No block/unblock UI (block ≠ soft-unpublish).

## Anti-patterns

- Pre-clearing `slot.answerCardId` before save.
- Showing editable card/task lists to non-staff on published packs.
- Staff Submit-for-moderation control on own staff edits.
- Calling removed draftStale / dual-submit APIs; conflating soft-unpublish with **block**.
- Letting non-staff navigate/Edit soft-unpublished packs or soft-unpublished task sets or open them via deep-link.
- Unpublishing the only published task set (client must disable; server 409 `last_published_task_set`).
- Unpublishing pack/set without confirm; requiring confirm on republish.
- Showing pack/set unpublish in staff moderation queue.
- Expanding all task questions inline on live (use summary + drill-in; SC-PACK-130).
- Releasing staff edit lock on cards↔tasks navigation (use `isStaffEditSessionNavigation`; unlock only when leaving the Edit session).
- Jumping `v-if` caption hints for submit / questionNeedsSlot / tasksSave (use `q-tooltip` or always-reserved caption; SC-PACK-119).
- Header add-task-set on live pack or row playlist_add on collection (place beside «Задания»; SC-PACK-117/118).
- Showing «На модерации» nav to staff (SC-PACK-116).
- Offering soft-unpublished packs in Support `change_pack` select.
- Applying `cascade-gap-outline` class on Editor without the matching scoped CSS (SC-PACK-126).
- Question lists without slot chips on staff hub / add-task-set / tasks drill-in (SC-PACK-127).
- Add-task-set amend without status + moderation thread while open (SC-PACK-128).
- Rectangular `q-btn` answer tiles on AddTaskSet (use `q-chip`; SC-PACK-133).
