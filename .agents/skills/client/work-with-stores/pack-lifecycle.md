# Content pack draft lifecycle (`stores/content.ts`)

Read with [content.md](content.md) when changing author draft activation,
persisted Submit dirty, pending→draft, hard-delete, or add-task-set pre-submit
draft. Pages: `ContentPackEditorPage`, `ContentPackAddTaskSetPage`,
`ContentPackTasksPage`, `ContentCatalogPage`.

## Store ownership (D9–D14 / SC-PACK-234/264…275)

- Pre-submit / after-Cancel never-live draft: quiet
  `saveAddTaskSet({ quiet: true })` persists incomplete payload; GET restores
  `moderationStatus: draft` + `pendingRequestId` + server
  `hasUnsubmittedChanges` / `canHardDelete`; Cancel never-live → `draft`
  (keeps `pendingRequestId`, dirty=true). `discardAddTaskSetDraft` →
  `POST …/add-task-set/discard` deletes the author’s retained never-live cycle
  when never published (draft|pending|needs_revision; live pack/published
  siblings untouched — SC-PACK-266/274/275). After quiet save, patch catalog
  row to `draft` + `openRequestType: 'task_set'` (SC-PACK-268) and sync
  dirty/delete flags from response. Author-only drafts filter / neverLive row.
  Page: `lib/editorDirty` fingerprint = **autosave dedupe only** — Submit uses
  server `hasUnsubmittedChanges` (D10/D13).
- Pack draft activation (D12 / SC-PACK-269/270): create may open a hidden shell;
  first semantic save activates author-only `draft` in list/drafts. Trust list
  payloads — do not invent local catalog rows for untouched shells.
- Persisted dirty + status transitions (D13–D14 / SC-PACK-234/271…273): store
  refs `hasUnsubmittedChanges` / `canHardDelete` from draft / add-task-set /
  save / submit responses. Author save on `pending` → response `draft` + dirty;
  dirty `needs_revision` keeps author status/thread. Hard-delete pack via
  `deleteUnpublishedPack` when `canHardDelete` (never derive solely from status).

Open moderation (staff-actionable) = clean `pending`|`needs_revision`
(`hasUnsubmittedChanges=false`).

## Author Submit dirty (SC-PACK-234 / 271…273)

Pack editor / add-task-set / task-set author surfaces gate «На модерацию» with
**server** `hasUnsubmittedChanges` ∧ minima ∧ ACL/lock — not session fingerprint.
Reload must keep Submit ready when the marker is true. Own `pending` work
stays editable; first semantic save withdraws to author `draft` + dirty.
Dirty `needs_revision` keeps author-facing status/thread and Submit ready;
staff queue/approve only see clean open requests. Map editor still uses
`lib/editorDirty` session baseline (SC-MAP-65). Pack autosave may keep a local
fingerprint for put dedupe via `ensurePackSubmitBaseline` / clear on leave —
that fingerprint MUST NOT drive Submit.

## Never-live add-task-set ghost + cancel / draft (D22 / D9–D14)

Live pack payload may include `TaskSet.neverLive === true` rows for the **set
author only** (pending / needs_revision / **draft** pre-submit or after Cancel).
Staff and other users do not see ghosts on live (queue for staff — clean open
only). Activating the ghost row opens add-task-set Edit (amend), not live
tasks drill-in.

**Cancel → draft (SC-PACK-196/197 / 267):** Cancel of a never-live open cycle
sets request `status=draft` (not terminal wipe) + dirty. GET
`/api/content/packs/:id/add-task-set` MUST return that revision’s `taskSets`
plus `moderationStatus: draft`, `pendingRequestId`, `hasUnsubmittedChanges`,
and `canHardDelete`. Legacy rows may still be `cancelled` — treat as author
draft. Do not special-case “no open request → blank form” when
`draft.taskSets` is non-empty. Submit promotes retained `draft` /
dirty `needs_revision` → clean `pending` (reuse revision/cycle). Top delete
uses `discardAddTaskSetDraft` when `canHardDelete` (SC-PACK-266/274/275),
not Cancel — covers draft|pending|needs_revision never-live cycle.

## Page / UI contracts

- Catalog: untouched new pack shell (`draft_activated=false`) absent from
  list/drafts until first meaningful save (SC-PACK-269/270).
- Editor: author «На модерацию» = server dirty ∧ minima ∧ ACL/lock; top
  hard-delete when `canHardDelete` via `deleteUnpublishedPack`; own `pending`
  stays editable → first semantic save shows `draft` (SC-PACK-272); dirty
  `needs_revision` keeps label/thread and Submit ready after reload
  (SC-PACK-273).
- AddTaskSet: quiet debounce `saveAddTaskSet`; Submit = server dirty ∧ minima;
  top delete by `canHardDelete` (not status-only).

## Anti-patterns

- Enabling author Submit from session `lib/editorDirty` fingerprint instead of
  server `hasUnsubmittedChanges` (SC-PACK-234/271).
- Deriving top delete from status alone instead of server `canHardDelete`
  (SC-PACK-274/275).
- Showing untouched pack shells in catalog/drafts (SC-PACK-269).
- Treating add-task-set GET with empty `draft.taskSets` / null ids as a blank
  new set after never-live Cancel or pre-submit draft (must bind restored
  `draft.taskSets` + `moderationStatus: draft` — SC-PACK-196/265/267).
- Locking own `pending` as read-only (SC-PACK-272 — first save withdraws).
