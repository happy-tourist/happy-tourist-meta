# Game page (`GamePage.vue`)

Read with the [core pages skill](SKILL.md) for the `game` route. Board/presence/move
chrome: `work-with-game-board` (+ `peek.md` / `focus.md` / `knowledge.md`). CSS:
`work-with-styles/board.md`. Store I/O: `work-with-stores/game.md`. Leave/status:
[shell.md](shell.md) in App.

## Composition

- Board in scroll region (`tileSkinCssVars()` on `.tourist-board`); unfinished pieces +
  finish travel + fade + return travel; holes for `removedTaskKeys` (still landable);
  grille overlays (`GRILLE_ANIM_MS = 1000`; defer drops while catapult queue busy);
  catapult sequential overlays (`CATAPULT_ANIM_MS = 1500`, broken 450+450+300,
  drop-shadow SC-BOARD-58).
- Rescue/return above sprite then +20px down; **no** peek-open / push affordances.
- Top presence row; sticky bottom `.game-hud` when seated (strip row / HUD ≤~420 → 2×2).
- Presence clock: say **12** / focus **3** (end-turn retired — SC-PRESENCE-53/55) +
  60s turn ring when `turnUntil` armed; knowledge counter + lightbulb beside avatar;
  **strip-only** yellow TR carried (+ lightbulb); red TR `remainingTurns` above grille
  (`work-with-game-board/knowledge.md`).
- Open-on-land shared Q&A modal (place/submit only — `peek.md`); Notify peek toasts;
  finish\|cut knowledge modal on `finishPlace` 0→1 by `knowledgeOutcome` (no place /
  leftover timer / all-jail teleport modals).
- Sync via `useGameStore()`; `rejoinGame(roomId)` on mount / soft-fail.
- **No** page-local leave/status/roomId — App brand-logo + `header-game-leave` + status
  (incl. 10s peek outcome flash — SC-PRESENCE-36; toast from store Notify).

## Domain notes

| Surface | Notes |
|---------|--------|
| Route | `/game/:roomId` → `GamePage`; `meta.requiresAuth`; `roomId` reconnect only — not chrome |
| Store actions | `rejoinGame` / `sendMove` / `sendRescue` / `sendReturnFromFinish` / `sendPeekPlace` / `sendPeekSubmit` / `sendSay` (`sendPush`/`sendEndTurn`/`sendPeek` retired) |
| Room | name `tourist`; mirror seats/pieces/`remainingTurns` / `turnUntil` / traps; private `knowledge` (`outcome?`); `allJailWarning` inert |
| Economy | One board action per turn; free rescue; fixed exclusive traps **15%/15%**; multi 60s `turnUntil` (≥2 eligible); leftover/single-player no deadline |
