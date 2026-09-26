---
name: work-with-schema
description: >-
  Use when creating, changing, reviewing, or debugging @colyseus/schema sync
  state in the happy-tourist Colyseus server — MyRoomState phase/maxSeats/
  grid/touristsPerPlayer/packTitle/flippedCells/answerCards/peek session +
  countdownRemaining/started/seats (touristId + pieces by pieceId +
  finished/trapped + connected/reconnectUntil/ready/finishPlace/timeExpired) +
  currentTurnSessionId + turnUntil + turnBudgetSeconds + nextFinishPlace +
  removedTaskKeys + holdingGrilleKeys + revealingCatapultKeys + brokenCatapultKeys,
  MapSchema / ArraySchema, schema() + t.* (v5), or aligning the sync surface with
  the sibling client tourist contract.
---

# Work With Schema

Use this skill when editing **synced room state** (`@colyseus/schema`) in
`happy-tourist-server`.

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are
relative to the server repo root.

Sibling client: `../happy-tourist.github.io` (room `tourist`; board geometry from
synced `grid` via `lib/boardGeometry`; seats/`phase`/`maxSeats`/`grid`/
`touristsPerPlayer`/peek session/`countdownRemaining`/`currentTurnSessionId`/
`turnUntil`/`turnBudgetSeconds`/`removedTaskKeys`/`holdingGrilleKeys`/
`revealingCatapultKeys`/`brokenCatapultKeys`/connectivity/ready/`finishPlace`/
`timeExpired`/piece `trapped` mirrored in `stores/game`).

Schema is the **sync surface only**. Seating assignment, reconnect grace, start
countdown timers, turn order (`turnOrder` room-private), private step/peek budgets,
task deck / cell bindings, hidden grille + hidden catapult seed, turn deadlines, and
move/peek/grille/catapult rules live in the Room / `work-with-game` — not in schema
definitions.

## Overview

| Concern | Path / pattern |
| --- | --- |
| Sync state | `src/rooms/schema/MyRoomState.ts` |
| API | `@colyseus/schema` **v5**: `schema({...})`, `t.*`, `SchemaType` |
| Room wiring | Room sets `this.setState(...)` / mutates seats + phase/countdown + `currentTurnSessionId` + content snapshot fields |
| Client consumer | `../happy-tourist.github.io` — `stores/game.ts` `onStateChange`; GamePage presence + move UX |

Package: `@colyseus/schema` `^5.0.14` (see `package.json`).

## Product Sync Surface (today)

```ts
import { schema, t, type SchemaType } from "@colyseus/schema";

/**
 * One tourist token. Map key = seat-local piece id `"0"`…`"touristsPerPlayer-1"`.
 * No board side identity (N/E/S/W).
 */
export const Piece = schema(
  {
    row: t.uint8(),
    col: t.uint8(),
    finished: t.boolean().default(false),
    trapped: t.boolean().default(false),
  },
  "Piece",
);

export const Seat = schema(
  {
    touristId: t.uint8(), // 1…4
    pieces: t.map(Piece), // key = pieceId
    connected: t.boolean().default(true),
    reconnectUntil: t.number().default(0),
    ready: t.boolean().default(false),
    /** Finish place; `0` until all own pieces finished. */
    finishPlace: t.uint8().default(0),
    timeExpired: t.boolean().default(false),
  },
  "Seat",
);

export const FlippedCell = schema(
  {
    taskId: t.string(),
    difficulty: t.uint8(), // 1|2|3
  },
  "FlippedCell",
);

export const AnswerCardSync = schema(
  {
    id: t.string(),
    content: t.string(),
    description: t.string(),
  },
  "AnswerCardSync",
);

export const MyRoomState = schema(
  {
    started: t.boolean().default(false),
    phase: t.string().default("waiting"),
    maxSeats: t.uint8().default(2), // chosen 1…map.players at create (omit → min(2, map.players))
    countdownRemaining: t.uint8().default(0),
    seats: t.map(Seat),
    currentTurnSessionId: t.string().default(""),
    nextFinishPlace: t.uint8().default(1),
    turnUntil: t.number().default(0),
    turnBudgetSeconds: t.uint16().default(0),
    removedTaskKeys: t.array("string"),
    holdingGrilleKeys: t.array("string"),
    revealingCatapultKeys: t.array("string"),
    brokenCatapultKeys: t.array("string"),
    /** Map snapshot grid (100 chars) — authoritative board layout. */
    grid: t.string().default(""),
    touristsPerPlayer: t.uint8().default(4),
    packTitle: t.string().default(""),
    flippedCells: t.map(FlippedCell), // key = "r,c"
    answerCards: t.array(AnswerCardSync),
    peekActive: t.boolean().default(false),
    peekSessionId: t.string().default(""),
    peekPieceId: t.string().default(""),
    peekRow: t.uint8().default(0),
    peekCol: t.uint8().default(0),
    peekTaskId: t.string().default(""),
    peekQuestion: t.string().default(""),
    peekDifficulty: t.uint8().default(0),
    peekPlacements: t.array("string"), // ordered answerCard ids / empty
  },
  "MyRoomState",
);
```

### Start / capacity fields (game/start)

| Field | Meaning |
|-------|---------|
| `phase` | `waiting` \| `countdown` \| `playing` — **primary** start gate |
| `maxSeats` | Table capacity chosen at create (`1…map.players`; omit → `min(2, map.players)`) |
| `grid` / `touristsPerPlayer` / `packTitle` | Immutable create snapshot (map + pack chrome) |
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
| `Seat.timeExpired` | Solo budget elapsed; moves/peeks rejected; seat remains |

