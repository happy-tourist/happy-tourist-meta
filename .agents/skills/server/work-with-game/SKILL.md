---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating (new seat only while waiting), reconnect grace, deferred
  pieces until playing, turn order (skip finished / time-expired), private
  steps/peeks budgets, peek / endTurn, removed task tiles, turn deadlines
  (60s / solo 5min), center finish / finishPlace, and one-step move rules in the
  happy-tourist Colyseus server — Room handlers, schema seats/phase/maxSeats/
  ready/countdown/currentTurnSessionId/turnUntil/turnBudgetSeconds/removedTaskKeys/
  connectivity/finish/timeExpired fields, or pure rules in src/game/touristMove.ts.
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating, reconnect grace, turn order, private step/peek budgets, peek resolve, and one-step move validation. Client board geometry is local CSS Grid; pieces/strip/presence/hints mirror synced seats + `currentTurnSessionId` + `removedTaskKeys`. Do not trust client-local seat assignment or client hints as authority. Do not invent legacy draughts rules. Preset **`say`** is **not** game rules — whitelist I/O + live-limit live in `work-with-messages` / `MyRoom.handleSay` (ephemeral broadcast; not `touristMove.ts`, not schema).

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; seat assign; start / ready / countdown; `turnOrder` + private `budgets` / `taskRewards` / `openPeek`; `onMessage('move'\|'peek'\|'peekAnswer'\|'endTurn'\|'ready'\|'say')`; `onDrop`/`onReconnect`/`onLeave` |
| Schema | `src/rooms/schema/MyRoomState.ts` | `phase` + `maxSeats` + `countdownRemaining` + `started` (legacy) + `seats` (+ `ready` / `finishPlace` / `timeExpired`) + piece `finished` + `currentTurnSessionId` + `turnUntil` + `turnBudgetSeconds` + `nextFinishPlace` + `removedTaskKeys` |
| Rules | `src/game/touristMove.ts` | Pure validate/apply one-step move; playable includes removed `*`; `hasLegalMove` / `hasLegalPeek`; occupancy ignores `finished`; center landing is room side-effect |
| Registration | `src/app.config.ts` | Room name must be `tourist` for client lobby |

Constants: `RECONNECT_GRACE_SECONDS = 30`, `COUNTDOWN_SECONDS = 5`, `TURN_BUDGET_SECONDS = 60`, `SOLO_BUDGET_SECONDS = 300` in `MyRoom.ts` (test overrides via `setTurnBudgetsForTests` / `resetTurnBudgets`).

## Seating (shipped — game/pieces + game/start)

- Create options: `{ maxSeats: 2|3|4 }` (invalid → default **2**). Synced `state.maxSeats`.
- New seat **only** while `phase === 'waiting'` and `seats.size < maxSeats`: unique `touristId` 1…4, `connected=true`, `reconnectUntil=0`, `ready=false`, `finishPlace=0`, `timeExpired=false`.
- Join during `countdown` or `playing` → spectator (no seat/kind/pieces), even if capacity is free after leave/grace.
- **Pieces deferred:** in `waiting`/`countdown` — seat+kind only (no pieces). On enter `playing` — `materializePiecesForAllSeats()` (four pieces N/E/S/W on free start cells).
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
| **Consented leave** (`client.leave` / intentional exit) | Immediate `onLeave` → delete seat (all four pieces). Kind/cells back to pools. Subsequent join gets a seat **only** while `phase === 'waiting'` and under maxSeats. |
| **Unexpected drop** (`onDrop`) | Seated only: `connected=false`, `reconnectUntil=now+30s`, `allowReconnection(client, 30)`; **hold** seat + pieces. Spectators: no grace. Does not cancel countdown. |
| **Reconnect within grace** (`onReconnect`) | Same seat/kind/pieces; `connected=true`, `reconnectUntil=0`. |
| **Grace timeout / denied** | Permanent remove as consented leave; no seating reopen outside `waiting`. |
| **Empty seated** | After permanent remove, if `seats.size === 0` → `this.disconnect()` even if spectators remain. |

LobbyRoom: **no** grace / `allowReconnection` — see `work-with-rooms` (D7).

## Turn Order (shipped — D1 / D4 + game/finish + turn timer + steps)

