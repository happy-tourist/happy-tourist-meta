## Context

See `proposal.md` — Why. Базовый runtime: `lib/content.ts` (packs, collections, staff edit lock TTL 5 мин, moderation requests), `lib/contentMaps.ts`, `lib/defaultContentPacks.ts`; client `stores/content` + `stores/maps`, collection/catalog/my-moderation/staff pages, `MapsListPage`.

Explore decisions (закрыты): D1–D9, D2b out of scope (game wiring), D3c author via moderation only, D5 take-to-moderate, D5c no moderate without take, D7/D7b verified add-task-set without collection, D8 favorites = starred, D9 task-set authors edit own sets.

## Goals / Non-Goals

**Goals**

- Unified packs list + map list parity (statuses, filters; favorites only packs)
- Remove collection product surface and default-grant
- Author / task-set-author re-edit via queue; exclusive edit lock shared with staff
- Staff moderation take with same TTL as edit lock

**Non-Goals (design-level)**

- Wiring packs/maps into `tourist-room` / createGame
- New npm/SaaS dependencies
- Redesign support `change_pack` beyond dropping collection dependency (catalog select stays)

## Decisions

### D1 — Packs list API replaces collection-first

- **Choice:** Expand `GET /api/content/packs` (or dedicated list) to return in-catalog + caller’s never-published + staff soft-unpublished, with `moderationStatus`, `isMine`, `isFavorite`, contribution flags for filters. Deprecate collection list as primary; remove collect/uncollect UI.
- **Why:** Mirrors `listMaps`.
- **Alt:** Keep collection table as favorites — rejected (D1: favorites separate, collection gone).

### D2 — Favorites table

- **Choice:** New `content_pack_favorites` (userId, packId, createdAt); star/unstar endpoints; list field `isFavorite`. Registered non-anonymous only; in-catalog packs.
- **Why:** Personal filter without membership semantics.
- **Alt:** Reuse `content_pack_collections` — rejected (product meaning differs; cleaner drop).

### D3 — Drop collections + default grants

- **Choice:** Stop using `content_pack_collections` in product paths; remove `defaultContentPacks` grant on create/startup (or no-op). Drop/ignore `DEFAULT_CONTENT_PACK_IDS` grant behavior. Client: lobby → packs list; delete or redirect collection route; remove my-moderation nav for non-staff.
- **Migration:** Leave table unused or migrate once empty; no user-facing collection.
- **Why:** Explore D1/D2/(2).

### D4 — Author working copy after publish

- **Choice:** When published pack/map is edited by eligible author, mutate `workingRevisionId` (create working from live if needed), submit opens/resumes moderation request; approve promotes to live. Staff direct `staff-save` blocked while open author request on that entity.
- **Why:** D3c — author only via moderation.
- **Alt:** Author live save — rejected.

### D5 — Task-set author ACL

- **Choice:** Pack-level `edit_locked_*` shared. Permission: `createdBy` → cards + any sets; task-set author → own set only; staff → all when allowed. Submit type `task_set` for set-only cycles.
- **Why:** D9 / explore C + pack owner may edit all sets.

### D6 — Edit lock for authors

- **Choice:** Reuse `edit_locked_by` / `edit_locked_at` + `EDIT_LOCK_TTL_MS` (5 min); acquire on author Edit enter; same errors `edit_locked` / `edit_lock_required`.
- **Why:** Existing pattern; explore «как редактирование».

### D7 — Moderation take

- **Choice:** Columns on `content_moderation_requests`: `taken_by`, `taken_at` (TTL = edit lock). Endpoints: take / release (or unlock-on-leave). Approve/reject/cancel require `taken_by === caller` and non-expired. UI: button on staff queue row + detail. Author resubmit allowed while taken.
- **Why:** D5/D5c; page-only lock fragile on tab close — explicit take + TTL.
- **Alt:** Page-entry lock without button — rejected by product.

### D8 — Maps list enrichment

- **Choice:** `listMaps` / `mapSummary` include `moderationStatus`; client badges + filters (all / moderation / drafts / mine). Remove non-staff my-moderation nav from Maps chrome. Author re-edit + take same as packs.
- **Why:** Parity; no favorites.

### D9 — Client surfaces

- **Choice:** Primary packs page = unified list with filter chips + star; pack detail star; staff queue take control; Map list filters; routes: collection → redirect to packs list; hide `content-my-moderation` for non-staff (staff may keep deep-link unused).
- **Layers:** pages → Pinia `content` / `maps` → `client.http`; errors via store + banner.

## Risks / Trade-offs

- **[Risk] Stale collection rows / default-grant env** → Mitigation: ignore membership in ACL; no-op grants; document env ignore; optional table drop later.
- **[Risk] Two locks (edit vs take) confuse staff** → Mitigation: clear i18n («занято редактированием» / «взято на модерацию»); block staff Edit when author request open.
- **[Risk] Concurrent author resubmit while staff reviews** → Mitigation: accepted (explore); preview refreshes on load; take does not freeze author.
- **[Risk] Large list without server-side filter** → Mitigation: start with client filter on ≤200 rows (current list limits); add query params if needed.
- **[Risk] Support/default-pack skills/docs drift** → Mitigation: update meta skills/AGENTS hints in apply follow-up via check-changes (not blocking design).

## Migration Plan

1. Server: schema favorites + moderation take; list endpoints; ACL without collection; disable default grants; author re-edit + lock; take gates.
2. Client: unified list UI, filters, star; remove collection/my-moderation nav; staff take; map filters/statuses; author Edit affordances.
3. Tests: mocha SC-PACK-148… / SC-MAP-31…; vitest list/filter/ACL/take.
4. Rollback: feature flags not required; revert deploy; favorites table harmless if unused.

## Open Questions

None material — defaults: pack `createdBy` may edit all sets; favorites only in-catalog; take release on leave + TTL like edit unlock.
