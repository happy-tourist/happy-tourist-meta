---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating, reconnect grace, turn order, and one-step move rules in
  the happy-tourist Colyseus server — Room handlers, schema seats/started/
  currentTurnSessionId/connectivity, or pure rules in src/game/touristMove.ts.
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating, reconnect grace, turn order, and one-step move validation. Client board geometry is local CSS Grid; pieces/strip/presence/hints mirror synced seats + `currentTurnSessionId`. Do not trust client-local seat assignment or client hints as authority. Do not invent legacy draughts rules. Preset **`say`** is **not** game rules — whitelist I/O + live-limit live in `work-with-messages` / `MyRoom.handleSay` (ephemeral broadcast; not `touristMove.ts`, not schema).

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; seat assign; `turnOrder` + turn hooks; `onMessage('move')` (+ `say` → see messages skill); `onDrop`/`onReconnect`/`onLeave` |
| Schema | `src/rooms/schema/MyRoomState.ts` | `started` + `seats` + `currentTurnSessionId` |
| Rules | `src/game/touristMove.ts` | Pure validate/apply one-step move (no Colyseus I/O) |
| Registration | `src/app.config.ts` | Room name must be `tourist` for client lobby |

Constant: `RECONNECT_GRACE_SECONDS = 30` in `MyRoom.ts`.

## Seating (shipped)

- Until `started` and `seats.size < 4`: join gets a seat — unique `touristId` 1…4 not used by any current seat, plus **exactly four pieces** (one per side `N|E|S|W`). For each side, pick a start cell uniformly from that side’s free starts (not occupied by any piece in the room). Starts: N row0 cols3–6; E col9 rows3–6; S row9 cols3–6; W col0 rows3–6. New seat: `connected=true`, `reconnectUntil=0`.
- Fourth seated player → `started = true` (optional metadata `status: "playing"`).
- After `started`: join does **not** get a seat or pieces (spectator). No `maxClients = 4`.
- Assign only in Room lifecycle — client never invents seats.

## Leave And Reconnect (shipped — D1 / D4)

| Path | Behavior |
|------|----------|
| **Consented leave** (`client.leave` / intentional exit) | Immediate `onLeave` → delete seat (all four pieces). Before start → kind/cells back to pools. After start → `started` stays; no new seats. |
| **Unexpected drop** (`onDrop`) | Seated only: `connected=false`, `reconnectUntil=now+30s`, `allowReconnection(client, 30)`; **hold** seat + pieces. Spectators: no grace. |
| **Reconnect within grace** (`onReconnect`) | Same seat/kind/pieces; `connected=true`, `reconnectUntil=0`. |
| **Grace timeout / denied** | Permanent remove as consented leave. |
| **Empty seated** | After permanent remove, if `seats.size === 0` → `this.disconnect()` even if spectators remain. |

LobbyRoom: **no** grace / `allowReconnection` — see `work-with-rooms` (D7).

## Turn Order (shipped — D1 / D4)

- Room-private `turnOrder: string[]` (join order of seated sessionIds) — **not** in schema.
- Synced `currentTurnSessionId` = current seated `sessionId`, or `""` if no seated.
- First seated → set turn; later seats append to end (before seating lock).
- Successful move → advance to next in circle (solo wraps to self).
- Permanent seat remove: drop from `turnOrder`; if removed was current → next (or `""` if empty → dispose).
- `onDrop` / offline grace: **do not** change `currentTurnSessionId` (turn waits).
- `started` does **not** gate moves — turns available as soon as ≥1 seated.

## Move Rules (shipped — D2 / D3)

- Message: `room.send('move', { side: 'N'|'E'|'S'|'W', row, col })` — `side` = own piece; `row`/`col` = target.
- Pure module `src/game/touristMove.ts`: playable = start + task + center (mirrors client `LAYOUT`); Chebyshev distance === 1; occupancy includes own pieces.
- `MyRoom.onMessage('move')`: seated + current turn only → validate/apply → update piece `row`/`col` → advance turn; reject → no state change.
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
| Synced state | `started`, `seats` Map → `touristId` + four `pieces` + connectivity, `currentTurnSessionId` |
| Message | Client → server `move` `{ side, row, col }` via game store `sendMove` |
| Say (ephemeral) | `say` `{ presetId }` → `broadcast('say', { sessionId, presetId, at })` — see `work-with-messages`; **not** schema |
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
- [x] Product sync: `started` + `seats` (+ connectivity) + `currentTurnSessionId`; seating in `onJoin` / leave/reconnect hooks (no `maxClients=4`).
- [x] Unexpected drop grace 30 s + `allowReconnection`; consented leave immediate; empty seated → dispose; mocha SC-PIECE-07…16.
- [x] `turnOrder` + `onMessage('move')` + `src/game/touristMove.ts`; mocha SC-MOVE-*.

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Keep seating pools and start-cell geometry in Room (or a pure helper), not in schema files.
- Keep reconnect grace only on tourist seated players — not LobbyRoom.
- Keep `turnOrder` room-private; sync only `currentTurnSessionId`.
- Reject illegal moves server-side without mutating state.
- Prefer pure rules modules + thin Room glue.

## Don't

- Cap the room with `maxClients = 4` — spectators are allowed; seated ≤ 4 via `seats`/`started`.
- Treat unexpected drop as immediate seat delete without grace, or advance turn on drop.
- Let spectators hold a room with zero seated players.
- Gate moves on `started`.
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
