# Game page (`GamePage.vue`)

Read with the [core pages skill](SKILL.md) for the `game` route. Board/presence/move
chrome: `work-with-game-board` (+ `peek.md` / `focus.md`). CSS: `work-with-styles/board.md`.
Store I/O: `work-with-stores/game.md`. Leave/status chrome: [shell.md](shell.md) in App.

## Composition

- Board in scroll region (`tileSkinCssVars()` on `.tourist-board`; CSS in
  `work-with-styles/board.md`); unfinished pieces + finish travel + fade + return travel;
  holes for `removedTaskKeys`; grille overlays (`GRILLE_ANIM_MS = 1000`; defer drops
  while catapult queue busy — SC-BOARD-29/30); catapult sequential overlays
  (`CATAPULT_ANIM_MS = 1500`, broken 450+450+300, land→overlay→fling, D13,
  `isBoardBusy` — SC-BOARD-23/24/27/32; drop-shadow SC-BOARD-58).
- Rescue / push affordances **top-center**; **top** presence row (seated reserves
  height — SC-PRESENCE-26); sticky bottom `.game-hud` only when seated (strip row /
  HUD ≤~420 → 2×2; **no** chip/`q-menu`).
- Return → green `undo` when `canReturn` (no confirm modal); finish cells per-cell
  click/outline (SC-MOVE-65); peek eye top-center + shared Q&A (dots+word badge, portrait
  slots, `.peek-answer-mini` pool; testid `peek-answer-chip` on pool cards;
  same answer MAY fill multiple slots; no used-state chrome — SC-BOARD-49/50…54;
  peeks=0 finite: hide eye on flipped too; details `work-with-game-board/peek.md`);
  presence clock: say **12** / focus **9** / end-turn `skip_next` **3**;
  budgets beside avatar; say top↓ / own↑.
- Sync via `useGameStore()`; `rejoinGame(roomId)` on mount / soft-fail.
- Solo end copy (timer vs steps): watch own `timeExpired` → `inferSoloEndModalKind` from
  `src/lib/soloEndModalKind.ts` (legal peek = free unfinished on present `*`; **!trapped** —
  SC-PRESENCE-21/35); details in `work-with-game-board`.
- **No** page-local leave/status/roomId — App brand-logo + `header-game-leave` + status.

## Domain notes

| Surface | Notes |
|---------|--------|
| Route | `/game/:roomId` → `GamePage`; `meta.requiresAuth`; `roomId` reconnect only — not chrome |
| Store actions | `rejoinGame` / `sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish` / `sendPeek*` / `sendEndTurn` / `sendSay` |
| Room | name `tourist`; mirror seats / turn / `removedTaskKeys` / grilles/catapults; private `budgets` / `peekOpen` / `allJailWarning` |
| Solo end | Page-local `endModalKind` via `lib/soloEndModalKind` (not Pinia); vitest pure lib — `client-work-with-test` / `composables.md` |
