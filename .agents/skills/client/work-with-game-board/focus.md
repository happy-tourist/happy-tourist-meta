# Focus nearest actionable (SC-PRESENCE-30/31/32/33)

Read this when changing the focus control on the own avatar or nearest-actionable selection.

## Contract

| Piece | Behavior |
|-------|----------|
| Affordance | Illustrated `src/assets/board/focus.png` (`<img>`) on own avatar at **left-center (9)** when `canShowFocusControl` — own turn, playing, actionable unfinished non-trapped pieces exist; **no** Material `center_focus_strong` / solid colored circular chrome. Aria: `game.focusActionable`. |
| Layout (clock 12/9/3) | Say **12** top-center, focus **9** left-center, end-turn **3** right-center — no hit-area overlap (SC-PRESENCE-33): `.say-affordance` `top: -40px; left: 50%; translateX(-50%)`; `.focus-affordance` `top: 50%; left: -8px; translateY(-50%)`; `.end-turn-affordance` `top: 50%; right: -8px; translateY(-50%)` on the 96px marker (~36px transparent hits stay clear; icons may be non-circular). |
| Algorithm | `src/lib/focusActionable.ts` — Manhattan from current selection (or `BOARD_CENTER` 4.5,4.5 if none); ties → smaller seat piece index. |
| Click | `pickNearestActionablePieceId` → set local selection to that `pieceId` (same as strip/board select). |

## Do / Don't

- Do: keep pure helper in `lib/focusActionable`; page wires candidates from own unfinished non-trapped pieces; keep clock-axis CSS (12/9/3) when moving affordances.
- Don't: put focus geometry in Pinia; invent a server message for focus; stack say/focus/end-turn so 36px hit areas overlap; revive vertical-stack layout (`top: 32%` focus between say and end-turn on the right); revive Material focus glyph or colored circular chrome behind say/focus/end-turn.
