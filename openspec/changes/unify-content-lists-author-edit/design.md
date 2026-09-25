## Context

See `proposal.md`. Базовый runtime уже содержит unified packs list, favorites, author re-edit, moderation take (packs+maps). Follow-up: in-pack task-set status marks, cancel→draft, App header + breadcrumbs, Game leave-right, drop in_catalog list badge, map View vs Edit, never-published → Edit.

Explore closed: D1–D9; Q1–Q3, G1–G4, C1–C4, G1b; V1–V5.

## Goals / Non-Goals

**Goals**

- Unified packs/maps lists + favorites/take/author re-edit (first pass — done in code)
- Per–task-set moderation badges on live pack set list (set author + staff; not pack creator for others’ sets)
- Cancel (staff|author) keeps working copy; author sees **draft** on unified list; others keep live catalog snapshot
- Shared header: packs/maps/support + staff «Модерация»; burger; breadcrumbs; Game leave right + logo leave
- List rows: **no** green «В каталоге» / in_catalog status badge; keep pending / needs_revision / draft / unpublished
- Published map: View first (author + players×tourists; no paint tools); Edit outside and inside under exclusive lock
- Never-published pack/map: open **Edit** directly
- Pack live browsing stays as today (content for discovery)

**Non-Goals**

- Game header center pack title
- tourist-room content wiring
- New npm/SaaS
- Redesigning pack live content layout beyond existing browse + Edit

## Decisions

### D1–D9 — First pass (unchanged summary)

D1 unified packs list API; D2 favorites table; D3 drop collection/default-grant; D4 author working after publish; D5 task-set author ACL; D6 edit lock for authors; D7 moderation take; D8 maps list enrichment; D9 client list surfaces. Runtime already landed.

### D10 — Per–task-set moderation status on pack live list

- **Choice:** Each task-set row on the live pack surface exposes moderation phase for eligible viewers: pending / needs_revision / draft-awaiting-submit when applicable; published/live otherwise. Visible to **that set’s author** and **staff**. Pack `createdBy` MUST NOT see another author’s set moderation marks solely by being creator. API: per-set `moderationStatus` on live/editor pack payload.
- **Why:** Q1.

### D11 — Cancel → author draft; others keep live

- **Choice:** Staff/author Cancel keeps working; author list status `draft`; others see live snapshot. Hard-delete remains delete. Soft-unpublish cascade cancel same keep-working rule.
- **Why:** C1–C4.

### D12 — Shared header navigation

- **Choice:** Packs/Maps/Support + staff «Модерация» in App header; burger on narrow; remove Lobby section toolbar and list-embedded staff links; hide sections on Game.
- **Why:** G4 + explore.

### D13 — Breadcrumbs replace redundant back

- **Choice:** Crumbs under header; remove «К наборам» / «Вернуться» where crumbs cover the path.
- **Why:** User request.

### D14 — Game leave on the right; logo also leave

- **Choice:** Right-side leave-from-room + logo leave on Game; no session logout on Game.
- **Why:** G1b/G3.

### D15 — Server list draft after cancel

- **Choice:** Author-facing `draft` after cancel with retained working; maps parity.
- **Files:** `lib/content.ts`, `lib/contentMaps.ts`, client lists.

### D16 — No in_catalog status badge on list rows

- **Choice:** Packs and Maps list rows MUST NOT show an «В каталоге» / in_catalog status badge (the green one). Rows that are simply published appear without that badge. Badges for **pending**, **needs_revision**, **draft**, and **unpublished** (staff) remain. Filters unchanged. API may still expose catalog membership for ACL; UI simply omits the in_catalog badge.
- **Why:** V2/V3.
- **Alt:** Remove all status badges — rejected.

### D17 — Map View vs Edit; never-published → Edit

- **Choice:** Opening a **published / in-catalog** map (row click or equivalent) enters **View**: show author, players×tourists (and grid preview as read-only); MUST NOT show paint-tool controls or other edit chrome at the bottom. **Edit** affordances exist on the list row and inside View; activating Edit acquires the exclusive lock and enters edit mode (tools + submit/staff-save as eligible). **Never-published** maps open **Edit** directly (no View-first). Exclusive lock: who holds it blocks others (existing TTL).
- **Layers:** `MapsListPage` routing; `MapEditorPage` hide tools when view-only; lock on Edit enter.
- **Why:** V1/V4 maps half.

### D18 — Pack never-published → Edit; live browse unchanged

- **Choice:** Never-published packs open the editor (Edit) directly from the list. In-catalog / published packs open the existing live pack surface for content discovery (author, cards, task sets as today); Edit from list and/or live enters editor under exclusive lock (existing). No strip-down of pack live browsing.
- **Why:** V4/V5.

## Risks / Trade-offs

- **[Risk] Dual view of same pack (author draft vs public in_catalog)** → Mitigation: status per caller.
- **[Risk] Logo+right leave duplicate** → Accepted.
- **[Risk] Header crowding on mobile** → Burger.
- **[Risk] Map View still shares editor route** → Use view-only mode; hide tools rather than new page unless needed.

## Migration Plan

1. Server: per-set status + cancel→draft (packs+maps).
2. Client: set badges; no in_catalog badge; map View/Edit + never-published→Edit; pack never-published→Edit; header/crumbs/leave.
3. Tests: mocha/vitest for new SC-*; lint/typecheck.
4. Rollback: revert deploy.

## Open Questions

None — V1–V5 closed.
