# Shared peek modal (SC-BOARD-42…54)

Read this when changing peek eye affordance, shared Q&A modal, flipped difficulty digits, or `sendPeek` / `sendPeekPlace` / `sendPeekSubmit`.

## Contract

| Piece | Behavior |
|-------|----------|
| Affordance | Eye **top-center** only on **selected** own **non-trapped** piece on present `*` (peeks remain / solo peeks∞ / free re-peek of flipped; no one-peek/turn gate). Spent grille cell still peekable. Keep-focus → eye without re-click (SC-BOARD-14). **No** ambient peekable tile chrome. |
| Open | `game.sendPeek(pieceId)` → server binds deck task on first open; shared session via synced `peekActive*` / `openPeek` (every client sees modal). Peeker alone may place/submit; others read-only (`isPeekOwner`). |
| Modal chrome | Header: 3 read-only pack difficulty dots + **word-only** badge (`content.difficultyWord.{n}` — no `Сложность:` / digit; SC-BOARD-50); dots+badge **centered**. Question: game peek ~24px (SC-BOARD-54; compose `q-input` untouched). Empty slots: dashed unlabeled portrait (`.peek-slot--portrait`). Filled: mini-card + thin primary `#1976d2` border; pool: `.peek-answer-mini` neutral `--pack-card-border` — both half-size `--peek-tile-max-w` ~43 + AR 156∶206 (SC-BOARD-51). Pool: `.peek-answer-pool` ≤4×4 + scroll (`max-height` from tile band). Peeker: full-width primary submit (`peek-submit`, pack action-h 28–36). Spectators: hint, no enabled submit (SC-BOARD-52/53). |
| Place / submit | Peeker: click pool mini-card → fill next empty slot via `sendPeekPlace(slotIndex, answerCardId\|null)`; click filled slot → clear; submit → `sendPeekSubmit()`. Testid `peek-answer-chip` remains on pool cards. Spectators: `game.peekSpectatorHint` (no Correct/Wrong stub). |
| Reuse | Same answer card MAY fill any number of slots (incl. all). Pool mini-cards stay selectable; **no** disable / used / consumed chrome after place (SC-BOARD-49). Full slots → pool click no-op. Task compose dialog mirrors this (`PackTaskComposeDialog`, SC-PACK-263/276/277). |
| Resolve | Correct → +difficulty steps + remove tile (`removedTaskKeys`). Wrong / leave / timeout → KEEP bind; flipped digit stays (`flippedCells`). Fresh peek spends peeks; flipped free at peeks=0. Correct order still exact match (repeats allowed when task defines them). |
| Flipped digit | Public difficulty on still-present bound tasks (`game.flippedCells` / tile chrome SC-BOARD-42). |

## Do / Don't

- Do: Colyseus I/O only via store (`sendPeek` / `sendPeekPlace` / `sendPeekSubmit`); clear session from sync/`peekClose`, not local clear on submit; keep pool mini-cards reusable without used-state chrome; wrap modal in `.peek-chrome` for `--pack-card-*` / `--pack-ts-dot-*`.
- Don't: revive `peekAnswer` / Correct/Wrong buttons; gate client on peeks when flipped re-peek is free; ambient-highlight other peekable cells; mark a pool card used solely because it already appears in a slot; force peek portrait AR onto `PackTaskTile` list slots (SC-PACK-238 carve-out); change server peek messages/schema in a chrome-only change.
