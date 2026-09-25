# Shared peek modal (SC-BOARD-42…48)

Read this when changing peek eye affordance, shared Q&A modal, flipped difficulty digits, or `sendPeek` / `sendPeekPlace` / `sendPeekSubmit`.

## Contract

| Piece | Behavior |
|-------|----------|
| Affordance | Eye **top-center** only on **selected** own **non-trapped** piece on present `*` (peeks remain / solo peeks∞ / free re-peek of flipped; no one-peek/turn gate). Spent grille cell still peekable. Keep-focus → eye without re-click (SC-BOARD-14). **No** ambient peekable tile chrome. |
| Open | `game.sendPeek(pieceId)` → server binds deck task on first open; shared session via synced `peekActive*` / `openPeek` (every client sees modal). Peeker alone may place/submit; others read-only (`isPeekOwner`). |
| Modal | Difficulty + question + ordered slots + pack `answerCards` chips. Peeker: click chip → `sendPeekPlace(slotIndex, answerCardId\|null)`; submit → `sendPeekSubmit()`. Spectators: `game.peekSpectatorHint` (no Correct/Wrong stub). |
| Resolve | Correct → +difficulty steps + remove tile (`removedTaskKeys`). Wrong / leave / timeout → KEEP bind; flipped digit stays (`flippedCells`). Fresh peek spends peeks; flipped free at peeks=0. |
| Flipped digit | Public difficulty on still-present bound tasks (`game.flippedCells` / tile chrome SC-BOARD-42). |

## Do / Don't

- Do: Colyseus I/O only via store (`sendPeek` / `sendPeekPlace` / `sendPeekSubmit`); clear session from sync/`peekClose`, not local clear on submit.
- Don't: revive `peekAnswer` / Correct/Wrong buttons; gate client on peeks when flipped re-peek is free; ambient-highlight other peekable cells.
