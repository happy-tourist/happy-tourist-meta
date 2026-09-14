---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating, reconnect grace, and (later) board-game rules in the
  happy-tourist Colyseus server — Room handlers, schema seats/started/
  connectivity, or pure rules modules (moves deferred).
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating truth and reconnect grace today and will own move rules later. Client board geometry is local CSS Grid; pieces/strip/presence mirror synced seats. Do not trust client-local seat assignment. Do not invent legacy draughts rules.

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; seat assign; `onDrop`/`onReconnect`/`onLeave`; future `onMessage` for moves |
| Schema | `src/rooms/schema/MyRoomState.ts` | `started` + `seats` Map → `touristId` + `pieces` + `connected` / `reconnectUntil` |
| Rules (preferred) | new pure module e.g. `src/game/` or `src/rooms/tourist/` | Validate / apply moves later without Colyseus I/O |
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

## Authority

- Server state is the only seating (and future rules) truth. Reject illegal actions; leave state unchanged.
- Client tourist layout is **UI geometry** — not an authority and not a rules engine.
- Future wire protocol for turns/actions MUST be agreed with the client in lockstep. Do not reintroduce legacy 8×8 draughts `move` `{ from, to }` / cell encoding unless product explicitly revives that contract.
- Do not put rules in Express HTTP routes.

## Client Contract (align server to this)

| Field / message | Client expectation |
| --- | --- |
| Room name | `tourist` |
| Board UI | Client-only `LAYOUT` in `GamePage`; server does **not** sync tile kinds |
| Synced state | `started`, `seats` Map keyed by `sessionId` → `touristId` + four `pieces` + `connected` / `reconnectUntil` |
| Reconnect | Colyseus token; client `sessionStorage` + `reconnect` then `joinById` |
| Presence | Client-only chrome from mirrored connectivity |
| Game messages | None for seating; add with client lockstep when moves land |

Client-local only (do **not** put on schema): Pinia `idle` / `connecting` status strings; presence layout. Lobby leave-before-enter / quiet listing stays in `work-with-rooms` / client `work-with-lobby`.

## Rules (later)

When implementing moves:

1. Prefer **pure functions** (testable without Colyseus) separate from Room I/O.
2. Room: auth, seats, reconnect timeouts, applying results, broadcasting via schema.
3. Do not duplicate a second authoritative engine on the client.
4. Document product-specific choices (turn order, tile flips, win conditions) as open questions until fixed in code + specs.

## Architecture Preference

```
onMessage(<action>) → parse payload → pure validate/apply(state, action, seat)
                     → if ok: write schema fields
```

## Implementation Checklist

- [x] Register room as `tourist` (+ `.enableRealtimeListing()`); tests/loadtest use `tourist`.
- [x] Product sync: `started` + `seats` (+ connectivity); seating in `onJoin` / leave/reconnect hooks (no `maxClients=4`).
- [x] Unexpected drop grace 30 s + `allowReconnection`; consented leave immediate; empty seated → dispose; mocha SC-PIECE-07…16.
- [ ] Wire room messages to pure move rules; unit-test pure rules.

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Keep seating pools and start-cell geometry in Room (or a pure helper), not in schema files.
- Keep reconnect grace only on tourist seated players — not LobbyRoom.
- Reject illegal actions server-side once move rules exist.
- Prefer pure rules modules + thin Room glue.
- Label undecided product choices as open questions.

## Don't

- Cap the room with `maxClients = 4` — spectators are allowed; seated ≤ 4 via `seats`/`started`.
- Treat unexpected drop as immediate seat delete without grace.
- Let spectators hold a room with zero seated players.
- Treat draughts cell encoding or move UX as the product canon.
- Trust client layout constants or local UI as game authority.
- Invent unsupported product rules and present them as hard fact.
- Put authoritative logic only in Express routes.
- Change room name or message shapes without updating the client in lockstep.

## Related

- Client board + presence: `.agents/skills/client/work-with-game-board/SKILL.md`
- Schema seats + connectivity: `.agents/skills/server/work-with-schema/SKILL.md`
- Rooms / registration / LobbyRoom policy: `.agents/skills/server/work-with-rooms/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