- Room-private `turnOrder: string[]` (join order of seated sessionIds) — **not** in schema.
- Synced `currentTurnSessionId` = current **eligible** seated `sessionId` (`finishPlace === 0` && `!timeExpired`), or `""` if none.
- First seated → set turn; later seats append only while joining in `waiting`. Join during `countdown`/`playing` is spectator — turn order unchanged (SC-MOVE-19).
- **Successful `move` does NOT advance the turn** (SC-MOVE-35). Advance via: `endTurn`, auto-end (no legal move ∧ no legal peek), turn timeout, or full-seat finish (`finishPlace` assigned).
- Permanent seat remove: drop from `turnOrder`; if removed was current → next eligible (or `""`).
- `onDrop` / offline grace: **do not** change `currentTurnSessionId` for **non-finished** seats (turn waits); turn deadline **keeps ticking** (SC-MOVE-26). Finished / time-expired seats are never eligible.
- Having a current-turn seat does **not** allow moves before `phase === 'playing'`.

## Private budgets / peek / end-turn (shipped — game/move + game/board)

Room-private (not schema):

| Bookkeeping | Meaning |
|-------------|---------|
| `budgets: Map<sessionId, { steps, peeks, infinite, peekedThisTurn }>` | Owner-only; resend via `client.send('budgets', { steps, peeks, infinite, peekedThisTurn })` on grant / change / reconnect |
| `taskRewards: Map<"r,c", 1\|2\|3>` | Pregen on `enterPlaying` — bag **28×1 / 14×2 / 6×3**; revealed only in private `peekOpen` |
| `openPeek` | At most one unresolved peek for current seat |

Synced: `removedTaskKeys: string[]` (`"r,c"`) — holes for all clients; cells stay **walkable**.

Behavior:

- Enter `playing`: budgets start **0/0**; seed rewards; clear `removedTaskKeys`; grant current seat.
- Multi (≥2 eligible) on becoming current: `steps++`, `peeks++`, reset `peekedThisTurn`. Solo (1 eligible): `infinite=true` — no +1/+1, no end-turn, unlimited peeks.
- `move`: spend 1 step (finite); **keep** turn; then `maybeAutoEndTurn`.
- `peek` `{ side }` → private `peekOpen` `{ side, row, col, reward }`; `peekAnswer` `{ correct }` → spend peek (finite), Correct adds reward steps, always remove tile; multi marks `peekedThisTurn`.
- `endTurn` (no payload): multi only; open peek → force incorrect first; then `advanceTurn` (+1/+1 next).
- Auto-end: after move / peekAnswer / budget change, if multi and no legal move and no legal peek → `advanceTurn`.
- Timeout + open peek → force incorrect (remove tile) then advance / solo `timeExpired` (SC-MOVE-42).
- Consented/permanent leave with open peek → same force incorrect (remove tile) before seat delete.

## Turn deadline (shipped — game/move)

- Only while `phase === 'playing'` with an eligible current seat: synced `turnUntil` (unix ms) + `turnBudgetSeconds` (60 multi / 300 solo). Else both `0`.
- ≥2 eligible → 60s fresh deadline on each **turn assign** (endTurn / auto / leave-advance / enter playing / finish-advance). Exactly 1 eligible → fresh **300s** (solo), including mid-turn when others finish/leave; solo seat’s own moves **preserve** remaining budget.
- Timeout: ≥2 eligible → force-close open peek as wrong if any, then `advanceTurn` without moving pieces; solo → `seat.timeExpired=true`, clear deadline, reject further moves/peeks; room stays until leave/grace.
- Waiting/countdown: no turn auto-pass. Clear deadline on dispose / countdown restart / no eligible.

## Move Rules (shipped — D2 / D3 + game/finish + steps)

