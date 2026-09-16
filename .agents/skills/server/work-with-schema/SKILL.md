---
name: work-with-schema
description: >-
  Use when creating, changing, reviewing, or debugging @colyseus/schema sync
  state in the happy-tourist Colyseus server — MyRoomState phase/maxSeats/
  countdownRemaining/started/seats (touristId + pieces + finished + connected/
  reconnectUntil/ready/finishPlace/timeExpired) + currentTurnSessionId +
  turnUntil + turnBudgetSeconds + nextFinishPlace,
  MapSchema / ArraySchema, schema() + t.* (v5), or aligning the sync surface
  with the sibling client tourist contract.
---

# Work With Schema

Use this skill when editing **synced room state** (`@colyseus/schema`) in
`happy-tourist-server`.

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are
relative to the server repo root.

Sibling client: `../happy-tourist.github.io` (room `tourist`; board geometry local;
seats/`phase`/`maxSeats`/`countdownRemaining`/`currentTurnSessionId`/`turnUntil`/
`turnBudgetSeconds`/connectivity/ready/`finishPlace`/`timeExpired` mirrored in
`stores/game`; presence dual rings + move hints + countdown overlay on GamePage).

Schema is the **sync surface only**. Seating assignment, reconnect grace, start
countdown timers, turn order (`turnOrder` room-private), turn deadlines, and move
rules live in the Room / `work-with-game` — not in schema definitions.

## Overview

| Concern | Path / pattern |
| --- | --- |
| Sync state | `src/rooms/schema/MyRoomState.ts` |
| API | `@colyseus/schema` **v5**: `schema({...})`, `t.*`, `SchemaType` |
| Room wiring | Room sets `this.setState(...)` / mutates seats + phase/countdown + `currentTurnSessionId` |
| Client consumer | `../happy-tourist.github.io` — `stores/game.ts` `onStateChange`; GamePage presence + move UX |

Package: `@colyseus/schema` `^5.0.14` (see `package.json`).

## Product Sync Surface (today)

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

export const Piece = schema(
  {
    side: t.string(), // "N"|"E"|"S"|"W"
    row: t.uint8(),
    col: t.uint8(),
    /** True after landing on a center cell; off-board for occupancy. */
    finished: t.boolean().default(false),
  },
  "Piece",
);

/** Seated player: unique kind; pieces empty until playing (materialize on enterPlaying). */
export const Seat = schema(
  {
    touristId: t.uint8(), // 1…4
    pieces: t.map(Piece),
    connected: t.boolean().default(true),
    reconnectUntil: t.number().default(0),
    ready: t.boolean().default(false),
    /** Finish place; `0` until all four pieces finished. */
    finishPlace: t.uint8().default(0),
    /** Solo turn budget elapsed; moves rejected; seat stays. */
    timeExpired: t.boolean().default(false),
  },
  "Seat",
);

export const MyRoomState = schema(
  {
    /** Legacy mirror of phase === "playing" (prefer reading phase). */
    started: t.boolean().default(false),
    phase: t.string().default("waiting"), // waiting | countdown | playing
    maxSeats: t.uint8().default(2), // 2|3|4
    countdownRemaining: t.uint8().default(0), // 5…1 during countdown
    seats: t.map(Seat), // key = sessionId
    currentTurnSessionId: t.string().default(""),
    /** Next finish place to assign (starts at 1; increments on full finish). */
    nextFinishPlace: t.uint8().default(1),
    /** Unix ms turn deadline; `0` = no active turn timer. */
    turnUntil: t.number().default(0),
    /** Active turn budget seconds (60 multi / 300 solo); `0` when none. */
    turnBudgetSeconds: t.uint16().default(0),
  },
  "MyRoomState",
);
```

### Start / capacity fields (game/start)

| Field | Meaning |
|-------|---------|
| `phase` | `waiting` \| `countdown` \| `playing` — **primary** start gate |
| `maxSeats` | Table capacity from create options (2\|3\|4) |
| `countdownRemaining` | Authoritative second 5…1 while countdown; else 0 |
| `started` | Legacy mirror of `phase === 'playing'`; client prefers `phase` |
| `Seat.ready` | One-shot ready-to-start while underfilled waiting |

### Connectivity fields (D2 / SC-PIECE-16)

| Field | Meaning |
|-------|---------|
| `connected` | `true` online; `false` during unexpected-disconnect grace |
| `reconnectUntil` | Unix ms deadline while offline; **`0` when online / no grace** |

Room mutates these in `onDrop` / `onReconnect` / seat assign — see `work-with-rooms` / `work-with-game`. Client mirrors into Pinia `GameSeat` and drives presence dual rings from `turnUntil` / `reconnectUntil`.

### Turn fields (D1 / game/move + turn timer)

| Field | Meaning |
|-------|---------|
| `currentTurnSessionId` | Synced whose turn; empty when no eligible seat |
| `turnUntil` | Unix ms deadline; `0` = no active turn timer |
| `turnBudgetSeconds` | `60` multi / `300` solo while timer active; else `0` |
| `Seat.timeExpired` | Solo budget elapsed; moves rejected; seat remains |

**Not synced:** room-private `turnOrder: string[]` in `MyRoom` (join-order queue). Client only needs «чей ход» + deadline chrome.

### Finish fields (game/finish)

| Field | Meaning |
|-------|---------|
| `Piece.finished` | `true` after legal move onto a center cell; ignored for occupancy / board render |
| `Seat.finishPlace` | `0` until all four pieces finished; then `1…n` (monotonic room order) |
| `nextFinishPlace` | Next place to assign (starts `1`; increment after each full finish) |

Finished seats remain in `seats` (count toward `maxSeats`) until consented leave / grace timeout. No room phase `finished`.

**Not synced (ephemeral messages):** preset `say` bubbles — room-private `liveSays` + `broadcast('say', …)`; client keeps `sayEvents` in Pinia. Do **not** add bubble fields to schema.

- Client mirrors `phase` / `maxSeats` / `countdownRemaining` + seats (+ `ready` / `finishPlace` / `timeExpired` / piece `finished`) + `currentTurnSessionId` / `turnUntil` / `turnBudgetSeconds` into Pinia; GamePage draws unfinished pieces (+ short disappear), strip only once pieces exist, place/timeout modals, leave confirm only when seated ∧ playing ∧ `finishPlace === 0` ∧ `!timeExpired`, and say bubbles from `sayEvents`.
- Board **tile geometry** stays a client CSS Grid constant — not in schema.
- Do **not** revive draughts `board` / cell `0`–`4` / `{ from, to }` encoding.

## TypeScript Pattern

Always use **declarative** `schema()` + `t.*` (not `@type` decorators):

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

export const MyRoomState = schema({
  // fields...
}, "MyRoomState");
export type MyRoomState = SchemaType<typeof MyRoomState>;
```

