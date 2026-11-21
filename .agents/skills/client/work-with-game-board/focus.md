# Focus nearest actionable tourist

Read this when changing the focus control or nearest-actionable selection.

## Contract (product)

| Topic | Rule |
|-------|------|
| Affordance | Illustrated `src/assets/board/focus.png` (`<img>`) on own presence marker at **3 o’clock** (former end-turn slot: `left:100%; top:50%; transform: translate(-50%,-50%)`) when `canShowFocusControl` — own turn, playing, actionable unfinished non-trapped pieces exist; **no** Material focus glyph / colored circle; **not** under knowledge counter. Aria: `game.focusActionable` (SC-PRESENCE-53/30/55). |
| Size | Hit **36×36**, glyph fills hit (`.board-action-icon`). |
| Algorithm | `src/lib/focusActionable.ts` — Manhattan from current selection (or `BOARD_CENTER` 4.5,4.5 if none); ties → smaller seat piece index. |

## Do / Don't

- Do: keep pure helper in `lib/focusActionable`; page wires candidates from own unfinished non-trapped pieces; keep focus @3.
- Don't: put focus under knowledge counter / budgets; revive end-turn at @3; revive focus@9 or Material focus glyph.