- Message: `room.send('move', { side: 'N'|'E'|'S'|'W', row, col })` — `side` = own piece; `row`/`col` = target. **No** separate `finish` message.
- Pure module `src/game/touristMove.ts`: playable = start + task + center + **removed task cells** (mirrors client `LAYOUT` holes); Chebyshev distance === 1; occupancy = unfinished pieces only; reject already-finished mover piece; helpers `hasLegalMove` / `hasLegalPeek(removedKeys)`.
- `MyRoom.onMessage('move')`: require `phase === 'playing'` + seated + `finishPlace === 0` + `!timeExpired` + current turn + (`infinite` ∨ `steps > 0`) → validate → update `row`/`col`; finite → `steps--` + `sendBudgets`; if target is center → `piece.finished=true`; when seat’s 4th piece finishes → assign `finishPlace = nextFinishPlace++` then advance (skip finished / time-expired); else **do not** advance solely because move succeeded — run auto-end check. Reject → no state change.
- Finished / time-expired seat may still `say`; cannot move/peek. Finished seats count toward `maxSeats` until leave/grace.
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
| Board UI | Client-only `LAYOUT` in `GamePage`; server does **not** sync tile kinds; holes from synced `removedTaskKeys` |
| Synced state | `phase`, `maxSeats`, `countdownRemaining`, `started` (legacy), `seats` Map → `touristId` + `pieces` (+ `finished`; may be empty pre-playing) + connectivity + `ready` + `finishPlace` + `timeExpired`, `currentTurnSessionId`, `turnUntil`, `turnBudgetSeconds`, `nextFinishPlace`, `removedTaskKeys` |
| Messages | `move` `{ side, row, col }` via `sendMove` (no turn advance); `peek` `{ side }` / `peekAnswer` `{ correct }` / `endTurn` via store; `ready` via `sendReady` |
| Private | Server → owner `budgets` `{ steps, peeks, infinite, peekedThisTurn }`; `peekOpen` `{ side, row, col, reward }` |
| Say (ephemeral) | `say` `{ presetId: hello\|luck }` → broadcast; readiness preset only via `ready` — see `work-with-messages` |
| Reconnect | Colyseus token; client `localStorage` + `reconnect` then `joinById`; resend `budgets` on reclaim |
| Presence / hints | Client-only chrome (own counters + end-turn + eye); selection + red targets local to current-turn client |

Client-local / room-private only (do **not** put on schema): Pinia status strings; `selectedSide` / legal hints; presence layout; say bubbles (`sayEvents` / `liveSays`); `turnOrder` / `budgets` / `taskRewards` / `openPeek` (room-private). Lobby leave-before-enter / quiet listing stays in `work-with-rooms` / client `work-with-lobby`.

## Architecture Preference

```
onMessage('move') → parse { side, row, col } → reject if finished / no steps
                  → pure validate (touristMove; occupancy ignores finished)
                  → if ok: write row/col; spend step; if center → finished + maybe finishPlace
                  → do NOT advance solely on move; maybeAutoEndTurn / finish-advance

onMessage('peek'|'peekAnswer'|'endTurn') → budgets / remove tile / advanceTurn
```

## Implementation Checklist

- [x] Register room as `tourist` (+ `.enableRealtimeListing()`); tests/loadtest use `tourist`.
- [x] Product sync: `phase` / `maxSeats` / `countdownRemaining` + `seats` (+ connectivity + `ready`) + `currentTurnSessionId` + `removedTaskKeys`; new seat only while `waiting` under maxSeats (no `maxClients` seat lock; no mid-game seat).
- [x] Start: auto/all-ready countdown; `onMessage('ready')`; move gated on `phase === 'playing'`; mocha SC-START-* / SC-MOVE-18.
- [x] Unexpected drop grace 30 s + `allowReconnection`; consented leave immediate; empty seated → dispose; mocha SC-PIECE-*.
- [x] `turnOrder` + private budgets + `onMessage('move'|'peek'|'peekAnswer'|'endTurn')` + `touristMove.ts` (removed `*` walkable); mocha SC-MOVE-33… / SC-BOARD-07….

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Keep seating pools and start-cell geometry in Room (or a pure helper), not in schema files.
- Prefer **`phase`** over legacy `started` for start/move gates.
- Keep reconnect grace only on tourist seated players — not LobbyRoom.
- Keep `turnOrder` / `budgets` / `taskRewards` room-private; sync only `currentTurnSessionId` + `removedTaskKeys` (not steps/peeks).
- Reject illegal / pre-playing moves/peeks server-side without mutating state.
- Prefer pure rules modules + thin Room glue.

## Don't

- Cap the room with `maxClients = maxSeats` — spectators are allowed; seated ≤ maxSeats via seats check.
- Treat unexpected drop as immediate seat delete without grace, or advance turn on drop.
- Cancel countdown solely because a seated player drops/leaves.
- Let spectators hold a room with zero seated players.
- Accept moves while `phase !== 'playing'`.
- Advance the turn solely because a `move` succeeded.
- Put steps/peeks on public `Seat` schema (owner-only `budgets` message).
- Treat draughts cell encoding or move UX as the product canon.
- Trust client layout constants or local hints as game authority.
- Put authoritative logic only in Express routes.
- Change room name or message shapes without updating the client in lockstep.

## Related

- Client board + presence: `.agents/skills/client/work-with-game-board/SKILL.md`
- Schema seats + turn + removed tiles: `.agents/skills/server/work-with-schema/SKILL.md`
- Messages (`move` / `peek` / `endTurn` / `say`): `.agents/skills/server/work-with-messages/SKILL.md`
- Rooms / registration / LobbyRoom policy: `.agents/skills/server/work-with-rooms/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
