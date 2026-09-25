---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating (new seat only while waiting), reconnect grace, deferred
  pieces until playing, turn order (skip finished / time-expired), private
  steps/peeks budgets (become-current grant: multi +1/+1; solo +1 step;
  already-current→solo no re-grant), peek / endTurn, grille + catapult density
  seed / paced trap pipeline / pendingTurnAdvance / idle re-eval (SC-MOVE-93) /
  fling / rescue / push / returnFromFinish / all-jail, removed task tiles, turn
  deadlines (60s / solo 5min), center finish / finishPlace, and one-step move
  rules in the happy-tourist Colyseus server —
  Room handlers, schema seats/phase/maxSeats/ready/countdown/currentTurnSessionId/
  turnUntil/turnBudgetSeconds/removedTaskKeys/holdingGrilleKeys/
  revealingCatapultKeys/brokenCatapultKeys/piece trapped/connectivity/finish/
  timeExpired fields, or pure rules in src/game/touristMove.ts.
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating, reconnect grace, turn order, private step/peek budgets, peek resolve, grille + catapult seed, **paced** `resolveCellTraps` / trap pipeline (one hop + presentation budget; shared `MOVE_ANIM_MS` / `CATAPULT_ANIM_MS`; no `presentationDone`), deferred `pendingTurnAdvance` until pipeline idle (**idle re-eval** auto-end/solo when no pending — SC-MOVE-93), rescue/push/return/all-jail, and one-step move validation. Client board geometry is local CSS Grid; pieces/strip/presence/hints mirror synced seats + `currentTurnSessionId` + `removedTaskKeys` + `holdingGrilleKeys` + short-lived `revealingCatapultKeys` / `brokenCatapultKeys`. Do not trust client-local seat assignment or client hints as authority. Do not invent legacy draughts rules. Preset **`say`** is **not** game rules — whitelist I/O + live-limit live in `work-with-messages` / `MyRoom.handleSay` (ephemeral broadcast; not `touristMove.ts`, not schema).

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; async create via `roomContentSnapshot`; seat assign; start/ready/countdown; `turnOrder` + private `budgets` / task deck / BoardGeometry / `openPeek` / hidden grilles+catapults; paced trap pipeline; `onMessage('move'|'rescue'|'push'|'returnFromFinish'|'peek'|'peekPlace'|'peekSubmit'|'endTurn'|'ready'|'say')`; `onDrop`/`onReconnect`/`onLeave` |
| Schema | `src/rooms/schema/MyRoomState.ts` | `phase` + `maxSeats` + `countdownRemaining` + `started` (legacy) + `seats` (+ `ready` / `finishPlace` / `timeExpired`) + piece `finished`/`trapped` + `currentTurnSessionId` + `turnUntil` + `turnBudgetSeconds` + `nextFinishPlace` + `removedTaskKeys` + `holdingGrilleKeys` + `revealingCatapultKeys` + `brokenCatapultKeys` |
| Rules | `src/game/touristMove.ts` | Pure validate/apply one-step move; playable layout; landing excludes removed holes; `hasLegalMove` / `hasLegalPeek` / `hasLegalRescue` / `hasLegalReturn` / `farSideCell` / `validateTouristPush` / `hasLegalPush`; occupancy ignores `finished` (trapped still occupy); center landing is room side-effect; grille/catapult density + `catapultFlingCandidates` / `pickCatapultFlingDest` / `shuffleArray` |
| Registration | `src/app.config.ts` | Room name must be `tourist` for client lobby |

Constants: `RECONNECT_GRACE_SECONDS = 30`, `COUNTDOWN_SECONDS = 5`, `TURN_BUDGET_SECONDS = 60`, `SOLO_BUDGET_SECONDS = 300` in `MyRoom.ts` (test overrides via `setTurnBudgetsForTests` / `resetTurnBudgets`).

## Seating (shipped — game/pieces + game/start)

