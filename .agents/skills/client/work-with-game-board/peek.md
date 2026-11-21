# Shared peek modal (open-on-land — SC-BOARD-42…54 / SC-BOARD-84)

Read this when changing the shared Q&A modal, flipped difficulty digits, or `sendPeekPlace` / `sendPeekSubmit`. **No** board peek-open affordance — server opens on land.

## Contract

| Piece | Behavior |
|-------|----------|
| Open | Server open-on-land after trap pipeline idle on a Q&A cell. Client **must not** send `peek` to open (rejected). Shared session via synced `peekActive*` / `openPeek` (every client sees modal). Peeker alone may place/submit; others read-only (`isPeekOwner`). |
| Affordance | **Retired** — do not show `peek.png` / eye on board (SC-BOARD-84). Spent grille / exclusive trap cells have **no** underlying Q&A. |
| Modal chrome | Header: 3 read-only pack difficulty dots (**left edge**) + **word-only** badge (`content.difficultyWord.{n}` — no `Сложность:` / digit; SC-BOARD-50) **horizontally centered**. Question ~24px (SC-BOARD-54). Empty slots: dashed unlabeled portrait. Filled: mini-card + thin primary `#1976d2` border; pool: `.peek-answer-mini` neutral `--pack-card-border` — half-size `--peek-tile-max-w` ~43 + AR 156∶206 (SC-BOARD-51). Pool ≤4×4 + scroll. Peeker: full-width primary submit. Spectators: hint, no enabled submit (SC-BOARD-52/53). |
| Place / submit | Peeker: pool → next empty via `sendPeekPlace`; click filled → clear; submit → `sendPeekSubmit()`. Testid `peek-answer-chip` on pool. |
| Reuse | Same answer card MAY fill any number of slots; pool stays selectable (SC-BOARD-49). |
| Resolve | Correct → +difficulty **knowledge** on landing piece + remove tile. Wrong / leave → KEEP bind + `peekClose.reason:'wrong'`; turn advances. Multi turn timeout → `peekClose.reason:'timeout'` (incorrect). Header flash 10s + top-center Notify toast (SC-PRESENCE-36 / SC-BOARD-88). |
| Flipped digit | Public difficulty on still-present bound tasks (`flippedCells`). |

## Do / Don't

- Do: Colyseus I/O via store place/submit only; clear session from sync/`peekClose`; map `peekClose.correct` + `reason` → `peekStatusFlash` + Notify; keep `framework.plugins` includes `Notify`.
- Don't: revive `sendPeek` open / peeks budget gates / peek affordance; revive Correct/Wrong buttons; drop timeout toast/flash when multi `turnUntil` fires.
