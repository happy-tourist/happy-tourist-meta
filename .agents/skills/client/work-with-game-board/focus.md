# Focus nearest actionable (SC-PRESENCE-30/31/32/33/37)

Read this when changing the focus control or nearest-actionable selection.

## Contract

| Piece | Behavior |
|-------|----------|
| Affordance | Illustrated `src/assets/board/focus.png` (`<img>`) **under peeks** in own `.budget-counters` when `canShowFocusControl` — own turn, playing, actionable unfinished non-trapped pieces exist; **no** Material `center_focus_strong` / solid colored circular chrome; **not** on marker @9. Aria: `game.focusActionable`. |
| Layout | Focus stacks **under** the peeks counter row inside `.presence-budgets` / `.budget-counters` (relative hit ~56–64, glyph ~36, `margin` auto). Marker circumference: say **12** + end-turn **3** only — left @9 empty. See `work-with-styles/board.md`. |
| Algorithm | `src/lib/focusActionable.ts` — Manhattan from current selection (or `BOARD_CENTER` 4.5,4.5 if none); ties → smaller seat piece index. |
| Click | `pickNearestActionablePieceId` → set local selection to that `pieceId` (same as strip/board select). |

## Do / Don't

- Do: keep pure helper in `lib/focusActionable`; page wires candidates from own unfinished non-trapped pieces; keep focus in budgets under peeks (SC-PRESENCE-37).
- Don't: put focus geometry in Pinia; invent a server message for focus; revive focus on marker @9 (`left:0; top:50%`); revive offset hits (`top:-40` / `left:-8` / `right:-8`); stack say/focus/end-turn on the marker so ~60px hits overlap; revive vertical-stack layout (`top: 32%` focus between say and end-turn on the right); revive Material focus glyph or colored circular chrome; pad HUD `padding-left` to keep focus@9.