- Create options: `{ maxSeats: 2|3|4, grilleDensity?: 'few'|'medium'|'many', catapultDensity?: 'few'|'medium'|'many' }` (`maxSeats` invalid → **2**; either density invalid/omit → **medium**). Synced `state.maxSeats`; both densities are room-private until seed.
- New seat **only** while `phase === 'waiting'` and `seats.size < maxSeats`: unique `touristId` 1…4, `connected=true`, `reconnectUntil=0`, `ready=false`, `finishPlace=0`, `timeExpired=false`.
- Join during `countdown` or `playing` → spectator (no seat/kind/pieces), even if capacity is free after leave/grace.
- **Pieces deferred:** in `waiting`/`countdown` — seat+kind only (no pieces). On enter `playing` — `materializePiecesForAllSeats()` (`touristsPerPlayer` pieces keyed by pieceId on free start cells).
- When `seats.size === maxSeats` while waiting: further joiners are spectators. No `maxClients = maxSeats`.
- Metadata for lobby: `{ title, status, maxSeats, seats }` — `seats` = occupied seated count (not `clients`).
- Assign only in Room lifecycle — client never invents seats. Reconnect within grace is not a new seat.

## Start phases / ready / countdown (shipped — game/start)

- Synced `phase`: `waiting` → `countdown` → `playing`. New room starts `waiting`.
- Full table (`seats.size === maxSeats` while `waiting`) → immediate `countdown` (5…1 via `clock.setTimeout`).
- Underfilled (`≥2` seated, `< maxSeats`): `onMessage('ready')` one-shot → `seat.ready=true` + broadcast say preset `ready`; when all seated ready → same countdown. Solo seated: reject ready.
- Leave/drop during countdown does **not** cancel countdown; remaining seats’ `ready` marks are **not** cleared on leave.
- Countdown completion → `enterPlaying()`: materialize deferred pieces, set `phase=playing`, start turn deadline.
- Legacy `started`: mirror of `phase === 'playing'`; **client/canon is phase-first** (`playing` unlocks moves). Prefer `phase`, not `started`, for new logic.

## Leave And Reconnect (shipped — D1 / D4)

| Path | Behavior |
|------|----------|
| **Consented leave** (`client.leave` / intentional exit) | Immediate `onLeave` → clear holding grilles of that seat’s trapped cells (SC-PIECE-28), then delete seat (all pieces). Kind/cells back to pools. Subsequent join gets a seat **only** while `phase === 'waiting'` and under maxSeats. |
| **Unexpected drop** (`onDrop`) | Seated only: `connected=false`, `reconnectUntil=now+30s`, `allowReconnection(client, 30)`; **hold** seat + pieces; **do not** clear holding grilles. Spectators: no grace. Does not cancel countdown. |
| **Reconnect within grace** (`onReconnect`) | Same seat/kind/pieces; `connected=true`, `reconnectUntil=0`. |
| **Grace timeout / denied** | Permanent remove as consented leave; no seating reopen outside `waiting`. |
| **Empty seated** | After permanent remove, if `seats.size === 0` → `this.disconnect()` even if spectators remain. |

LobbyRoom: **no** grace / `allowReconnection` — see `work-with-rooms` (D7).

## Turn Order (shipped — D1 / D4 + game/finish + turn timer + steps)

- Room-private `turnOrder: string[]` (join order of seated sessionIds) — **not** in schema.
- Synced `currentTurnSessionId` = current **eligible** seated `sessionId` (`finishPlace === 0` && `!timeExpired`), or `""` if none.
- First seated → set turn; later seats append only while joining in `waiting`. Join during `countdown`/`playing` is spectator — turn order unchanged (SC-MOVE-19).
- **Successful `move` / `push` does NOT advance the turn** (SC-MOVE-35/72). Advance via: `endTurn`, auto-end (no legal move ∧ not (peeks≥1 ∧ live `*`) ∧ no legal rescue ∧ no legal return ∧ no legal push), turn timeout, or full-seat finish (`finishPlace` assigned).
- Permanent seat remove: drop from `turnOrder`; if removed was current → next eligible (or `""`) + `applyTurnGrant` (multi +1/+1; solo become-current +1 step). If non-current left → no re-grant; only `syncSoloInfiniteMode` + fresh solo 5:00 when applicable (SC-MOVE-40/50).
- `onDrop` / offline grace: **do not** change `currentTurnSessionId` for **non-finished** seats (turn waits); turn deadline **keeps ticking** (SC-MOVE-26). Finished / time-expired seats are never eligible.
- Having a current-turn seat does **not** allow moves before `phase === 'playing'`.

## Private budgets / peek / end-turn (shipped — game/move + game/board)

Room-private (not schema):

