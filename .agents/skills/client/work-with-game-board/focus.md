# Focus nearest actionable tourist

Read this when changing the focus control or nearest-actionable selection.

## Contract (product)

| Topic | Rule |
|-------|------|
| Affordance | Illustrated `src/assets/board/focus.png` (`<img>`) on own presence marker **below end-turn**, hit-center at the **bottom-right corner of the 72px avatar’s bounding square** (`left/top: calc(50% + 36px); transform: translate(-50%,-50%)`) when `canShowFocusControl` — own turn, playing, actionable unfinished non-trapped pieces exist; **no** Material `center_focus_strong` / solid colored circular chrome; **not** under peeks in budgets; **not** @9. Aria: `game.focusActionable`. |
| Size | Same as say/end-turn: hit **36×36**, glyph fills hit (`.board-action-icon` 36×36). |
| Algorithm | `src/lib/focusActionable.ts` — Manhattan from current selection (or `BOARD_CENTER` 4.5,4.5 if none); ties → smaller seat piece index. |

## Do / Don't

- Do: keep pure helper in `lib/focusActionable`; page wires candidates from own unfinished non-trapped pieces; keep focus under end-turn at avatar square corner (SC-PRESENCE-30/33/37).
- Don't: put focus geometry in Pinia; invent a server message for focus; revive focus under peeks in budgets; revive focus on marker @9 (`left:0; top:50%`); revive offset hits (`top:-40` / `left:-8` / `right:-8`); revive Material focus glyph or colored circular chrome; pad HUD `padding-left` for focus@9; shrink glyph below the 36×36 hit.
