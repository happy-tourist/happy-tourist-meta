---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating (waiting-only), reconnect grace, deferred pieces, turn
  order (skip finishPlace>0), private knowledge, open-on-land peek, one
  move|return board action + free rescue, exclusive grille/catapult seed
  15%/15%, paced trap pipeline / pendingTurnAdvance / idle ensure+absorb,
  remainingTurns grille tick, 60s multi turn deadline, all-jail wait → advance,
  penultimate cut, center finish, or one-step rules in happy-tourist-server
  (`MyRoom`, schema, src/game/touristMove.ts). Push/endTurn/solo leftover
  clock/density options/all-jail teleport retired.
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating, reconnect grace, turn order, private **knowledge**, open-on-land peek resolve, **fixed exclusive** grille/catapult seed **15%/15%**, paced trap pipeline (`MOVE_ANIM_MS` / `CATAPULT_ANIM_MS`; deferred `pendingTurnAdvance`; idle ensure+absorb / re-eval — SC-MOVE-91…93/96), free rescue, returnFromFinish, **60s multi turn deadline**, all-jail **wait** (no teleport/`allJailWarning`) → advance, penultimate cut, and one-step move validation. **Retired:** steps/peeks `budgets`, push, endTurn, solo leftover wall-clock / permanent `timeExpired`, create density options, ranked finish places, all-jail start teleport.

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`. Messages: `work-with-messages`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; async create via `roomContentSnapshot`; seat assign; start/ready/countdown; `turnOrder` + private `knowledge` / `boardActionUsed` / task deck / BoardGeometry / `openPeek` / exclusive traps; paced trap pipeline; `onMessage('move'|'rescue'|'returnFromFinish'|'peek'|'peekPlace'|'peekSubmit'|'push'|'endTurn'|'ready'|'say')` (push/peek-open/endTurn reject); `onDrop`/`onReconnect`/`onLeave` |
| Schema | `src/rooms/schema/MyRoomState.ts` | `phase` + `maxSeats` + seats (`finishPlace` 0\|1 gate; `timeExpired` always false) + piece `finished`/`trapped`/`remainingTurns` + turn + `removedTaskKeys` + grilles/catapults + peek session |
| Rules | `src/game/touristMove.ts` | Pure one-step move; landable removed holes; `hasLegalMove` / `hasLegalRescue` / `hasLegalReturn`; exclusive seed helpers `exclusiveGrilleCount` / `exclusiveCatapultCount` / `pickExclusiveTrapKeys` (15% / 15%); catapult fling helpers. Push validators retired from product paths. |
| Registration | `src/app.config.ts` | Room name `tourist` |

Constants: `RECONNECT_GRACE_SECONDS = 30`, `COUNTDOWN_SECONDS = 5`. Multi turn deadline: `TURN_BUDGET_SECONDS = 60` when ≥2 eligible (`turnUntil` / `turnBudgetSeconds`; mocha `setTurnBudgetsForTests` / `resetTurnBudgets`). Solo leftover wall-clock (`SOLO_BUDGET_SECONDS`) retired — never arms permanent `timeExpired`. Trap presentation: `MOVE_ANIM_MS`, `CATAPULT_ANIM_MS=1500`, broken **1200**, `GRILLE_ANIM_MS=1000` — `setTrapPresentationBudgetsForTests` / `resetTrapPresentationBudgets`.

## Seating / start

- Create options: `{ mapId, packId, taskSetIds, maxSeats?: 1…map.players }` (`maxSeats` omit → **`min(2, map.players)`**; invalid → reject). Density keys if present are **ignored**.
- New seat only while `phase === 'waiting'` and under `maxSeats`. Pieces deferred until `enterPlaying` / `materializePiecesForAllSeats`.
- Full table (incl. `maxSeats=1`) → countdown → `enterPlaying`: materialize, seed knowledge zeros + exclusive traps, become-current first eligible → arm 60s multi deadline when ≥2 eligible.

## Turn machine (D3)

- Room-private `turnOrder`; synced `currentTurnSessionId` = eligible (`finishPlace === 0`).
- Per-seat-turn `boardActionUsed` (reset on become-current).
- **Rescue** while `!boardActionUsed` — free; does not set flag; grille clear → empty cell (`removedTaskKeys`); `remainingTurns=0`.
- **One** of `move` | `returnFromFinish` sets `boardActionUsed`, runs paced land pipeline; on idle: if Q&A cell → **open-on-land** shared peek (hold turn); else `advanceTurn`.
- Wrong peek → KEEP digit; piece stays; turn advances. Correct → +difficulty knowledge on piece + empty cell → advance.
- **60s** multi deadline keeps ticking while modal open; timeout → `peekClose` incorrect + `reason:'timeout'` → advance. Non-current leave MUST NOT re-arm a fresh deadline for the current seat.
- All-jail: pieces stay trapped with `remainingTurns`; **no** teleport / **no** `allJailWarning`; after settle → **`advanceTurn`**.
- `push` / client `peek` open / `endTurn` → reject. No auto-end from peeks/steps; no leftover solo wall-clock.

## Private knowledge (D2)

Room-private `knowledge: Map<sessionId, { delivered, pieces: Map<pieceId, number>, outcome?: 'finish'|'cut' }>`.

- `client.send('knowledge', { delivered, pieces: Record, outcome? })` to owner only.
- Correct solve → piece carried `+= difficulty`; re-send.
- Piece finish (center) → `delivered += carried`; piece carried `= 0`; re-send; full seat finish → `finishPlace=1` + `outcome:'finish'`.
- Penultimate cut (`maxSeats≥2`, one other eligible): last seat `finishPlace=1`, burn unfinished carried to 0 **without** adding to delivered, finish those pieces; `outcome:'cut'`; re-send (D8). `maxSeats===1` skips cut.

## Traps (D1 / D5 / D6)

- Exclusive partition at seed: `grilleCount = min(round(n*0.15), n)`; `catapultCount = min(round(n*0.15), n - grilleCount)`; rest = Q&A. Never overlap; traps never bind Q&A.
- Land grille → trap + `remainingTurns=3` + `holdingGrilleKeys`. Become-current: if `remainingTurns===1` auto-free + empty cell; else decrement.
- Catapult consume / grille clear → empty via `removedTaskKeys` (no underlying task).
- Paced pipeline unchanged (one hop + presentation budgets; pending turn absorb / idle re-eval).

## Finish / timeExpired

- `finishPlace`: `0` active; `1` after full finish or cut — **no** monotonic places / place UX.
- `timeExpired`: schema retained; multi 60s turn clock **never** sets leftover lock (solo leftover wall-clock retired).

## Do / Don't

| Do | Don't |
|----|--------|
| Seed exclusive 15%/15%; private knowledge (+ `outcome?`); open-on-land after pipeline idle | Revive density create options, steps/peeks budgets, push, endTurn, solo leftover `timeExpired` |
| Tick `remainingTurns` on become-current; all-jail **wait** → advance (no teleport/`allJailWarning`) | Ranked finish places; public knowledge fields; client peek-open as land substitute; all-jail start teleport |
| Arm 60s multi `turnUntil` when ≥2 eligible; timeout → `peekClose.reason:'timeout'` + advance | Assert leftover ∞ / permanent leftover lock / density seed counts 22/35% / 12% grille |
| Mocha: knowledge helpers / `boardActionUsed` / trap presentation + turn-budget overrides | — |

## Related

- Messages contract: `work-with-messages`
- Schema fields: `work-with-schema`
- Client HUD: `../client/work-with-game-board/` (+ `knowledge.md` / `peek.md` / `focus.md`)