| Bookkeeping | Meaning |
|-------------|---------|
| `budgets: Map<sessionId, { steps, peeks, infinite, peekedThisTurn }>` | Owner-only; `infinite` = **peeks∞ only** (steps always finite). Resend `client.send('budgets', { steps, peeks, infinite, peekedThisTurn })`. `peekedThisTurn` kept for client mirror — **no** one-peek/turn gate |
| `taskDeck` / `taskPool` / `cellBindings` | Shuffled pack tasks at create; bind on first peek; flipped digit syncs via `flippedCells` |
| `openPeek` | At most one unresolved peek for current seat |
| `grilleDensity` + hidden grille keys | Create-time `few`/`medium`/`many` → **12/22/35%** of task cells (6/11/17 on 48); hidden keys never synced until reveal |
| `catapultDensity` + hidden catapult keys | Same ratios/count formula as grille (`CATAPULT_DENSITY_RATIO` = `GRILLE_DENSITY_RATIO`); independent create option; seed only on task `*`; overlap with grilles OK; hidden never synced until consume reveal |

Synced: `removedTaskKeys: string[]` (`"r,c"`) — holes for all clients; **not landable** (stand OK; leave OK). Synced: `holdingGrilleKeys: string[]` — revealed grilles currently holding a trapped piece; removed when spent (rescue / all-jail / permanent leave of that seat’s trapped cells). Synced short-lived: `revealingCatapultKeys` / `brokenCatapultKeys` — fade UI (~`CATAPULT_ANIM_MS = 1000`); cleared at hop fire / end; hidden unspent catapults stay room-private. Presentation budgets (`MOVE_ANIM_MS`, `CATAPULT_ANIM_MS`, `CATAPULT_BROKEN_PRESENTATION_MS`, `GRILLE_ANIM_MS`) are shared with the client (D14); mocha may zero them via `setTrapPresentationBudgetsForTests`.

Behavior:

- Enter `playing`: budgets start **0/0**; seed rewards + hidden grilles + hidden catapults; clear `removedTaskKeys` / `holdingGrilleKeys` / catapult reveal arrays; grant current seat (multi +1/+1; solo become-current +1 step only).
- Multi (≥2 eligible) on becoming current: `steps++`, `peeks++`. Solo (1 eligible) on becoming current: `infinite=true` (peeks∞) then **+1 step only** (no peek increment); already-current→solo (non-current leave/finish) → peeks∞ + 5:00 **without** re-grant. No end-turn while solo.
- `move`: always spend 1 step; **keep** turn; reject if piece `trapped`; after land → paced trap pipeline; then `maybeAutoEndTurn` (deferred while pipeline active) / solo step-loss. Landing on removed hole → reject.
- **Paced trap pipeline (`startTrapPipeline` / `resolveNextTrapHop`):** one next trap on current cell after land/arrival budget → shuffle → grille applies holding then idle; catapult consume + sync reveal → wait overlay budget → fire-time fling/broken → clear reveal → fling travel budget → resolve **only** dest cell. No full-chain sync in one tick; no client `presentationDone`. Fling does **not** spend an extra step. `pendingTurnAdvance` + `onTrapPipelineIdle`: pending → `advanceTurn`; else **re-eval** `maybeAutoEndTurn` + solo exhaustion (SC-MOVE-90…93).
- `rescue` `{ pieceId }`: own trapped + Chebyshev-1 free own piece + steps≥1 → −1 step; clear trapped + holding grille; then paced pipeline if a catapult remains.
- `push` `{ pusherSide, targetSessionId, targetSide, row, col }`: own free pusher + free adj target → far-side dest; −1 step; relocate **target only**; land side-effects like move (paced pipeline on target). Reject trapped pusher/target, hole, occupied, bad geometry. Solo = multi.
- `returnFromFinish` `{ pieceId, row, col }`: `finishPlace===0` + finished piece + legal center-ring cell → −1 step; unfinish onto cell → paced pipeline.
- All-jail (4 trapped): free pieces; clear 4 holding grilles; place on free starts per side; keep turn/steps/timer; private `allJailWarning` to that seat only.
- `peek` `{ pieceId }` → shared peek session; reject if trapped; bind deck task on first open; flipped re-peek free. `peekPlace`/`peekSubmit` → peeker places chips / validates order; Correct +difficulty + remove tile; Incorrect KEEP.
- `endTurn` (no payload): multi only; open peek → force incorrect KEEP first; then `advanceTurn` (+1/+1 next).
- Auto-end (multi): advance only if no legal move **and not** (peeks≥1 ∧ unfinished on live `*`) **and not** legal rescue/return/push (SC-MOVE-38/47/62/73).
- Solo step-loss: `steps===0` ∧ ¬hasLegalPeek on live `*` ∧ no legal rescue/return/push → `timeExpired` (SC-MOVE-48); distinct from timer (SC-MOVE-45).
- Timeout + open peek → force incorrect KEEP then advance / solo timer `timeExpired` (SC-MOVE-42).
- Consented/permanent leave with open peek → same force incorrect KEEP before seat delete.

