---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative
  tourist-room seating and (later) board-game rules in the happy-tourist
  Colyseus server — Room handlers, schema seats/started, or pure rules modules
  (moves deferred).
---

# Work With Game

Use this skill for **authoritative tourist-room game logic** («Счастливый турист») in `happy-tourist-server`.

Server owns seating truth today and will own move rules later. Client board geometry is local CSS Grid; pieces/strip mirror synced seats. Do not trust client-local seat assignment. Do not invent legacy draughts rules.

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`.

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | JWT `onAuth`; seat assign/remove in `onJoin`/`onLeave`; future `onMessage` for moves |
| Schema | `src/rooms/schema/MyRoomState.ts` | `started` + `seats` Map (`touristId`, `side`, `row`, `col`) |
| Rules (preferred) | new pure module e.g. `src/game/` or `src/rooms/tourist/` | Validate / apply moves later without Colyseus I/O |
| Registration | `src/app.config.ts` | Room name must be `tourist` for client lobby |

## Seating (shipped)

- Until `started`: join gets a seat — unique `touristId` 1…4 and side `N|E|S|W` from remaining pools; start cell uniform from the four starts on that side (N row0 cols3–6; E col9 rows3–6; S row9 cols3–6; W col0 rows3–6).
- Fourth seat → `started = true` (optional metadata `status: "playing"`).
- After `started`: join does **not** get a seat (spectator). No `maxClients = 4`.
- Leave before start: delete seat → kind/side back in pools. Leave after start: delete seat; `started` stays true; no new seats.
- Assign only in Room lifecycle — client never invents seats.

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
| Synced state | `started`, `seats` Map keyed by `sessionId` → `touristId`/`side`/`row`/`col` |
| Game messages | None for seating; add with client lockstep when moves land |

Client-local only (do **not** put on schema): Pinia `idle` / `connecting` status strings. Lobby leave-before-enter stays in `work-with-rooms` / client `work-with-lobby`.

## Rules (later)

When implementing moves:

1. Prefer **pure functions** (testable without Colyseus) separate from Room I/O.
2. Room: auth, seats, timeouts, applying results, broadcasting via schema.
3. Do not duplicate a second authoritative engine on the client.
4. Document product-specific choices (turn order, tile flips, win conditions) as open questions until fixed in code + specs.

## Architecture Preference

```
onMessage(<action>) → parse payload → pure validate/apply(state, action, seat)
                     → if ok: write schema fields
```

## Implementation Checklist

- [x] Register room as `tourist` (+ `.enableRealtimeListing()`); tests/loadtest use `tourist`.
- [x] Product sync: `started` + `seats`; seating in `onJoin`/`onLeave` (no `maxClients=4`).
- [ ] Wire room messages to pure move rules; unit-test pure rules.
- [ ] Confirm disconnect / forfeit / reconnect policy with client when product decides.

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Keep seating pools and start-cell geometry in Room (or a pure helper), not in schema files.
- Reject illegal actions server-side once move rules exist.
- Prefer pure rules modules + thin Room glue.
- Label undecided product choices as open questions.

## Don't

- Cap the room with `maxClients = 4` — spectators are allowed; seated ≤ 4 via `seats`/`started`.
- Treat draughts cell encoding or move UX as the product canon.
- Trust client layout constants or local UI as game authority.
- Invent unsupported product rules and present them as hard fact.
- Put authoritative logic only in Express routes.
- Change room name or message shapes without updating the client in lockstep.

## Related

- Client board skill: `.agents/skills/client/work-with-game-board/SKILL.md`
- Schema seats: `.agents/skills/server/work-with-schema/SKILL.md`
- Rooms / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
