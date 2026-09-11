---
name: work-with-rooms
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room lifecycle in
  the happy-tourist checkers client: createGame / joinGame / leaveGame,
  _enterRoom / _attachRoom / _resetRoomState, onStateChange / onError / onLeave,
  sendMove, GamePage rejoin by roomId after refresh, or CHECKERS_ROOM wiring.
---

# Work With Rooms

Use this skill for **Colyseus room lifecycle** in the happy-tourist client (`happy-tourist.github.io`).

Room connect, listeners, and leave live in **`src/stores/game.ts`**. Pages call store actions only. Do **not** put `room.onStateChange` / `onError` / `onLeave` in pages.

Coordinate schema / protocol (room name, state shape, `move` message) with [`../happy-tourist-server`](../../../../happy-tourist-server).

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Store | `src/stores/game.ts` | `CHECKERS_ROOM`, create/join/leave, `_attachRoom` listeners, `sendMove` |
| Lobby | `src/pages/LobbyPage.vue` | `createGame` / `joinGame` → navigate to `game` with `roomId` |
| Game | `src/pages/GamePage.vue` | Rejoin by `roomId` if Pinia lost room; `sendMove`; `leaveGame` |
| Boot | `src/boot/colyseus.ts` | Shared `Client` (`VITE_COLYSEUS_URL`) |
| Route | `/game/:roomId` | Hash mode; `meta.requiresAuth` |

Room type constant: `CHECKERS_ROOM = 'checkers'`.

## Lifecycle Flow

```text
createGame / joinGame(roomId?) / joinGame()
        │
        ▼
  _enterRoom(connect)
        │  status = 'connecting'; clear error
        │  await leaveGame()          ← leave before enter
        │  room = await connect()     ← create | joinById | joinOrCreate
        │  _attachRoom(room)
        ▼
  listeners (store only)
        │  onStateChange → board, currentTurn, status, myColor
        │  onError → error string
        │  onLeave → _resetRoomState()
        ▼
  leaveGame() or remote leave
        │  _resetRoomState(); room.leave() (swallow closed-room errors)
```

### Public actions → SDK

| Intent | Store action | SDK |
|--------|--------------|-----|
| New room | `createGame(options?)` | `client.create(CHECKERS_ROOM, options)` |
| Join by id | `joinGame(roomId, options?)` | `client.joinById(roomId, options)` |
| Join or create | `joinGame()` (no id) | `client.joinOrCreate(CHECKERS_ROOM, options)` |
| Leave | `leaveGame()` | `room.leave()` after `_resetRoomState` |
| Move | `sendMove(from, to)` | `room.send('move', { from, to })` |

All connect paths go through `_enterRoom`. Do not call `client.create` / `joinById` / `joinOrCreate` from pages.

## Private Actions Pattern

Three private helpers own the room session. Keep this split when extending the store.

### `_enterRoom(connect)`

1. Set `status = 'connecting'`, `error = null`.
2. **`await this.leaveGame()`** — always leave before enter (detach previous room / clear state).
3. `const room = await connect()`.
4. `this._attachRoom(room)` and return `room`.
5. On failure: `status = 'idle'`, set `error`, rethrow.

```ts
async _enterRoom(connect: () => Promise<Room>) {
  this.status = 'connecting';
  this.error = null;
  try {
    await this.leaveGame();
    const room = await connect();
    this._attachRoom(room);
    return room;
  } catch (e) {
    this.status = 'idle';
    this.error = e instanceof Error ? e.message : String(e);
    throw e;
  }
}
```

### `_attachRoom(room)`

Wire listeners **once** in the store (never in `GamePage`):

1. Assign `this.room`, `this.roomId = room.roomId`, `this.status = 'waiting'`.
2. `room.onStateChange` — map server state → Pinia.
3. `room.onError` — set `this.error`.
4. `room.onLeave` — call `_resetRoomState()`.

### `_resetRoomState()`

Clear session fields: `room`, `roomId`, `board` (empty 8×8), `myColor`, `currentTurn`, `status = 'idle'`. Does **not** clear lobby `rooms` / `listing` / `error` (leave that to callers when appropriate).

## leaveGame

```ts
async leaveGame() {
  const room = this.room;
  this._resetRoomState();
  if (room) {
    try {
      await room.leave();
    } catch {
      // room may already be closed
    }
  }
}
```

