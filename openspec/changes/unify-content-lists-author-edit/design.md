## Context

See `proposal.md`. Runtime уже содержит unified lists, favorites, author re-edit, take, per-set marks, cancel→draft, header/crumbs/leave, no in_catalog badge, map View/Edit, never-published→Edit. Follow-up: crumbs below elevated header; map Edit top parity; fix open-status fan-out; never-live add-task-set rows for set author; author draft/pending/needs_revision → Edit (packs+maps); pack list with open add-task-set → live first.

Explore closed: D1–D18; Q1–Q3 crumbs/Edit/badge; ghost set Q1–Q3.

## Goals / Non-Goals

**Goals**

- Prior goals (unified lists, take, set marks, cancel→draft, header, leave, no in_catalog badge, map View, never-published→Edit) — landed
- Breadcrumbs in page zone **below** elevated `q-header` (links readable, not blended into blue header)
- Map View Edit control in **title row** (pack live parity)
- Open moderation status applies only to set(s) in that request (no authorId fan-out across sibling live sets)
- Never-live add-task-set visible as a task-set row to **set author only**, with pending/needs_revision/draft; row → Edit
- Author-facing draft | pending | needs_revision → **Edit** from lists (packs + maps); clean published → View / live browse
- Pack list when open work is **add-task-set only** → live pack (answers + sets), then ghost row → Edit

**Non-Goals**

- Game header center pack title
- tourist-room content wiring
- New npm/SaaS
- Staff seeing never-live ghost sets on live pack (staff use moderation queue)
- Redesigning pack live cards layout beyond ghost set rows + Edit entry

## Decisions

### D1–D9 — First pass (unchanged summary)

D1–D9 as before. Runtime already landed.

### D10 — Per–task-set moderation status (revised)

- **Choice:** Marks for eligible viewers on the **set(s) that belong to the open request** (match via request `revisionId` payload / semantic identity among that author’s sets — **not** solely `changeAuthorId`). Values: pending / needs_revision / draft / live as applicable. **Set author** and **staff** see marks on **already-live** sets under open re-edit. Pack `createdBy` MUST NOT see foreign marks solely as creator. A never-live add-task-set ghost row (D19) is **set-author only** (staff use queue).
- **Why:** Q1 explore + badge bug (fan-out).
- **Files:** `lib/content.ts` `withTaskSetModerationStatuses`; mocha multi-set case.

### D11 — Cancel → author draft; others keep live

Unchanged.

### D12 — Shared header navigation

Unchanged.

### D13 — Breadcrumbs below elevated header (revised)

- **Choice:** Crumbs render **below** the shared elevated header chrome (after `</q-header>` / page-layout strip), not as a second row inside the sticky blue header. Remove redundant «К наборам» / «Вернуться» where crumbs cover the path.
- **Why:** Links blended with blue header when inside `q-header`.
- **Layers:** `App.vue` layout; packs/maps chrome.

### D14 — Game leave on the right; logo also leave

Unchanged.

### D15 — Server list draft after cancel

Unchanged.

### D16 — No in_catalog status badge on list rows

Unchanged.

### D17 — Map View vs Edit; author work → Edit (revised)

- **Choice:** Clean **published / in-catalog** map (no author-facing draft/pending/needs_revision for the caller) opens **View** first; View shows author + players×tourists; no paint tools. **Edit** on list and in View (title-row primary, pack parity) acquires lock. **Never-published** OR author-facing **draft / pending / needs_revision** opens **Edit** directly (not View-first).
- **Why:** V1/V4 + author “any draft → Edit” including on moderation.
- **Layers:** `MapsListPage` open routing; `MapEditorPage` title-row Edit when view-only.

### D18 — Pack open routing; live browse (revised)

- **Choice:** Never-published OR pack-level author draft/pending/needs_revision → **Edit** from packs list. Clean in-catalog → live pack browse. When the caller’s only open work on P is **add-task-set**, choosing P from the packs list opens **live** (answers + task-set list), not the add-task-set editor directly; the ghost set row (D19) then opens Edit.
- **Why:** Q3 — through answers list for add-task-set.

### D19 — Never-live add-task-set row for set author

- **Choice:** While set author U has a never-live task set in an open or draft add-task-set cycle (pending / needs_revision / draft after cancel), live pack task-set list for U MUST include that set as a row with the same status vocabulary as the unified packs list. Other users (including pack creator and staff on live) MUST NOT see that never-live row — staff review via moderation queue. Activating the row opens **Edit** (add-task-set amend surface).
- **Why:** Author could not enter the set after needs_revision; parity with “only authors see drafts” on the unified list.
- **Files:** `getLivePack` / payload merge; `ContentPackPage` row + route; mocha/vitest.

## Risks / Trade-offs

- **[Risk] Dual view of same pack (author draft vs public in_catalog)** → Mitigation: status per caller.
- **[Risk] Logo+right leave duplicate** → Accepted.
- **[Risk] Header crowding on mobile** → Burger.
- **[Risk] Map View still shares editor route** → view-only mode; hide tools.
- **[Risk] Set identity across revisions** → Prefer revision payload match + fingerprint/index among author sets; document in tests.
- **[Risk] Staff miss ghost on live** → Accepted; queue remains source of truth for staff.

## Migration Plan

1. Server: fix fan-out; merge never-live author sets into live pack payload for set author.
2. Client: crumbs below header; map Edit top; ghost row + Edit; list open routing (draft→Edit; add-task-set→live first).
3. Tests: mocha/vitest for new SC-*; lint/typecheck.
4. Rollback: revert deploy.

## Open Questions

None — crumbs / Edit top / ghost / Q1–Q3 closed.
