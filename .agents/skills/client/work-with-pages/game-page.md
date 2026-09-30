# Game page (`GamePage.vue`)

Read with the [core pages skill](SKILL.md) for the `game` route. Board/presence/move
chrome: `work-with-game-board` (+ `peek.md` / `focus.md`). CSS: `work-with-styles/board.md`.
Store I/O: `work-with-stores/game.md`. Leave/status chrome: [shell.md](shell.md) in App.

## Composition

- Board in scroll region; unfinished pieces + finish travel + fade + return travel;
  holes for `removedTaskKeys`; grille overlays (`GRILLE_ANIM_MS = 1000`; defer drops
  while catapult queue busy — SC-BOARD-29/30); catapult sequential overlays
  (`CATAPULT_ANIM_MS = 1000`, land→overlay→fling, D13, `isBoardBusy` — SC-BOARD-27/32).
- Rescue / push affordances **top-center**; **top** presence row (seated reserves
  height — SC-PRESENCE-26); sticky bottom `.game-hud` only when seated (strip row /
  HUD ≤~420 → 2×2; **no** chip/`q-menu`).
- Return → green `undo` when `canReturn` (no confirm modal); finish 2×2 → nearest
  legal center; peek eye top-center + shared Q&A; end-turn `skip_next` right-center;
  budgets beside avatar; say top↓ / own↑.
- Sync via `useGameStore()`; `rejoinGame(roomId)` on mount / soft-fail.
- **No** page-local leave/status/roomId — App brand-logo + `header-game-leave` + status.

## Domain notes

| Surface | Notes |
|---------|--------|
| Route | `/game/:roomId` → `GamePage`; `meta.requiresAuth`; `roomId` reconnect only — not chrome |
| Store actions | `rejoinGame` / `sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish` / `sendPeek*` / `sendEndTurn` / `sendSay` |
| Room | name `tourist`; mirror seats / turn / `removedTaskKeys` / grilles/catapults; private `budgets` / `peekOpen` / `allJailWarning` |
