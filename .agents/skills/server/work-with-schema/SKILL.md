---
name: work-with-schema
description: >-
  Use when creating, changing, reviewing, or debugging @colyseus/schema sync
  state in the happy-tourist Colyseus server — MyRoomState, board / currentTurn /
  status / players fields, MapSchema / ArraySchema, schema() + t.* (v5), or
  aligning the sync surface with the sibling client checkers contract.
---

# Work With Schema

Use this skill when editing **synced room state** (`@colyseus/schema`) in
`happy-tourist-server`.

**Temp skills path:** this skill lives under `.agents/skills/server/` in this
package for now. Canonical skills are intended to live in
`happy-tourist-meta/.agents/skills/server/` once meta is available — prefer that
path when choosing skills if it exists.

Sibling client: `../happy-tourist.github.io` (expects `board`, `currentTurn`,
`status`, `players[sessionId].color`; cell values `0`–`4`).

Schema is the **sync surface only**. Authoritative checkers rules, move
validation, and side effects live in the Room / checkers logic — not in schema
definitions.

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

**Intended** (align with client; replace scaffold fields):

| Field | Role | Client mapping |
| --- | --- | --- |
| `board` | Cell values for 8×8 | `game.board` (`CellValue[][]`) |
| `currentTurn` | `'white' \| 'black'` | `game.currentTurn` |
| `status` | `'waiting' \| 'playing' \| 'finished'` | `game.status` (room status, not lobby listing alone) |
| `players` | Map keyed by `sessionId` | `players[sessionId].color` → `game.myColor` |

### Cell values

| Value | Meaning |
| --- | --- |
| `0` | empty |
| `1` | white |
| `2` | black |
| `3` | white king |
| `4` | black king |

Keep encoding `0`–`4` stable; change only in lockstep with the client.

## TypeScript Pattern

Always use **declarative** `schema()` + `t.*` (not `@type` decorators):

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

export const Player = schema({
  color: t.string(), // 'white' | 'black'
}, "Player");
export type Player = SchemaType<typeof Player>;

export const MyRoomState = schema({
  // fields...
}, "MyRoomState");
export type MyRoomState = SchemaType<typeof MyRoomState>;
```

Rules:

- `export const X = schema({...}[, "X"])` then `export type X = SchemaType<typeof X>`.
- Named schema string (`"Player"`, `"MyRoomState"`) aids reflection / tooling.
- Collections take the **child type name or Schema class**, never a nested builder:
  - ✅ `t.array("uint8")`, `t.array(Player)`, `t.map(Player)`
  - ❌ `t.array(t.uint8())`, `t.map(t.string())`

## Extending Collections (Board / Players)

### Players — `MapSchema` via `t.map`

Key = Colyseus `client.sessionId`. Value = small player schema with at least
`color`.

```ts
export const MyRoomState = schema({
  players: t.map(Player),
  // ...
}, "MyRoomState");
```

Room mutates with map API (not plain-object assign as source of truth):

```ts
this.state.players.set(client.sessionId, new Player({ color: "white" }));
// leave / dispose: this.state.players.delete(sessionId)
```

Client reads `state.players?.[room.sessionId]?.color`.

### Board — `ArraySchema` via `t.array`

Prefer a **flat** `ArraySchema` of 64 `uint8` cells (`index = row * 8 + col`)
unless the client already consumes a nested shape in lockstep:

```ts
export const MyRoomState = schema({
  board: t.array("uint8"), // length 64; values 0–4
  currentTurn: t.string(), // 'white' | 'black'
  status: t.string(),      // 'waiting' | 'playing' | 'finished'
  players: t.map(Player),
}, "MyRoomState");
```

Initialize in Room `onCreate` (not inside schema “logic”):

```ts
for (let i = 0; i < 64; i++) this.state.board.push(0);
// or assign starting russian-checkers setup cell-by-cell
```

If using nested rows (`ArraySchema` of row schemas / arrays), document the
wire shape and update the client `onStateChange` mapper in the same change.

### Intended sketch (replace scaffold)

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

export const Player = schema({
  color: t.string(),
}, "Player");
export type Player = SchemaType<typeof Player>;

export const MyRoomState = schema({
  board: t.array("uint8"),
  currentTurn: t.string(),
  status: t.string().default("waiting"),
  players: t.map(Player),
}, "MyRoomState");
export type MyRoomState = SchemaType<typeof MyRoomState>;
```

Remove `mySynchronizedProperty` when adopting the checkers shape.

## Do

- Keep schema fields to **what must sync** to clients (`board`, turn, status, seats).
- Mutate state from the **Room** after validating moves; treat client payloads as intents only.
- Align field names and cell encoding with `../happy-tourist.github.io` (`stores/game`, board skill).
- Use `schema()` + `t.*` + `export type X = SchemaType<typeof X>`.
- Use `t.map` for `players` (sessionId keys) and `t.array` for `board` cells.
- Default collection fields via Room init or `.default(...)` factories where needed — avoid shared mutable defaults across instances.
- Update tests / loadtest when the state shape or room name changes.

## Don't

- Put rules, capture chains, win detection, or rating DB writes inside schema files.
- Trust or echo a client-supplied full board as truth.
- Invent parallel field names (`cells`, `turn`, `gameStatus`) without changing the client in the same effort.
- Mix decorator `@type` Schema classes with the v5 `schema()` style in this package.
- Pass `t.*()` builders as collection element types (`t.array(t.uint8())`).
- Expand sync state with secrets, passwords, or JWT material.

## Alignment Checklist

When changing schema:

1. Field names match client expectations: `board`, `currentTurn`, `status`, `players[].color`.
2. Cell values stay `0`–`4` as above.
3. Room still owns `move` handling (`{ from, to }` with `{ row, col }`) — schema does not define messages.
4. Sibling client `onStateChange` still maps correctly (or is updated together).
5. Prefer server → client alignment over unilateral client rewrites.

## Related

- Room structure / layering: `server-work-with-structure`
- Where to edit for a task: `server-locate-change-points`
- Client board / move UX: `../happy-tourist.github.io/.agents/skills/client/work-with-game-board`
- Client room state mapping: `../happy-tourist.github.io/.agents/skills/client/work-with-rooms`
- Package overview: `AGENTS.md` (Current vs client contract)
