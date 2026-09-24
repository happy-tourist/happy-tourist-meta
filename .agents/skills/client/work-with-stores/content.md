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
`republishPack` (soft-hide `inCatalog`; SC-PACK-120…125). Open moderation =
`pending`|`needs_revision` (no hard-reject). Map API codes with
`contentErrorI18nKey` → `content.errors.*` (incl. `pack_unpublished`).

**Removed (do not reintroduce):** personal drafts API, dual `submitAnswers` /
`submitTasks`, `draftStale` / `rebaseDraft`, task-set unpublish, foreign-pending
co-edit of full pack. **Staff soft-unpublish/republish is in scope** (D9) — not
the same as product **block**.

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
| Published non-staff | verified + inCollection + in-catalog | add-task-set only → `content-pack-add-task-set` |
| Soft-unpublished non-staff | any | collection gray row only; no live/Edit; trash OK |
| Staff Edit | moderator\|admin | `acquireEditLock` → `loadStaffEdit` / `staffSavePack` (no Submit); includes soft-unpublished |
| Staff soft-unpublish | staff | `unpublishPack` / `republishPack` (catalog + live chrome) |
| Staff queue | staff | `approveRequest` / `needsRevisionRequest` (`/needs-revision`) |

## UI contracts

- Collection: Edit for unpublished creator or staff; **no** row add-task-set (SC-PACK-118); trash + click isolation (no row `:to`); «На модерации» nav hidden for staff (SC-PACK-116); soft-unpublished (`hasLive && inCatalog === false`) row gray + «Снято с публикации», non-navigating for non-staff, trash remains (SC-PACK-121/125).
- Catalog: staff see soft-unpublished with badge + «Опубликовать снова»; public list hides them (server); staff «Снять с публикации» on in-catalog rows (SC-PACK-120).
- Live: staff Edit (+ lock) including soft-unpublished; staff unpublish/republish controls; non-staff add-task-set **beside «Задания»** only while in-catalog (SC-PACK-117); no full Edit for non-staff after publish (SC-PACK-53/106); non-staff deep-link → `pack_unpublished` / empty (SC-PACK-122).
- Staff Edit session: cards↔tasks keep lock / `staffEditTarget`; tasks persist → `staffSavePack`; unlock only on leave Edit (SC-PACK-115). Helper: `isStaffEditSessionNavigation`.
- Support `change_pack` select: **in-catalog only** (`inCatalog !== false && hasLive`).
- Editor hints: tooltips / reserved space — no jumping `v-if` captions (SC-PACK-119).
- Delete-card confirm: `deleteCardConfirmPublished` iff `pack.hasLive`.
- Status labels: prefer `content.statuses.needs_revision` (keep `rejected` alias in i18n for legacy rows).
- No block/unblock UI (block ≠ soft-unpublish).

## Anti-patterns

- Pre-clearing `slot.answerCardId` before save.
- Showing editable card/task lists to non-staff on published packs.
- Staff Submit-for-moderation control on own staff edits.
- Calling removed draftStale / dual-submit APIs; conflating soft-unpublish with **block**.
- Letting non-staff navigate/Edit soft-unpublished packs or open them via deep-link.
- Releasing staff edit lock on cards↔tasks navigation (use `isStaffEditSessionNavigation`; unlock only when leaving the Edit session).
- Jumping `v-if` caption hints for submit / questionNeedsSlot / tasksSave (use `q-tooltip` or always-reserved caption; SC-PACK-119).
- Header add-task-set on live pack or row playlist_add on collection (place beside «Задания»; SC-PACK-117/118).
- Showing «На модерации» nav to staff (SC-PACK-116).
- Offering soft-unpublished packs in Support `change_pack` select.