## Turn deadline (shipped — game/move)

- Only while `phase === 'playing'` with an eligible current seat: synced `turnUntil` (unix ms) + `turnBudgetSeconds` (60 multi / 300 solo). Else both `0`.
- ≥2 eligible → 60s fresh deadline on each **turn assign** (endTurn / auto / leave-advance / enter playing / finish-advance). Exactly 1 eligible → fresh **300s** (solo), including mid-turn when others finish/leave; solo seat’s own moves **preserve** remaining budget.
- Timeout: ≥2 eligible → force-close open peek as incorrect KEEP if any, then `advanceTurn` without moving pieces; solo → `seat.timeExpired=true`, clear deadline, reject further moves/peeks; room stays until leave/grace.
- Waiting/countdown: no turn auto-pass. Clear deadline on dispose / countdown restart / no eligible.

## Move Rules (shipped — D2 / D3 + game/finish + steps + grilles)

- Message: `room.send('move', { pieceId, row, col })`. **No** separate `finish` message. Also `rescue` / `push` / `returnFromFinish` / `peekPlace` / `peekSubmit`.
- Pure module `src/game/touristMove.ts`: playable = start + task + center; **landing on `removedKeys` rejected**; stand/leave from hole OK; Chebyshev distance === 1; occupancy = unfinished (incl. trapped); reject move if piece `trapped`; helpers `hasLegalMove(..., removedKeys)` / `hasLegalPeek(removedKeys)` / `farSideCell` / `validateTouristPush` / `hasLegalPush` + grille/catapult density + fling helpers.
- `MyRoom.onMessage('move')`: require `phase === 'playing'` + seated + `finishPlace === 0` + `!timeExpired` + current turn + `steps > 0` → validate (holes not landable; not trapped) → update `row`/`col`; always `steps--` + `sendBudgets`; if target is center → `piece.finished=true`; else start paced trap pipeline; when seat’s 4th piece finishes → assign `finishPlace = nextFinishPlace++` then advance; else **do not** advance solely because move succeeded — run auto-end / solo step-loss (deferred if pipeline active). Reject → no state change.
- Finished / time-expired seat may still `say`; cannot move/peek/rescue/push/return. Finished seats count toward `maxSeats` until leave/grace.
- Player end-turn via `endTurn`; also auto-end and timeout. No draughts `{ from, to }` encoding.

## Authority

- Server state is the only seating / turn / move truth. Reject illegal actions; leave state unchanged.
- Client tourist layout + local hints are **UI** — not authority.
- Wire protocol for turns/moves MUST stay lockstep with the client. Do not reintroduce legacy draughts encoding.
- Do not put rules in Express HTTP routes.

## Client Contract (align server to this)

| Field / message | Client expectation |
| --- | --- |
| Room name | `tourist` |
| Board UI | Synced `grid` → client/server `boardGeometry`; holes from `removedTaskKeys`; flipped digits from `flippedCells` |
| Synced state | `phase`, `maxSeats`, `countdownRemaining`, `started` (legacy), `seats` Map → `touristId` + `pieces` (+ `finished`/`trapped`; may be empty pre-playing) + connectivity + `ready` + `finishPlace` + `timeExpired`, `currentTurnSessionId`, `turnUntil`, `turnBudgetSeconds`, `nextFinishPlace`, `removedTaskKeys`, `holdingGrilleKeys`, `revealingCatapultKeys`, `brokenCatapultKeys` |
| Messages | `move`/`rescue`/`push`/`returnFromFinish`/`peek` with **pieceId**; `peekPlace`/`peekSubmit` shared Q&A; `endTurn`/`ready`/`say` via store |
| Private | Server → owner `budgets` `{ steps, peeks, infinite` (peeks∞ only)`, peekedThisTurn }`; `allJailWarning` `{}`; peek session mostly schema-synced |
| Create | `{ maxSeats, grilleDensity?: 'few'\|'medium'\|'many', catapultDensity?: 'few'\|'medium'\|'many' }` (both default medium) |
| Say (ephemeral) | `say` `{ presetId: hello\|luck }` → broadcast; readiness preset only via `ready` — see `work-with-messages` |
| Reconnect | Colyseus token; client `localStorage` + `reconnect` then `joinById`; resend `budgets` on reclaim |
| Presence / hints | Client-only chrome (own counters + end-turn + eye); selection + red targets local to current-turn client |

