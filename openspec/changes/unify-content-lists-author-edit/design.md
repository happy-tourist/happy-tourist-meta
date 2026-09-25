## Context

See `proposal.md`. Runtime уже содержит unified lists … chrome FIX (D13/D20/D21). Bug: after Cancel of never-live add-task-set, `getAddTaskSet` ignores the cancelled request revision and returns empty `taskSets` with live pack title/description, while `neverLiveAuthorTaskSets` still loads that revision for the ghost row.

Explore closed: D1–D22; cancel add-task-set draft restore.

## Goals / Non-Goals

**Goals**

- Prior goals — landed
- After Cancel of never-live add-task-set, author Edit (GET add-task-set) restores the submitted working snapshot (questions/slots), same source of truth as the ghost row
- put/submit after cancel MAY reuse that cancelled revision id (no wipe)

**Non-Goals**

- Changing Cancel semantics for pack-level / live set re-edit `/draft` (already keeps workingRevisionId)
- Changing ghost visibility rules (D19)
- Client chrome / maps / branding in this FIX

## Decisions

### D1–D21 — Prior (unchanged summary)

As before. Runtime already landed for those decisions (incl. D13 page-container crumbs, D20 moderation chrome, D21 narrow burger fold).

### D22 — Cancelled add-task-set draft restores request revision

- **Choice:** `getAddTaskSet` MUST load draft from the author’s latest **retained** task_set cycle when there is no open request: same selection rules as `neverLiveAuthorTaskSets` (cancelled counts; superseded by a newer approved cycle does not). Response MUST include that revision’s `taskSets` (and revision title/description as today for add-task-set payload shape). MUST NOT fall back to `{ taskSets: [] }` solely because status is cancelled. `putAddTaskSet` / `submitAddTaskSet` when no open request SHOULD reuse that cancelled revision id when writing (overwrite children), so the author continues the same working copy. Cancel itself stays status-only (no delete of revision children).
- **Why:** Cancel already keeps the revision in DB; list ghost shows it; Edit GET only looked at open → empty draft with live pack title/description — user expectation and SC-PACK-175/179 violated for add-task-set path.
- **Alternatives:** Copy cancelled revision into a new staged id on cancel — unnecessary; status-only cancel already preserves payload.
- **Layers:** server `lib/content.ts` (`getAddTaskSet`, helper shared with never-live ghosts if practical; put/submit); mocha SC-PACK-196…; client only if store ignores restored draft.

## Risks / Trade-offs

- **[Risk] Dual view of same pack** → status per caller.
- **[Risk] Wrong cancelled cycle restored after later approve** → reuse neverLiveAuthorTaskSets supersede-by-approved rule.
- **[Risk] put without open creates orphan revisions** → prefer reuse cancelled revision id.

## Migration Plan

1. Server: getAddTaskSet (+ put/submit) restore/reuse cancelled add-task-set revision; mocha.
2. Client smoke if needed; lint/typecheck.
3. Rollback: revert deploy.

## Open Questions

None — restore cancelled never-live add-task-set snapshot on Edit; live re-edit `/draft` already covered.
