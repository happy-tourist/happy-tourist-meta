# Knowledge HUD + grille remainingTurns

Read this when changing avatar knowledge counter, +N fall, yellow BR carried badges, red TR grille countdown, or the delivered-knowledge finish modal.

## Contract

| Piece | Behavior |
|-------|----------|
| Avatar counter | Own seated HUD only: `.presence-knowledge` / `.knowledge-counter` reads private `game.knowledgeDelivered` (SC-PRESENCE-50). Aria `game.knowledgeCounterAria`. Not on opponents. |
| +N fall | Reuse `.budget-fall` (~2s / `BUDGET_FALL_MS`) when **delivered** increases after `knowledgeSynced` (piece-finish transfer). Correct solve updates yellow BR only — no avatar fall (SC-PRESENCE-51). |
| Yellow BR | Own board piece + strip: badge from `game.knowledgePieces[pieceId]`; hide when 0 (D2 / D9 / SC-BOARD-85). |
| Red TR | **All** clients: badge from public `piece.remainingTurns` while trapped (3…1); 0 clears (D5). |
| Finish modal | Local seat `finishPlace` 0→1 → `game.knowledgeDeliveredModal` with `{ n: knowledgeDelivered }` — full finish **and** penultimate cut; **no** place number / place badge (SC-FINISH-31/34). |
| Retired | Steps/peeks counters, ∞ leftover HUD, place modal, timer-only solo end, end-turn affordance, push affordance, peek-open affordance. |

## Presence clock

| Slot | Affordance |
|------|------------|
| **12** | Say (unchanged) |
| **3** | Focus (former end-turn slot — SC-PRESENCE-53/55); **no** end-turn control |
| Under peeks / budgets | **Removed** — do not put focus back under counters |

## Do / Don't

- Do: gate +N fall on `knowledgeSynced`; map `remainingTurns` from schema pieces; open one knowledge modal on finishPlace edge.
- Don't: show ranked place / leftover timer modals; revive `end-turn.png` / `push.png` / `peek.png` board affordances; put delivered/carried on public schema.
