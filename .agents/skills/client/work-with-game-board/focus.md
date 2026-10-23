# Focus nearest actionable (SC-PRESENCE-30/31/32/33)

Read this when changing the focus control on the own avatar or nearest-actionable selection.

## Contract

| Piece | Behavior |
|-------|----------|
| Affordance | Illustrated `src/assets/board/focus.png` (`<img>`) on own avatar at **left-center (9)** when `canShowFocusControl` — own turn, playing, actionable unfinished non-trapped pieces exist; **no** Material `center_focus_strong` / solid colored circular chrome. Aria: `game.focusActionable`. |
| Layout (clock 12/9/3) | Say **12** / focus **9** / end-turn **3** — hit-centers on marker circumference (R=48): `.say-affordance` `left:50%; top:0; translate(-50%,-50%)`; `.focus-affordance` `left:0; top:50%; translate(-50%,-50%)`; `.end-turn-affordance` `left:100%; top:50%; translate(-50%,-50%)` on the 96px marker (~56–64 transparent hits; glyph ~36; icons may be non-circular — SC-PRESENCE-33). |
| Algorithm | `src/lib/focusActionable.ts` — Manhattan from current selection (or `BOARD_CENTER` 4.5,4.5 if none); ties → smaller seat piece index. |
| Click | `pickNearestActionablePieceId` → set local selection to that `pieceId` (same as strip/board select). |

## Do / Don't

- Do: keep pure helper in `lib/focusActionable`; page wires candidates from own unfinished non-trapped pieces; keep clock-axis CSS (12/9/3) when moving affordances.
- Don't: put focus geometry in Pinia; invent a server message for focus; revive offset hits (`top:-40` / `left:-8` / `right:-8`); stack say/focus/end-turn so ~60px hits overlap; revive vertical-stack layout (`top: 32%` focus between say and end-turn on the right); revive Material focus glyph or colored circular chrome behind say/focus/end-turn.
