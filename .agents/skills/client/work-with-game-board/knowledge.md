# Knowledge HUD + grille remainingTurns

Read this when changing avatar knowledge counter, +N fall, strip yellow TR carried badges, red TR grille countdown, Notify peek toasts, turn ring, or the delivered-knowledge finish|cut modal.

## Contract

| Piece | Behavior |
|-------|----------|
| Avatar counter | Own seated HUD only: `.presence-knowledge` / `.knowledge-counter` + Material `lightbulb` (not `mdi-*`) reads private `game.knowledgeDelivered` (SC-PRESENCE-50/D9). Aria `game.knowledgeCounterAria`. Not on opponents. |
| +N fall | Reuse `.budget-fall` (~2s / `BUDGET_FALL_MS`) when **delivered** increases after `knowledgeSynced` (piece-finish transfer). Correct solve updates strip yellow only — no avatar fall (SC-PRESENCE-51). |
| Yellow TR (strip only) | Own strip slots: badge top-right from `game.knowledgePieces[pieceId]` + Material `lightbulb`; hide when 0. **Board** must **not** show carried-knowledge badge (SC-PRESENCE-52 / D9). |
| Red TR | **All** clients: badge from public `piece.remainingTurns` while trapped (3…1); board badge is a tourist-board sibling **above** grille overlay (`z-index`); strip TR too; 0 clears (D5 / SC-BOARD-85). |
| Finish modal | Local seat `finishPlace` 0→1 → i18n by `game.knowledgeOutcome`: `knowledgeDeliveredModal` (finish) vs `knowledgeDeliveredModalCut` (cut; «которые успели», not «что успели») with `{ n: knowledgeDelivered }` — **no** place number (SC-FINISH-31/32/34). |
| Peek toast | `peekClose` → Quasar Notify top-center «Правильно!» / «Ошибка!» / «Не успел!» (SC-PRESENCE-36 / SC-BOARD-88); header flash alone is not enough. |
| All-jail skip toast | `allJailSkip` → Notify top-center `game.allJailSkipToast`; never open start-reset modal (SC-PRESENCE-57). |
| Turn ring | Current seat when `turnUntil`/`turnBudgetSeconds` active (60s multi) — dual with reconnect grace when offline (SC-PRESENCE-54/10). |
| Retired | Steps/peeks counters, ∞ leftover HUD, place modal, timer-only solo end, end-turn affordance, push affordance, peek-open affordance, all-jail teleport modal, board yellow knowledge badge, `mdi-lightbulb-on` (use Material `lightbulb`). |

## Presence clock

| Slot | Affordance |
|------|------------|
| **12** | Say (unchanged) |
| **3** | Focus (former end-turn slot — SC-PRESENCE-53/55); **no** end-turn control |
| Under peeks / budgets | **Removed** — do not put focus back under counters |

## Do / Don't

- Do: gate +N fall on `knowledgeSynced`; map `remainingTurns` from schema pieces; open one knowledge modal on finishPlace edge; bind modal copy to live `knowledgeOutcome`.
- Don't: show ranked place / leftover timer modals; revive `end-turn.png` / `push.png` / `peek.png` board affordances; put delivered/carried on public schema; put yellow knowledge on board pieces.
