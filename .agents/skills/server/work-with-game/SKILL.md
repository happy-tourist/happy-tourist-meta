---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating, reconnect grace, turn order, and one-step move rules in
  the happy-tourist Colyseus server — Room handlers, schema seats/phase/
  maxSeats/ready/countdown/currentTurnSessionId/connectivity, or pure rules in
  src/game/touristMove.ts.
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating, reconnect grace, turn order, and one-step move validation. Client board geometry is local CSS Grid; pieces/strip/presence/hints mirror synced seats + `currentTurnSessionId`. Do not trust client-local seat assignment or client hints as authority. Do not invent legacy draughts rules. Preset **`say`** is **not** game rules — whitelist I/O + live-limit live in `work-with-messages` / `MyRoom.handleSay` (ephemeral broadcast; not `touristMove.ts`, not schema).

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; seat assign; start phases / ready / countdown; `turnOrder` + turn hooks; `onMessage('move'|'ready'|'say')`; `onDrop`/`onReconnect`/`onLeave` |
| Schema | `src/rooms/schema/MyRoomState.ts` | `phase` + `maxSeats` + `countdownRemaining` + `started` (legacy) + `seats` (+ `ready`) + `currentTurnSessionId` |
| Rules | `src/game/touristMove.ts` | Pure validate/apply one-step move (no Colyseus I/O) |
| Registration | `src/app.config.ts` | Room name must be `tourist` for client lobby |

Constants: `RECONNECT_GRACE_SECONDS = 30`, `COUNTDOWN_SECONDS = 5` in `MyRoom.ts`.

## Seating (shipped — game/pieces + game/start)

- Create options: `{ maxSeats: 2|3|4 }` (invalid → default **2**). Synced `state.maxSeats`.
- While `seats.size < maxSeats` **in any phase** (`waiting` / `countdown` / `playing`): join gets a seat — unique `touristId` 1…4, **exactly four pieces** (one per side `N|E|S|W`). Starts: N row0 cols3–6; E col9 rows3–6; S row9 cols3–6; W col0 rows3–6. New seat: `connected=true`, `reconnectUntil=0`, `ready=false`.
- When `seats.size === maxSeats`: further joiners are spectators (no seat/pieces). No `maxClients = maxSeats`.
- Metadata for lobby: `{ title, status, maxSeats, seats }` — `seats` = occupied seated count (not `clients`).
- Assign only in Room lifecycle — client never invents seats.

## Start phases / ready / countdown (shipped — game/start)

- Synced `phase`: `waiting` → `countdown` → `playing`. New room starts `waiting`.
- Full table (`seats.size === maxSeats` while `waiting`) → immediate `countdown` (5…1 via `clock.setTimeout`).
- Underfilled (`≥2` seated, `< maxSeats`): `onMessage('ready')` one-shot → `seat.ready=true` + broadcast say preset `ready`; when all seated ready → same countdown. Solo seated: reject ready.
- Leave/drop during countdown does **not** cancel countdown; remaining seats’ `ready` marks are **not** cleared on leave.
- Legacy `started`: set true from countdown onward; **client/canon is phase-first** (`playing` unlocks moves). Prefer `phase`, not `started`, for new logic.

## Leave And Reconnect (shipped — D1 / D4)

| Path | Behavior |
|------|----------|
| **Consented leave** (`client.leave` / intentional exit) | Immediate `onLeave` → delete seat (all four pieces). Kind/cells back to pools. Seats reopen while `seats.size < maxSeats` in any phase. |
| **Unexpected drop** (`onDrop`) | Seated only: `connected=false`, `reconnectUntil=now+30s`, `allowReconnection(client, 30)`; **hold** seat + pieces. Spectators: no grace. Does not cancel countdown. |
| **Reconnect within grace** (`onReconnect`) | Same seat/kind/pieces; `connected=true`, `reconnectUntil=0`. |
| **Grace timeout / denied** | Permanent remove as consented leave. |
| **Empty seated** | After permanent remove, if `seats.size === 0` → `this.disconnect()` even if spectators remain. |

LobbyRoom: **no** grace / `allowReconnection` — see `work-with-rooms` (D7).

## Turn Order (shipped — D1 / D4)

- Room-private `turnOrder: string[]` (join order of seated sessionIds) — **not** in schema.
- Synced `currentTurnSessionId` = current seated `sessionId`, or `""` if no seated.
- First seated → set turn; later seats (incl. mid-game) append to end.
- Successful move → advance to next in circle (solo wraps to self).
- Permanent seat remove: drop from `turnOrder`; if removed was current → next (or `""` if empty → dispose).
- `onDrop` / offline grace: **do not** change `currentTurnSessionId` (turn waits).
- Having a current-turn seat does **not** allow moves before `phase === 'playing'`.

