---
name: work-with-rooms
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room lifecycle in
  the happy-tourist tourist client: createGame / joinGame / leaveGame,
  _enterRoom / _attachRoom / _resetRoomState, onStateChange / onError / onLeave,
  GamePage rejoin by roomId after refresh, or TOURIST_ROOM wiring.
---

# Work With Rooms

Use this skill for **Colyseus room lifecycle** in the happy-tourist client (`happy-tourist.github.io`).

Room connect, listeners, and leave live in **`src/stores/game.ts`**. Pages call store actions only. Do **not** put `room.onStateChange` / `onError` / `onLeave` in pages.

Coordinate schema / protocol (room name, state shape, `move` message) with [`../happy-tourist-server`](../../../../happy-tourist-server).

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Store | `src/stores/game.ts` | `TOURIST_ROOM`, create/join/leave, `_attachRoom` listeners |
| Lobby | `src/pages/LobbyPage.vue` | `createGame` / `joinGame` → navigate to `game` with `roomId` |
| Game | `src/pages/GamePage.vue` | Static tourist board; rejoin by `roomId`; `leaveGame` |
| Boot | `src/boot/colyseus.ts` | Shared `Client` (`VITE_COLYSEUS_URL`) |
| Route | `/game/:roomId` | Hash mode; `meta.requiresAuth` |

Room type constant: `TOURIST_ROOM = 'tourist'`.

## Lifecycle Flow

```text
createGame / joinGame(roomId?) / joinGame()
        │
        ▼
  _enterRoom(connect)
        │  status = 'connecting'; clear error
        │  await _leaveTouristRoom()  ← prior tourist only; lobby stays live
        │  room = await connect()      ← create | joinById | joinOrCreate
        │  await unsubscribeLobby()    ← SC-LOBBY-05 only on success
        │  _attachRoom(room)
        ▼
  listeners (store only)
        │  onStateChange → status (synced rules fields later)
        │  onError → error string
        │  onLeave → _resetRoomState()
        ▼
  leaveGame() or remote leave
        │  unsubscribeLobby(); _resetRoomState(); room.leave() (swallow)
```

### Public actions → SDK

| Intent | Store action | SDK |
|--------|--------------|-----|
| New room | `createGame(options?)` | `client.create(TOURIST_ROOM, options)` |
| Join by id | `joinGame(roomId, options?)` | `client.joinById(roomId, options)` |
| Join or create | `joinGame()` (no id) | `client.joinOrCreate(TOURIST_ROOM, options)` |
| Leave | `leaveGame()` | `unsubscribeLobby` + `room.leave()` after `_resetRoomState` |
| Game messages | Deferred until rules land | Coordinate with `work-with-game` |

All connect paths go through `_enterRoom`. Do not call `client.create` / `joinById` / `joinOrCreate` from pages.

## Private Actions Pattern

Three private helpers own the room session (`_enterRoom`, `_attachRoom`, `_resetRoomState`) plus `_leaveTouristRoom` for enter cleanup. Keep this split when extending the store.

### `_enterRoom(connect)`

1. Set `status = 'connecting'`, `error = null`.
2. **`await this._leaveTouristRoom()`** — detach prior tourist room; **keep** lobby subscription during the attempt.
3. `const room = await connect()`.
4. **`await this.unsubscribeLobby()`** — only after success (SC-LOBBY-05).
5. `this._attachRoom(room)` and return `room`.
6. On failure: `status = 'idle'`, set `error`, rethrow (lobby stays subscribed on LobbyPage).

```ts
async _enterRoom(connect: () => Promise<Room>) {
  this.status = 'connecting';
  this.error = null;
  try {
    await this._leaveTouristRoom();
    const room = await connect();
    await this.unsubscribeLobby();
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

Clear session fields: `room`, `roomId`, `status = 'idle'`. Does **not** clear lobby `rooms` / `listing` / `error` (leave that to callers when appropriate).

## leaveGame

```ts
async leaveGame() {
  await this.unsubscribeLobby();
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

- Used for logout / explicit leave (Lobby «Выйти», GamePage «Лобби»).
- Reset Pinia first, then call `leave`.
- **Swallow** closed-room errors — do not surface them as `game.error`.
- `_enterRoom` uses `_leaveTouristRoom` (not `leaveGame`) so a failed enter keeps the lobby list live.

## onStateChange Mapping

**Today** (static tourist board phase — coordinate with `../happy-tourist-server`):

| Server field | Store field |
|--------------|-------------|
| `status` (optional) | `status` (`'waiting' \| 'playing' \| 'finished'`) |

Scaffold server state may still expose `mySynchronizedProperty`; client maps only what the product store needs. Synced board / turn / seats — **deferred** until rules land (`work-with-game` + client board skill).

```ts
room.onStateChange((state) => {
  const s = state as { status?: 'waiting' | 'playing' | 'finished' };
  if (s.status) this.status = s.status;
});

room.onError((_code, message) => {
  this.error = message || 'Room error';
});

room.onLeave(() => {
  this._resetRoomState();
});
```

`onLeave` always resets state (remote kick, disconnect, or peer close). Do not leave a stale `room` after leave.

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

## Game messages

**None today** — `GamePage` is a static tourist board (no `sendMove` / selection UX). When rules land, add store send helpers + server `onMessage` in lockstep; do not reintroduce legacy draughts `move` `{ from, to }` unless product explicitly revives that contract.

## Do / Don't

| Do | Don't |
|----|--------|
| Enter via `createGame` / `joinGame` → `_enterRoom` | Call `client.create` / `joinById` from a page |
| Leave before enter (`_leaveTouristRoom` inside `_enterRoom`; unsubscribe lobby on success) | Attach a second tourist room without leaving the first |
| Wire `onStateChange` / `onError` / `onLeave` in `_attachRoom` | Put room listeners in `GamePage` or components |
| Rejoin with `joinGame(roomId)` when Pinia lost room | Assume `game.room` survives refresh |
| On rejoin failure → lobby | Leave the user on `/game/:id` without a room |
| Swallow closed-room errors in `leaveGame` | Treat `room.leave()` failure as a user-facing banner |
| Use `TOURIST_ROOM` | Hardcode `'tourist'` in pages or invent a new room name alone |
| Change schema/protocol with `../happy-tourist-server` | Invent synced board/turn/move fields without a rules change |

## Change Checklist

1. Belongs in `stores/game` (listeners + connect), not the page.
2. New connect path still goes through `_enterRoom` (`_leaveTouristRoom` → connect → `unsubscribeLobby` → `_attachRoom`).
3. State fields / `move` payload match `../happy-tourist-server`.
4. `onLeave` / `leaveGame` still call `_resetRoomState`.
5. `GamePage` rejoin-by-`roomId` still works after refresh; fail → lobby.
6. Pages only call store actions and bind store state.
7. Run `npm run lint` / `npm run typecheck` from the client package root; fix failures before claiming done.

## Related

- Broader Colyseus I/O: `.agents/skills/client/colyseus-client/SKILL.md`
- Auth before rooms: `.agents/skills/client/client-work-with-auth/SKILL.md`
- Client overview: `AGENTS.md` (game session / board cell values)
- Sibling server: `../happy-tourist-server`
