---
name: work-with-schema
description: >-
  Use when creating, changing, reviewing, or debugging @colyseus/schema sync
  state in the happy-tourist Colyseus server — MyRoomState scaffold, MapSchema /
  ArraySchema, schema() + t.* (v5), or aligning the sync surface with the sibling
  client tourist contract (product fields deferred until rules land).
---

# Work With Schema

Use this skill when editing **synced room state** (`@colyseus/schema`) in
`happy-tourist-server`.

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are
relative to the server repo root.

Sibling client: `../happy-tourist.github.io` (room `tourist`; static Game board today;
synced rules fields deferred until product rules land).

Schema is the **sync surface only**. Authoritative board-game rules and
validation live in the Room / `work-with-game` — not in schema definitions.

## Overview

| Concern | Path / pattern |
| --- | --- |
| Sync state | `src/rooms/schema/MyRoomState.ts` |
| API | `@colyseus/schema` **v5**: `schema({...})`, `t.*`, `SchemaType` |
| Room wiring | Room sets `this.setState(...)` / mutates state fields |
| Client consumer | `../happy-tourist.github.io` — `stores/game.ts` `onStateChange` |

Package: `@colyseus/schema` `^5.0.14` (see `package.json`).

## Current Vs Intended

**Today** (`MyRoomState.ts`) — scaffold only:

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

export const MyRoomState = schema({
  mySynchronizedProperty: t.string().default("Hello world"),
});
export type MyRoomState = SchemaType<typeof MyRoomState>;
```

**When rules land** — replace scaffold fields with the product sync surface agreed
with the client in the **same** change. Do **not** treat legacy 8×8 draughts
`board` / `currentTurn` / cell `0`–`4` / `move` as current product canon unless
the product explicitly revives that contract.

Client today maps optional `status` only; board geometry is a client CSS Grid
constant (`work-with-game-board`), not schema.

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
  - ✅ `t.array("uint8")`, `t.array(Player)`, `t.map(Player)`
  - ❌ `t.array(t.uint8())`, `t.map(t.string())`

## Collections (when product needs them)

### Maps — `t.map`

Key = Colyseus `client.sessionId` (or another stable id). Value = small child schema.

```ts
this.state.players.set(client.sessionId, new Player({ /* … */ }));
// leave / dispose: this.state.players.delete(sessionId)
```

### Arrays — `t.array`

Prefer a flat `ArraySchema` unless the client already consumes a nested shape in
lockstep. Initialize collections in Room `onCreate` (not inside schema “logic”).

## Do

- Keep schema fields to **what must sync** to clients once product decides.
- Mutate state from the **Room** after validating actions; treat client payloads as intents only.
- Align field names with `../happy-tourist.github.io` (`stores/game`) in the same change.
- Use `schema()` + `t.*` + `export type X = SchemaType<typeof X>`.
- Update tests / loadtest when the state shape or room name changes.

## Don't

- Put rules, win detection, or rating DB writes inside schema files.
- Trust or echo a client-supplied full board/layout as truth.
- Invent parallel field names without changing the client in the same effort.
- Mix decorator `@type` Schema classes with the v5 `schema()` style in this package.
- Pass `t.*()` builders as collection element types (`t.array(t.uint8())`).
- Expand sync state with secrets, passwords, or JWT material.
- Reintroduce draughts cell encoding / `move` wire as “intended” without a product decision.

## Alignment Checklist

When changing schema:

1. Field names match **current** client expectations (today: mostly scaffold + optional `status`).
2. Room owns message handling — schema does not define messages.
3. Sibling client `onStateChange` is updated together when fields appear.
4. Prefer server → client alignment over unilateral client rewrites.

## Related

- Room structure / layering: `server-work-with-structure`
- Where to edit for a task: `server-locate-change-points`
- Rules (later): `work-with-game`
- Client board UI: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`
- Client room state mapping: `happy-tourist-meta/.agents/skills/client/work-with-rooms/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md` (Current vs client contract)
