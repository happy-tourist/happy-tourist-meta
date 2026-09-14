---
name: work-with-game
description: >-
  Use when implementing, changing, reviewing, or debugging authoritative board-game
  rules for Счастливый турист in the happy-tourist Colyseus server — Room handlers,
  schema state for the tourist room, or pure rules modules (rules come later).
---

# Work With Game

Use this skill for **authoritative board-game rules** («Счастливый турист») in `happy-tourist-server`.

Server owns truth once rules exist; the client currently shows a **static** tourist board (no move UX). Do not trust client-local board or interaction state. Product ruleset is **deferred** — mark undecided choices as open questions; do not invent legacy draughts rules.

Pair with client board UX: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md` (static layout today; this package remains source of truth for future rules).

## Overview

| Surface | Path | Role |
| --- | --- | --- |
| Room | `src/rooms/MyRoom.ts` | Lifecycle; future `onMessage` for game actions; apply validated results to state |
| Schema | `src/rooms/schema/MyRoomState.ts` | Synced state (scaffold today; extend when rules land) |
| Rules (preferred) | new pure module e.g. `src/game/` or `src/rooms/tourist/` | Validate / apply without Colyseus I/O |
| Registration | `src/app.config.ts` | Room name must be `tourist` for client lobby |

Scaffold today: room/schema are stubs (`tourist` registered, `MyRoomState` still `mySynchronizedProperty`). Guide implementation toward the client contract below; do not treat unimplemented behavior as shipped product fact.

## Authority

- Server state is the only game truth once rules exist. Reject illegal actions; leave state unchanged.
- Client tourist board is **static UI** (CSS Grid layout) — not an authority and not a rules engine.
- Future wire protocol for turns/actions MUST be agreed with the client in lockstep. Do not reintroduce legacy 8×8 draughts `move` `{ from, to }` / cell encoding unless product explicitly revives that contract.
- Do not put rules in Express HTTP routes.

## Client Contract (align server to this)

| Field / message | Client expectation today |
| --- | --- |
| Room name | `tourist` |
| Board UI | Client-only static layout (`GamePage`); server does **not** sync tile geometry in this phase |
| Synced state | Scaffold; fill when rules land (prefer shared names with client Pinia once defined) |
| Game messages | None required for static board; add with client lockstep when rules land |

Client-local only (do **not** put on schema): Pinia `idle` / `connecting` status strings if reintroduced. Lobby leave-before-enter and room join stay in `work-with-rooms` / client `work-with-lobby`.

## Rules (later)

When implementing rules:

1. Prefer **pure functions** (testable without Colyseus) separate from Room I/O.
2. Room: auth, seats, timeouts, applying results, broadcasting via schema.
3. Do not duplicate a second authoritative engine on the client.
4. Document product-specific choices (player count, turn order, tile flips, win conditions) as open questions until fixed in code + specs.

## Architecture Preference

```
onMessage(<action>) → parse payload → pure validate/apply(state, action, seat)
                     → if ok: write schema fields
```

## Implementation Checklist (scaffold → product)

- [x] Register room as `tourist` (+ `.enableRealtimeListing()`); tests/loadtest use `tourist`.
- [ ] Replace scaffold schema with product synced fields when rules land.
- [ ] Wire room messages to pure rules; unit-test pure rules.
- [ ] Confirm seating / disconnect / forfeit policy with client when product decides.

## Do

- Keep room name `tourist` aligned with client `TOURIST_ROOM`.
- Reject illegal actions server-side once rules exist.
- Prefer pure rules modules + thin Room glue.
- Label undecided product choices as open questions.

## Don't

- Reintroduce board-game rules (later) / tourist cell encoding or move UX as the product canon.
- Trust client layout constants or local UI as game authority.
- Invent unsupported product rules and present them as hard fact.
- Put authoritative logic only in Express routes.
- Change room name or message shapes without updating the client in lockstep.

## Related

- Client board skill: `.agents/skills/client/work-with-game-board/SKILL.md`
- Rooms / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Change-point map: `.agents/skills/server/server-locate-change-points/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
