# Content HTTP routes (`/api/content/*`)

Read with the [core routes skill](SKILL.md). Handlers stay thin in
`src/app.config.ts`; business logic in `src/lib/content.ts` /
`contentMaps.ts` (`server-work-with-structure`).

## Packs + moderation

| Method | Path | Source | Notes |
|--------|------|--------|-------|
| GET | `/api/content/packs` | `createEndpoint` | JWT; unified list — in-catalog + caller drafts/pending + staff soft-unpublished; rows carry `moderationStatus` / `openRequestType` (`pack`\|`task_set` SC-PACK-191/192) / `isMine` / `isFavorite` (SC-PACK-148…153); **caller working≠live → `draft`** even when in-catalog (SC-PACK-175…179); never-live add-task-set draft → `draft` + `openRequestType: task_set` when working==live (SC-PACK-268); lightweight **`taskSetsPreview[]`** per pack (`id`, 1-based `ordinal`, `taskCount`, `inCatalog`, `neverLive?`) — full array, count-only / no slots-cards (SC-PACK-252/253) |
| POST | `/api/content/packs` | `createEndpoint` | JWT + non-anonymous + `emailVerified` (DB); create pack + working copy |
| GET | `/api/content/packs/:id` | `createEndpoint` | JWT; live snapshot + `isFavorite` + `inCatalog` + per-set `moderationStatus` for set author/staff (SC-PACK-171…174, 187 — no authorId fan-out; **staff omit working≠live `draft` without open author request — SC-PACK-230**) + set-author-only `neverLive` ghost rows (SC-PACK-188…189; includes author `draft` pre-submit); legacy `inCollection`; non-staff soft-unpublished → `pack_unpublished` |
| POST | `/api/content/packs/:id/favorite` \| `/unfavorite` | `createEndpoint` | JWT registered non-anonymous; star/unstar in-catalog pack (SC-PACK-154) |
| POST | `/api/content/packs/:id/unpublish` \| `/republish` | `createEndpoint` | JWT staff; soft-hide / restore pack catalog; unpublish cascade-cancels open requests + RU email SC-PACK-137…141 |
| POST | `/api/content/task-set/unpublish` \| `/republish` | `createEndpoint` | JWT staff; soft-hide / restore **task set**; last published → 409 `last_published_task_set` |
| GET\|POST | `/api/content/packs/:id/draft` | `createEndpoint` | JWT + creator **or** task-set author; working copy incl. post-publish re-edit (`editorKind`); set `moderationStatus` on payload; staff → `author_request_open` when author request open |
| POST | `/api/content/packs/:id/submit` | `createEndpoint` | JWT + author; unified submit / resubmit (may stay pending while staff holds take) |
| GET\|PUT\|POST | `/api/content/packs/:id/add-task-set` (+ `/submit` + `/discard`) | `createEndpoint` | JWT + verified; **no** collection membership required (SC-PACK-164); put creates/updates author `status=draft` without submit-minima (D9 / SC-PACK-264); GET restores draft\|legacy cancelled (SC-PACK-265/196); submit draft→pending; Cancel never-live → `draft` (SC-PACK-267); `POST …/discard` deletes draft request + orphan revision (SC-PACK-266); staff queue stays pending\|needs_revision only |
| GET\|POST | `/api/content/packs/:id/edit-lock` | `createEndpoint` | JWT author **or** staff; acquire/status (TTL 5 min); staff blocked if author request open |
| POST | `/api/content/packs/:id/edit-unlock` | `createEndpoint` | JWT lock holder; release |
| GET\|POST | `/api/content/packs/:id/staff-edit` \| `/staff-save` | `createEndpoint` | JWT staff + lock; load / direct save; after live staff-save with no open author request **always** clear `workingRevisionId` + orphan revision without moderation refs (SC-PACK-233; extends SC-PACK-230 twin clear) |
| GET\|POST | `/api/content/packs/:id/moderation` (+ `/messages`) | `createEndpoint` | JWT; author ↔ staff thread |
| POST | `/api/content/packs/:id/block` \| `/unblock` | `createEndpoint` | JWT moderator\|admin (retained; client UI hidden) |
| POST | `/api/content/pack/delete` | `createEndpoint` | JWT + creator; hard-delete **unpublished** pack |
| POST | `/api/content/task-set/delete` | `createEndpoint` | JWT + creator; delete task set from unpublished working copy |
| GET | `/api/content/collection` | `createEndpoint` | JWT; **legacy** membership list (product UI removed; no auto-grant) |
| POST | `/api/content/collection` \| `/remove` | `createEndpoint` | JWT; legacy add/remove (do not rebuild product collection UX) |
| GET | `/api/content/my-moderation` | `createEndpoint` | JWT; caller’s open requests (pack **and** map) |
| GET | `/api/content/staff/pending` | `createEndpoint` | JWT staff; open queue (pack **and** map) + `takenBy`/`takenAt` |
| POST | `/api/content/staff/requests/:id/take` \| `/release` | `createEndpoint` | JWT staff; take-to-moderate / release (TTL = edit lock; SC-PACK-161…163 / SC-MAP-38/39) |
| GET\|POST | `/api/content/staff/requests/:id` (+ `/approve` `/needs-revision` `/reject` `/cancel` `/messages`) | `createEndpoint` | JWT staff; terminal actions require held take (`moderation_take_required`); GET = `previewPending` |

**Content moderation mail:** deep-links prefer `#/content/packs/:id/edit` (working copy / staff edit) — **not** bare `#/content/packs/:id/moderation`. Map mails deep-link `#/content/maps/:id/edit`.

## Maps

| Method | Path | Source | Notes |
|--------|------|--------|-------|
| GET\|POST | `/api/content/maps` | `createEndpoint` | JWT; list with `moderationStatus` / `authorRequestOpen` / create; author working≠live → `draft` (SC-MAP-41/42) |
| GET\|POST | `/api/content/maps/:id` (+ `/draft` `/submit` `/moderation` `/edit-lock` `/staff-edit` `/staff-save` `/unpublish` `/republish`) | `createEndpoint` | JWT; author re-edit + staff lock gated by open author request; live `staff-save` clears retained working when no open request (SC-MAP-64); soft-unpublish cascade SC-MAP |
| POST | `/api/content/map/delete` | `createEndpoint` | JWT + creator; hard-delete **never-approved** map |

## Add-task-set draft contract (D9–D11)

- **Put** (`PUT/POST …/add-task-set`): incomplete OK (`validateAddTaskSetDraft`); creates/updates `type=task_set` request `status=draft`.
- **Get**: restores draft \| legacy cancelled payload.
- **Submit**: draft→pending; still enforces submit minima.
- **Discard**: `POST …/add-task-set/discard` — delete never-live draft request + orphan revision; live untouched.
- **Cancel** open never-live: → `status=draft` (staff queue excludes draft).
