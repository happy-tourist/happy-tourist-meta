# Content pack store (`stores/content.ts`)

Read with the [core stores skill](SKILL.md) when changing pack HTTP, draft
flags, cascade yellow, or dual submit. Pages: `Content*` under `work-with-pages`.

## Ownership

Setup store `content` owns catalog / collection (trash confirm) /
`listMyModeration` (`GET /api/content/my-moderation`) / live (`inCollection` +
`pending*AuthorId` = **open** pending|rejected; patch membership on
collect/remove) / draft / `submitAnswers` / `submitTasks` / author delete
unpublished / moderation + staff (pending|rejected; approve-from-rejected;
`tasksOnly` / `answersActionsAvailable`; nested → hub or queue). Map API codes
with `contentErrorI18nKey` → `content.errors.*`. Never invent a single
`/submit` path; do not call deprecated `submitPack` from pages.

## Cascade yellow (SC-PACK-81 / D43–D46)

Server clears slots on card content change / delete (`cascadeNormalizeTasks`).
Client must **not** pre-clear slots before `saveDraft` (false `answers_dirty`
under D1′).

| Helper / state | Role |
|----------------|------|
| `taskIdsReferencingCard(content, cardId)` | Snapshot task ids that reference a card **before** delete/content change |
| `cascadeGapTaskIds` | Tasks with empty slots after cascade — yellow until filled |
| `markCascadeGaps(taskIds, draft)` | After save: add ids that still have empty slots |
| `pruneCascadeGaps(draft)` | Drop yellow when slots refilled or task gone (call after autosave flush) |
| `restoreCascadeGapsIfNeeded(draft)` | On `loadDraft`: if `answersDirty` and in-memory gaps empty, seed from dirty+empty slots (F5) |
| `taskHasCascadeGap` / `taskSetHasCascadeGap` | Page class `cascade-gap-outline` (yellow border, **not** `bg-warning` row fill) on task row / task-set list item |

Editor flow: content change or delete → `taskIdsReferencingCard` → save (no
slot wipe) → `markCascadeGaps`. Tasks page: show answer slots on each row;
prune after flush.

## Publish UX (SC-PACK-85…96)

| Helper / state | Role |
|----------------|------|
| `draftStale` | From GET draft / rebase — banner + «Подтянуть» on cards/tasks editors |
| `unpublishPack` / `republishPack` | Staff collection: clear/restore catalog live; patch `hasLive` / `unpublishedByStaff` / `hasLastLive` |
| `unpublishLiveTaskSet(packId, liveTaskSetId)` | Staff cards editor; API needs **live** id (draft ids remapped — match fingerprint) |
| `rebaseDraft` | Pull live base; keep local edits as new ids |

- Status subtitle: answers approved + `hasLive` → `taskSetStatusMarks.published` («Опубликовано»); tasks approved without catalog live → `approved` («Одобрено»).
- Live pack + staff tasks preview: slot chips (`slotLabel`), not `slotsCount` / joined text alone.
- Collection: hide Edit for non-staff when `unpublishedByStaff`; badge `unpublishedByStaff` ≠ `draftOnly`.

## UI contracts (pages mirror these)

- Delete-card confirm: `content.deleteCardConfirmPublished` iff `pack.hasLive`,
  else `content.deleteCardConfirm`.
- Three-phase page subtitles / list marks: **`content.taskSetStatusMarks.*`**
  (`pending` / `rejected` / `needs_moderation` / `approved` / `published`). Keep
  `statusCycle*` as aliases only — do not re-wire pages to them.
- Live Edit from `inCollection` (+ open-author exception; hide if `blocked` or
  foreign open). Collection trash + click isolation (no row `:to`).
- **D1′**/D5′ on open=pending|rejected mirrored in UI; cascade-only slot clears
  are not a tasks edit under that lock.
- Staff: pending|rejected / `tasksOnly` / `answersActionsAvailable`
  (approve-from-rejected; map `not_approvable`; hide answers approve when
  tasks-only; nested → hub or queue); no block UI; pack unpublish/republish +
  task-set unpublish when ≥2 live sets.

## Anti-patterns

- Pre-clearing `slot.answerCardId` in the page before save (removed
  `clearSlotsForCard` pattern).
- Letting cascade yellow die on F5 when answers stay dirty with empty slots
  (must restore via `restoreCascadeGapsIfNeeded`).
- Using `statusCycle*` for new page subtitles instead of `taskSetStatusMarks.*`.
