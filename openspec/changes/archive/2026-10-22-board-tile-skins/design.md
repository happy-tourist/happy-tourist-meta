## Context

See `proposal.md` — Why. Client board and map previews use flat CSS fills (`#4caf50` / `#8d6e63` / `#ffeb3b` / `#fdd835`). Finish cells are merged in client `buildBoardTiles` into one CSS-grid `span` block; click/outline use nearest-legal-center (SC-MOVE-65). Server already treats each grid char `7` as its own playable center cell (`boardGeometry` → `centerCells` / `playable`) and validates `move` to that exact cell — no protocol change. Flipped digit already comes from synced `flippedCells`. Shipped skins live at client `src/assets/tiles/` (`start.png`, `tile.png`, `tile-back.png`, `finish.png`) — square, flush-cropped.

**Unchanged server contract (this change):** room `tourist`; message `move` payload `{ pieceId, row, col }`; per-cell playable centers for every `7`; synced `flippedCells` (difficulty digit source). No new messages, schema fields, HTTP, auth, or DB. Illegal finish clicks are a **client** no-submit (no nearest-remap) — server reject paths stay as today.

## Goals / Non-Goals

**Goals:**

- One skin mapping for kinds across Game board, shared map grid preview, and map paint palette.
- Per-cell finish tiles (no visual merge; `BoardTile.kind` stays `'center'`).
- Ordinary per-cell click/outline for finish (no block-wide select/target).
- Readable difficulty digit on flipped face (`#212121`).
- Skin silhouette unclipped (`border-radius: 0` on playable board tiles and palette faces).

**Non-Goals:**

- Server/protocol changes; flip animation; new deps; regenerating tourist/grille/catapult art.
- Return-from-finish **animation origin** (nearest of the four centers, SC-FINISH-15) — leave as-is; this change only rewires **move** finish click/outline (SC-MOVE-65).

## Decisions

### D1 — Skin assets (already prepared)

| Kind / state | Client CSS hook | File |
|--------------|-----------------|------|
| start | `.tile-start` | `src/assets/tiles/start.png` |
| task unbound | `.tile-task` (no `.tile--flipped`) | `src/assets/tiles/tile.png` |
| task flipped | `.tile-task.tile--flipped` (class already on GamePage) | `src/assets/tiles/tile-back.png` |
| finish | `.tile-center` | `src/assets/tiles/finish.png` |

Import like grille/catapult PNGs; apply as `background-image` (`background-size: 100% 100%`, no repeat). Playable board tiles MUST use `border-radius: 0` (drop prior `--radius: 2` / `var(--radius)` on `.tile`) so notched art is not clipped. Removed tasks stay transparent holes (no skin). **As-built:** shared module `src/lib/tileSkins.ts` (`TILE_SKINS` + `tileSkinCssVars()` / `TILE_SKIN_CSS_VARS`) reused by GamePage, `MapGridPreview`, and map palette — do not copy divergent URL strings.

**Alternatives:** inline SVG / CSS-only — rejected; art is shipped PNG.

### D2 — Shared preview + palette reuse board skins

`MapGridPreview` and map editor palette swatches use the same kind images (palette/task = unbound `tile.png`; no flip in preview). Surfaces that already embed `MapGridPreview` (Maps list, lobby create/room cards, staff map request) inherit automatically. Palette swatches (~48px): same skins, `border-radius: 0` on the face (do not keep the old 6px round that clips notches). Tiny list previews (~7px cells): skins still required; 1px cell radius is negligible / acceptable. Preview frame may keep its outer radius + `overflow: hidden` (corner notches on ~7px cells may clip slightly — acceptable).

**Alternatives:** keep flat colors on tiny previews — rejected (product: «везде»).

### D3 — Drop center merge in `buildBoardTiles`

Emit **one** `BoardTile` per `7` cell with `kind: 'center'` (keep existing kind string — do **not** rename to `'finish'`). Each tile’s `style` is a single grid cell (`gridColumn` / `gridRow` without `span`), same as start/task.

GamePage finish chrome MUST match ordinary cells (today’s block-wide predicates to remove):

- `isTileSelected`: drop `tile.kind === 'center' → isCenterCell(sel…)` — use `tile.row/col === sel` (via `cellKey` equality).
- `isTileTarget`: drop `CENTER_CELLS.some(… legal …)` for the whole finish block — use `legalTargetKeys.has(cellKey(tile.row, tile.col))` per finish tile (keep “selection chrome wins over target” when the selected piece sits on that same cell).
- `onTileClick`: drop `nearestLegalCenterCell` remap — submit `tile.row/col` only when that key is legal; otherwise no-submit.
- Delete `nearestLegalCenterCell` once unused.
- White selection outline only on the selected piece’s cell; red target outline only on finish cells that are legal destinations (SC-MOVE-65).

OpenSpec delta for SC-MOVE-65: **REMOVED** obsolete `Finish-block click resolves to nearest legal center cell` + **ADDED** `Finish-cell click submits that cell only` (same scenario id; archive refuses MODIFIED that drops the old scenario title).

**Alternatives:** keep one spanning tile with four internal hit zones — rejected (user: paint per cell).

### D4 — Flipped digit ink

On `tile-back`, digit color **`#212121`** (no text-shadow) so it reads on the beige flip face; drop white + light text-shadow used on the brown unbound face.

### D5 — Skills / docs touchpoints (apply)

Client board/maps skill notes already describe D1–D4 / SC-MOVE-65 (task 2.4 done). At apply: only verify runtime matches those notes (skins, no merge center, gap 2, per-cell finish). Checklist → `tasks.md`.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Tiny preview (~7px cells) loses texture | Color silhouette still matches kinds; acceptable |
| Preview frame `overflow: hidden` clips corner notches | Acceptable at list size; do not invent per-cell masks |
| Outline on notched art looks square | Keep existing outline-offset; do not invent mask outlines |
| Tests assert hex fills / one `span` center / `center` length 1 | Update Vitest: N center tiles, no `span`; swatches/skins not flat hex-only |

## Migration Plan

Client-only deploy with static assets. No server migration. Rollback = revert client commit (flat fills + merge return).

## Open Questions

(none — explore decisions locked)