**Not synced:** room-private `turnOrder: string[]` in `MyRoom` (join-order queue). Client only needs «чей ход» + deadline chrome.

### Finish fields (game/finish)

| Field | Meaning |
|-------|---------|
| `Piece.finished` | `true` after legal move onto a center cell; ignored for occupancy / board render |
| `Seat.finishPlace` | `0` until all **own** pieces finished; then `1…n` (monotonic room order) |
| `nextFinishPlace` | Next place to assign (starts `1`; increment after each full finish) |

Finished seats remain in `seats` (count toward `maxSeats`) until consented leave / grace timeout. No room phase `finished`.

### Removed task tiles (game/board)

| Field | Meaning |
|-------|---------|
| `removedTaskKeys` | `ArraySchema<string>` of `"r,c"` after **correct** peek only; all clients see holes; **not landable** (stand OK). Incorrect KEEP does not sync a removal |

### Grille traps (game/board + game/pieces)

| Field | Meaning |
|-------|---------|
| `Piece.trapped` | `true` after landing on unspent grille; blocks move/peek for that piece; still occupies cell |
| `holdingGrilleKeys` | `ArraySchema<string>` of revealed holding `"r,c"`; drop/rise overlay on client; cleared on rescue / all-jail / permanent leave of that seat’s trapped cells (SC-PIECE-28; onDrop grace does not) |

### Catapult reveal (game/board + game/move)

| Field | Meaning |
|-------|---------|
| `revealingCatapultKeys` | Short-lived `"r,c"` while fade anim runs (~1000 ms); hidden unspent never sync |
| `brokenCatapultKeys` | Subset of revealing keys that show broken art on fade-out (no fling dest) |

**Not synced (room-private + messages):** step/peek budgets (`budgets` Map + `client.send('budgets')`); task deck / cell bindings / content snapshot (`roomContentSnapshot`); create-time `grilleDensity` / `catapultDensity` + hidden grille/catapult keys; private `allJailWarning`. Shared peek **session fields** sync on schema (above). Do **not** put steps/peeks or hidden trap locations on schema.

**Not synced (ephemeral messages):** preset `say` bubbles — room-private `liveSays` + `broadcast('say', …)`; client keeps `sayEvents` in Pinia. Do **not** add bubble fields to schema.

- Client mirrors `phase` / `maxSeats` / `grid` / `touristsPerPlayer` / peek session / `countdownRemaining` + seats (+ pieces by **pieceId**) + turn + `removedTaskKeys` / grilles/catapults into Pinia; GamePage draws from synced `grid` via `lib/boardGeometry`.
- Board **tile geometry** is derived from synced `grid` — not a client-only LAYOUT constant.
- Do **not** revive draughts `board` / cell `0`–`4` / `{ from, to }` encoding or piece `side` N/E/S/W.

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

- Keep schema fields to **what must sync** (today: capacity/start + `grid`/`touristsPerPlayer`/`packTitle`/`flippedCells`/`answerCards`/peek session + `seats` pieces by pieceId + turn + `removedTaskKeys` + grilles/catapults; legacy `started` OK).
- Mutate state from the **Room** after validating actions; treat client payloads as intents only.
- Align field names with `../happy-tourist.github.io` (`stores/game`) in the same change.
- Use `schema()` + `t.*` + `export type X = SchemaType<typeof X>`.
- Update tests / loadtest when the state shape or room name changes.

## Don't

- Put rules, seating pools, `turnOrder`, countdown timers, reconnect timers, say/`liveSays`, private step/peek budgets, task deck/bindings, content snapshot internals, hidden grille/catapult seed/`grilleDensity`/`catapultDensity`, win detection, or rating DB writes inside schema files.
- Trust or echo a client-supplied full board/layout as truth — create loads immutable snapshot via `roomContentSnapshot`.
- Invent parallel field names without changing the client in the same effort.
- Mix decorator `@type` Schema classes with the v5 `schema()` style in this package.
- Pass `t.*()` builders as collection element types (`t.array(t.uint8())`).
- Expand sync state with secrets, passwords, or JWT material.
- Reintroduce draughts cell encoding / `{ from, to }` wire or piece `side` N/E/S/W without a product decision.
- Sync selection / move hints / steps / peeks (client-local or private messages only).

## Alignment Checklist

When changing schema:

1. Field names match client Pinia / GamePage (`phase`, `maxSeats`, `grid`, `touristsPerPlayer`, peek session, `seats` → pieces by **pieceId**, turn, `removedTaskKeys`, grilles/catapults; client need not mirror `nextFinishPlace`).
2. Room owns seating, reconnect, start countdown, turn order, budgets/peek/grilles/catapults, content snapshot, and messages — schema does not define messages or `turnOrder` / steps/peeks / hidden traps.
3. Sibling client `onStateChange` / presence / `isPlaying` / `isMyTurn` is updated together when fields appear.
4. Prefer server → client alignment over unilateral client rewrites.

## Related

- Room structure / layering: `server-work-with-structure`
- Where to edit for a task: `server-locate-change-points`
- Seating / reconnect / turn / move: `work-with-game`
- Client board + presence UI: `happy-tourist-meta/.agents/skills/client/work-with-game-board/SKILL.md`
- Client room state mapping: `happy-tourist-meta/.agents/skills/client/work-with-rooms/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md` (Current vs client contract)