## Move Rules (shipped — D2 / D3)

- Message: `room.send('move', { side: 'N'|'E'|'S'|'W', row, col })` — `side` = own piece; `row`/`col` = target.
- Pure module `src/game/touristMove.ts`: playable = start + task + center (mirrors client `LAYOUT`); Chebyshev distance === 1; occupancy includes own pieces.
- `MyRoom.onMessage('move')`: require `phase === 'playing'` + seated + current turn → validate/apply → update piece `row`/`col` → advance turn; reject → no state change.
- No pass; no draughts `{ from, to }` encoding.

## Authority

- Server state is the only seating / turn / move truth. Reject illegal actions; leave state unchanged.
- Client tourist layout + local hints are **UI** — not authority.
- Wire protocol for turns/moves MUST stay lockstep with the client. Do not reintroduce legacy draughts encoding.
- Do not put rules in Express HTTP routes.

## Client Contract (align server to this)

| Field / message | Client expectation |
| --- | --- |
| Room name | `tourist` |
| Board UI | Client-only `LAYOUT` in `GamePage`; server does **not** sync tile kinds |
| Synced state | `phase`, `maxSeats`, `countdownRemaining`, `started` (legacy), `seats` Map → `touristId` + four `pieces` + connectivity + `ready`, `currentTurnSessionId` |
| Message | Client → server `move` `{ side, row, col }` via `sendMove`; `ready` (empty) via `sendReady` |
| Say (ephemeral) | `say` `{ presetId: hello\|luck }` → broadcast; readiness preset only via `ready` — see `work-with-messages` |
| Reconnect | Colyseus token; client `localStorage` + `reconnect` then `joinById` |
| Presence / hints | Client-only chrome; selection + red targets local to current-turn client |

Client-local / room-private only (do **not** put on schema): Pinia status strings; `selectedSide` / legal hints; presence layout; say bubbles (`sayEvents` / `liveSays`); `turnOrder` (room-private). Lobby leave-before-enter / quiet listing stays in `work-with-rooms` / client `work-with-lobby`.

## Architecture Preference

```
onMessage('move') → parse { side, row, col } → pure validate/apply (touristMove)
                  → if ok: write piece row/col + advance currentTurnSessionId
```

## Implementation Checklist

- [x] Register room as `tourist` (+ `.enableRealtimeListing()`); tests/loadtest use `tourist`.
- [x] Product sync: `phase` / `maxSeats` / `countdownRemaining` + `seats` (+ connectivity + `ready`) + `currentTurnSessionId`; seating while under maxSeats in any phase (no `maxClients` seat lock).
- [x] Start: auto/all-ready countdown; `onMessage('ready')`; move gated on `phase === 'playing'`; mocha SC-START-* / SC-MOVE-18.
- [x] Unexpected drop grace 30 s + `allowReconnection`; consented leave immediate; empty seated → dispose; mocha SC-PIECE-*.
- [x] `turnOrder` + `onMessage('move')` + `src/game/touristMove.ts`; mocha SC-MOVE-*.

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Keep seating pools and start-cell geometry in Room (or a pure helper), not in schema files.
- Prefer **`phase`** over legacy `started` for start/move gates.
- Keep reconnect grace only on tourist seated players — not LobbyRoom.
- Keep `turnOrder` room-private; sync only `currentTurnSessionId`.
- Reject illegal / pre-playing moves server-side without mutating state.
- Prefer pure rules modules + thin Room glue.

## Don't

- Cap the room with `maxClients = maxSeats` — spectators are allowed; seated ≤ maxSeats via seats check.
- Treat unexpected drop as immediate seat delete without grace, or advance turn on drop.
- Cancel countdown solely because a seated player drops/leaves.
- Let spectators hold a room with zero seated players.
- Accept moves while `phase !== 'playing'`.
- Treat draughts cell encoding or move UX as the product canon.
- Trust client layout constants or local hints as game authority.
- Put authoritative logic only in Express routes.
- Change room name or message shapes without updating the client in lockstep.

## Related

- Client board + presence: `.agents/skills/client/work-with-game-board/SKILL.md`
- Schema seats + turn: `.agents/skills/server/work-with-schema/SKILL.md`
- Messages (`move` / `say`): `.agents/skills/server/work-with-messages/SKILL.md`
- Rooms / registration / LobbyRoom policy: `.agents/skills/server/work-with-rooms/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