Rules:

- `export const X = schema({...}[, "X"])` then `export type X = SchemaType<typeof X>`.
- Named schema string (`"MyRoomState"`) aids reflection / tooling.
- Collections take the **child type name or Schema class**, never a nested builder:
  - ✅ `t.array("uint8")`, `t.array(Player)`, `t.map(Seat)`
  - ❌ `t.array(t.uint8())`, `t.map(t.string())`

## Collections

### Maps — `t.map`

Key = Colyseus `client.sessionId`. Value = small child schema (e.g. `Seat`).

```ts
this.state.seats.set(client.sessionId, new Seat({ /* … */ }));
this.state.seats.delete(sessionId);
```

### Arrays — `t.array`

Prefer a flat `ArraySchema` unless the client already consumes a nested shape in
lockstep. Initialize collections in Room `onCreate` (not inside schema “logic”).

## Do

- Keep schema fields to **what must sync** (today: `phase` / `maxSeats` / `countdownRemaining` + `seats` incl. connectivity/`ready`/`finishPlace` + piece `finished` + `currentTurnSessionId` + `nextFinishPlace`; legacy `started` OK).
- Mutate state from the **Room** after validating actions; treat client payloads as intents only.
- Align field names with `../happy-tourist.github.io` (`stores/game`) in the same change.
- Use `schema()` + `t.*` + `export type X = SchemaType<typeof X>`.
- Update tests / loadtest when the state shape or room name changes.

## Don't

- Put rules, seating pools, `turnOrder`, countdown timers, reconnect timers, say/`liveSays`, win detection, or rating DB writes inside schema files.
- Trust or echo a client-supplied full board/layout as truth.
- Invent parallel field names without changing the client in the same effort.
- Mix decorator `@type` Schema classes with the v5 `schema()` style in this package.
- Pass `t.*()` builders as collection element types (`t.array(t.uint8())`).
- Expand sync state with secrets, passwords, or JWT material.
- Reintroduce draughts cell encoding / `{ from, to }` wire as “intended” without a product decision.
- Sync selection / move hints (client-local only).

## Alignment Checklist

When changing schema:

1. Field names match client Pinia / GamePage (`phase`, `maxSeats`, `countdownRemaining`, `seats` → `touristId` + `pieces` (+ `finished`) + connectivity + `ready` + `finishPlace`, `currentTurnSessionId`; client need not mirror `nextFinishPlace`).
2. Room owns seating, reconnect, start countdown, turn order, and messages — schema does not define messages or `turnOrder`.
3. Sibling client `onStateChange` / presence / `isPlaying` / `isMyTurn` is updated together when fields appear.
4. Prefer server → client alignment over unilateral client rewrites.

## Related

- Room structure / layering: `server-work-with-structure`
- Where to edit for a task: `server-locate-change-points`
- Seating / reconnect / turn / move: `work-with-game`
- Client board + presence UI: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`
- Client room state mapping: `happy-tourist-meta/.agents/skills/client/work-with-rooms/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md` (Current vs client contract)