Client-local / room-private only (do **not** put on schema): Pinia status strings; `selectedPieceId` / legal hints; presence layout; say bubbles; `turnOrder` / `budgets` / task deck / cell bindings / contentSnapshot / `grilleDensity` / `catapultDensity` / hidden grille+catapult keys. Shared peek session fields **are** on schema.

## Architecture Preference

```
onMessage('move') → parse { pieceId, row, col } → reject if finished / trapped / no steps
                  → pure validate (touristMove; occupancy ignores finished)
                  → if ok: write row/col; spend step; if center → finished + maybe finishPlace
                    else paced trap pipeline (one hop + presentation budget; fling then dest)
                  → do NOT advance solely on move; maybeAutoEndTurn (deferred if pipeline) / finish-advance

onMessage('rescue'|'push'|'returnFromFinish') → spend step; clear trap / relocate target / unfinish onto ring
                                               → paced pipeline on land (rescue: remaining catapult)
onMessage('peek'|'peekPlace'|'peekSubmit'|'endTurn') → shared peek / remove tile only on Correct / advanceTurn
                                           (endTurn rejects while trap pipeline active)
```

## Implementation Checklist

- [x] Register room as `tourist` (+ `.enableRealtimeListing()`); tests/loadtest use `tourist`.
- [x] Product sync: `phase` / `maxSeats` / `countdownRemaining` + `seats` (+ connectivity + `ready` + piece `trapped`) + `currentTurnSessionId` + `removedTaskKeys` + `holdingGrilleKeys` + catapult reveal arrays; new seat only while `waiting` under maxSeats (no `maxClients` seat lock; no mid-game seat).
- [x] Start: auto/all-ready countdown; `onMessage('ready')`; move gated on `phase === 'playing'`; mocha SC-START-* / SC-MOVE-18.
- [x] Unexpected drop grace 30 s + `allowReconnection`; consented leave immediate; empty seated → dispose; mocha SC-PIECE-*.
- [x] `turnOrder` + private budgets + pieceId messages + shared peek + content snapshot + `boardGeometry` + grille/catapult seed + paced trap pipeline + `touristMove.ts`; mocha SC-MOVE / SC-BOARD / SC-LOBBY map+pack create.

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Keep seating pools and start-cell geometry in Room (or a pure helper), not in schema files.
- Prefer **`phase`** over legacy `started` for start/move gates.
- Keep reconnect grace only on tourist seated players — not LobbyRoom.
- Keep `turnOrder` / `budgets` / task deck / contentSnapshot / hidden grilles/catapults room-private; sync `grid` + peek session + `flippedCells` + turn + `removedTaskKeys` + grilles/catapults + `Piece.trapped` (not steps/peeks / hidden locations).
- Reject illegal / pre-playing moves/peeks/rescues/pushes/returns server-side without mutating state.
- Prefer pure rules modules + thin Room glue.

## Don't

- Cap the room with `maxClients = maxSeats` — spectators are allowed; seated ≤ maxSeats via seats check.
- Treat unexpected drop as immediate seat delete without grace, or advance turn on drop.
- Cancel countdown solely because a seated player drops/leaves.
- Let spectators hold a room with zero seated players.
- Accept moves while `phase !== 'playing'`.
- Advance the turn solely because a `move` or `push` succeeded.
- Put steps/peeks or hidden grille/catapult locations on public schema (owner-only `budgets` / private seed maps).
- Treat draughts cell encoding or move UX as the product canon.
- Trust client layout constants or local hints as game authority.
- Put authoritative logic only in Express routes.
- Change room name or message shapes without updating the client in lockstep.

## Related

- Client board + presence: `.agents/skills/client/work-with-game-board/SKILL.md`
- Schema seats + turn + removed tiles + grilles + catapult reveal: `.agents/skills/server/work-with-schema/SKILL.md`
- Messages (`move` / `rescue` / `push` / `returnFromFinish` / `peek` / `endTurn` / `say`): `.agents/skills/server/work-with-messages/SKILL.md`
- Rooms / registration / LobbyRoom policy: `.agents/skills/server/work-with-rooms/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
