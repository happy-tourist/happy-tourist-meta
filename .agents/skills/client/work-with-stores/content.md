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
`approveRequest` / author delete unpublished. Open moderation =
`pending`|`needs_revision` (no hard-reject). Map API codes with
`contentErrorI18nKey` → `content.errors.*`.

**Removed (do not reintroduce):** personal drafts API, dual `submitAnswers` /
`submitTasks`, `draftStale` / `rebaseDraft`, staff `unpublishPack` /
`republishPack` / task-set unpublish, foreign-pending co-edit of full pack.

## Cascade yellow (SC-PACK-81 / D43–D46)

Server clears slots on card content change / delete (`cascadeNormalizeTasks`).
Client must **not** pre-clear slots before save (false `answers_dirty` under
D1′). Helpers: `cascadeGapTaskIds`, `markCascadeGaps`, `pruneCascadeGaps`,
`restoreCascadeGapsIfNeeded`, `taskHasCascadeGap` / `taskSetHasCascadeGap` →
page class `cascade-gap-outline` (not `bg-warning` row fill).

## ACL surfaces (simplify-content-pack-editing)

| Surface | Who | Store / route |
|---------|-----|----------------|
| Unpublished editor | creator | `loadDraft` / `saveDraft` / `submitPack` → `content-pack-edit` |
| Published non-staff | verified + inCollection | add-task-set only → `content-pack-add-task-set` |
| Staff Edit | moderator\|admin | `acquireEditLock` → `loadStaffEdit` / `staffSavePack` (no Submit) |
| Staff queue | staff | `approveRequest` / `needsRevisionRequest` (`/needs-revision`) |

## UI contracts

- Collection: Edit for unpublished creator or staff; **no** row add-task-set (SC-PACK-118); trash + click isolation (no row `:to`); «На модерации» nav hidden for staff (SC-PACK-116).
- Live: staff Edit (+ lock); non-staff add-task-set **beside «Задания»** (not header; SC-PACK-117); no full Edit for non-staff after publish (SC-PACK-53/106).
- Staff Edit session: cards↔tasks keep lock / `staffEditTarget`; tasks persist → `staffSavePack`; unlock only on leave Edit (SC-PACK-115). Helper: `isStaffEditSessionNavigation`.
- Editor hints: tooltips / reserved space — no jumping `v-if` captions (SC-PACK-119).
- Delete-card confirm: `deleteCardConfirmPublished` iff `pack.hasLive`.
- Status labels: prefer `content.statuses.needs_revision` (keep `rejected` alias in i18n for legacy rows).
- No block/unblock UI; no unpublish/republish chrome.

## Anti-patterns

- Pre-clearing `slot.answerCardId` before save.
- Showing editable card/task lists to non-staff on published packs.
- Staff Submit-for-moderation control on own staff edits.
- Calling removed draftStale / unpublish / dual-submit APIs.
- Releasing staff edit lock on cards↔tasks navigation (use `isStaffEditSessionNavigation`; unlock only when leaving the Edit session).
- Jumping `v-if` caption hints for submit / questionNeedsSlot / tasksSave (use `q-tooltip` or always-reserved caption; SC-PACK-119).
- Header add-task-set on live pack or row playlist_add on collection (place beside «Задания»; SC-PACK-117/118).
- Showing «На модерации» nav to staff (SC-PACK-116).