- Reset Pinia first, then call `leave`.
- **Swallow** closed-room errors — do not surface them as `game.error`.
- `_enterRoom` always calls `leaveGame` before connecting so only one room is attached.

## onStateChange Mapping

Expected server state (coordinate with `../happy-tourist-server`):

| Server field | Store field |
|--------------|-------------|
| `board` | `board` (`CellValue[][]`: `0` empty, `1` white, `2` black, `3` white king, `4` black king`) |
| `currentTurn` | `currentTurn` (`'white' \| 'black'`) |
| `status` | `status` (`'waiting' \| 'playing' \| 'finished'`) |
| `players[sessionId].color` | `myColor` via `room.sessionId` |

```ts
room.onStateChange((state) => {
  // board → this.board
  // currentTurn → this.currentTurn
  // status → this.status
  const me = state.players?.[room.sessionId];
  if (me?.color) this.myColor = me.color;
});

room.onError((_code, message) => {
  this.error = message || 'Room error';
});

room.onLeave(() => {
  this._resetRoomState();
});
```

`onLeave` always resets state (remote kick, disconnect, or peer close). Do not leave stale `room` / `myColor` after leave.

## GamePage Rejoin After Refresh

Pinia is in-memory. After full page refresh, `game.room` is null but the hash route still has `/game/:roomId`.

In `GamePage` `onMounted`:

1. Read `route.params.roomId`.
2. If **`!game.room && roomId`** → `await game.joinGame(roomId)` (rejoin via `_enterRoom` → `joinById`).
3. If join throws → `router.replace({ name: 'lobby' })`.
4. If **`!game.room`** and no `roomId` → lobby.

```ts
onMounted(async () => {
  const roomId = /* string from route.params.roomId */;
  if (!game.room && roomId) {
    try {
      await game.joinGame(roomId);
    } catch {
      await router.replace({ name: 'lobby' });
    }
  } else if (!game.room) {
    await router.replace({ name: 'lobby' });
  }
});
```

Normal lobby → game navigation already has `game.room` set; skip rejoin.

**Do not** attach room listeners in the page — rejoin only calls `joinGame`; `_attachRoom` wires listeners in the store.

## Moves

```ts
sendMove(from, to) {
  if (!this.room) return;
  this.room.send('move', { from, to });
}
```

- Gate UI with getter `canMove` (`status === 'playing'` && `currentTurn === myColor`).
- Local cell highlights on `GamePage` are **UI hints only**; board truth is server state via `onStateChange`.
- Do not invent new message types without updating the server room handler.

## Do / Don't

| Do | Don't |
|----|--------|
| Enter via `createGame` / `joinGame` → `_enterRoom` | Call `client.create` / `joinById` from a page |
| Leave before enter (`leaveGame` inside `_enterRoom`) | Attach a second room without leaving the first |
| Wire `onStateChange` / `onError` / `onLeave` in `_attachRoom` | Put room listeners in `GamePage` or components |
| Rejoin with `joinGame(roomId)` when Pinia lost room | Assume `game.room` survives refresh |
| On rejoin failure → lobby | Leave the user on `/game/:id` with empty board |
| Swallow closed-room errors in `leaveGame` | Treat `room.leave()` failure as a user-facing banner |
| Use `CHECKERS_ROOM` | Hardcode `'checkers'` in pages or invent a new room name alone |
| Change schema/protocol with `../happy-tourist-server` | Assume board/turn/players shape without checking server |

## Change Checklist

1. Belongs in `stores/game` (listeners + connect), not the page.
2. New connect path still goes through `_enterRoom` (leave → connect → `_attachRoom`).
3. State fields / `move` payload match `../happy-tourist-server`.
4. `onLeave` / `leaveGame` still call `_resetRoomState`.
5. `GamePage` rejoin-by-`roomId` still works after refresh; fail → lobby.
6. Pages only call store actions and bind store state.
7. Propose `npm run lint` / `npm run typecheck`; wait for user «готово».

## Related

- Broader Colyseus I/O: `.agents/skills/client/colyseus-client/SKILL.md`
- Auth before rooms: `.agents/skills/client/client-work-with-auth/SKILL.md`
- Client overview: `AGENTS.md` (game session / board cell values)
- Sibling server: `../happy-tourist-server`
