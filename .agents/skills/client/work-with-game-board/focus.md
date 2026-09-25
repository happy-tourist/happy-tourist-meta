# Focus nearest actionable (SC-PRESENCE-30/31/32)

Read this when changing the focus control on the own avatar or nearest-actionable selection.

## Contract

| Piece | Behavior |
|-------|----------|
| Affordance | Icon `center_focus_strong` on own avatar (between say and end-turn) when `canShowFocusControl` — own turn, playing, actionable unfinished non-trapped pieces exist. Aria: `game.focusActionable`. |
| Algorithm | `src/lib/focusActionable.ts` — Manhattan from current selection (or `BOARD_CENTER` 4.5,4.5 if none); ties → smaller seat piece index. |
| Click | `pickNearestActionablePieceId` → set local selection to that `pieceId` (same as strip/board select). |

## Do / Don't

- Do: keep pure helper in `lib/focusActionable`; page wires candidates from own unfinished non-trapped pieces.
- Don't: put focus geometry in Pinia; invent a server message for focus.
