---
name: work-with-schema
description: >-
  Use when creating, changing, reviewing, or debugging @colyseus/schema sync
  state in the happy-tourist Colyseus server — MyRoomState started/seats,
  MapSchema / ArraySchema, schema() + t.* (v5), or aligning the sync surface
  with the sibling client tourist contract (move rules deferred).
---

# Work With Schema

Use this skill when editing **synced room state** (`@colyseus/schema`) in
`happy-tourist-server`.

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are
relative to the server repo root.

Sibling client: `../happy-tourist.github.io` (room `tourist`; board geometry local;
seats/`started` mirrored in `stores/game`).

Schema is the **sync surface only**. Seating assignment and future rules live in
the Room / `work-with-game` — not in schema definitions.

## Overview

| Concern | Path / pattern |
| --- | --- |
| Sync state | `src/rooms/schema/MyRoomState.ts` |
| API | `@colyseus/schema` **v5**: `schema({...})`, `t.*`, `SchemaType` |
| Room wiring | Room sets `this.setState(...)` / mutates seats in `onJoin` / `onLeave` |
| Client consumer | `../happy-tourist.github.io` — `stores/game.ts` `onStateChange` |

Package: `@colyseus/schema` `^5.0.14` (see `package.json`).

## Product Sync Surface (today)

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

export const Seat = schema(
  {
    touristId: t.uint8(), // 1…4
    side: t.string(),     // "N"|"E"|"S"|"W"
    row: t.uint8(),
    col: t.uint8(),
  },
  "Seat",
);

export const MyRoomState = schema(
  {
    started: t.boolean().default(false),
    seats: t.map(Seat), // key = sessionId
  },
  "MyRoomState",
);
```

- Client mirrors `started` + seats into Pinia; GamePage draws pieces / strip.
- Board **tile geometry** stays a client CSS Grid constant — not in schema.
- Move / turn fields — later; do **not** revive draughts `board` / `currentTurn` / cell `0`–`4` / `move` unless product revives that contract.

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

- Keep schema fields to **what must sync** (today: `started` + `seats`).
- Mutate state from the **Room** after validating actions; treat client payloads as intents only.
- Align field names with `../happy-tourist.github.io` (`stores/game`) in the same change.
- Use `schema()` + `t.*` + `export type X = SchemaType<typeof X>`.
- Update tests / loadtest when the state shape or room name changes.

## Don't

- Put rules, seating pools, win detection, or rating DB writes inside schema files.
- Trust or echo a client-supplied full board/layout as truth.
- Invent parallel field names without changing the client in the same effort.
- Mix decorator `@type` Schema classes with the v5 `schema()` style in this package.
- Pass `t.*()` builders as collection element types (`t.array(t.uint8())`).
- Expand sync state with secrets, passwords, or JWT material.
- Reintroduce draughts cell encoding / `move` wire as “intended” without a product decision.

## Alignment Checklist

When changing schema:

1. Field names match client Pinia / GamePage (`started`, `seats` → `touristId`/`side`/`row`/`col`).
2. Room owns seating and messages — schema does not define messages.
3. Sibling client `onStateChange` is updated together when fields appear.
4. Prefer server → client alignment over unilateral client rewrites.

## Related

- Room structure / layering: `server-work-with-structure`
- Where to edit for a task: `server-locate-change-points`
- Seating / future rules: `work-with-game`
- Client board UI: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`
- Client room state mapping: `happy-tourist-meta/.agents/skills/client/work-with-rooms/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md` (Current vs client contract)
