---
name: work-with-rooms
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room lifecycle in
  the happy-tourist tourist client: createGame / joinGame / rejoinGame /
  leaveGame, localStorage tourist reconnection token, _enterRoom /
  _attachRoom / _resetRoomState, onStateChange / onError / onLeave, GamePage
  reconnect-then-joinById after refresh / browser reopen, or TOURIST_ROOM wiring.
---

# Work With Rooms

Use this skill for **Colyseus room lifecycle** in the happy-tourist client (`happy-tourist.github.io`).

Room connect, listeners, and leave live in **`src/stores/game.ts`**. Pages call store actions only. Do **not** put `room.onStateChange` / `onError` / `onLeave` in pages.

Coordinate schema / protocol (room name, state shape, seat connectivity, `move` message) with [`../happy-tourist-server`](../../../../happy-tourist-server).

**Tourist vs lobby:** reconnection grace and `localStorage` token apply **only** to room `tourist`. Lobby listing is fire-and-forget / quiet resubscribe — see `work-with-lobby` (design D3 / D7).

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Store | `src/stores/game.ts` | `TOURIST_ROOM`, create/join/rejoin/leave, token persist, `_attachRoom` listeners |
| Lobby | `src/pages/LobbyPage.vue` | `createGame` / `joinGame` → navigate to `game` with `roomId` |
| Game | `src/pages/GamePage.vue` | Board + presence; `rejoinGame(roomId)` on mount / soft-fail; `leaveGame` |
| Boot | `src/boot/colyseus.ts` | Shared `Client` (`VITE_COLYSEUS_URL`) |
| Route | `/game/:roomId` | Hash mode; `meta.requiresAuth` |

Room type constant: `TOURIST_ROOM = 'tourist'`.

## Lifecycle Flow

```text
createGame / joinGame(roomId?) / joinGame() / rejoinGame(roomId)
        │
        ▼
  _enterRoom(connect)
        │  status = 'connecting'; clear error
        │  await _leaveTouristRoom()  ← prior tourist only; lobby stays live
        │  room = await connect()      ← create | joinById | joinOrCreate | reconnect
        │  _attachRoom(room)           ← BEFORE any await; save tourist token
        │  await unsubscribeLobby()    ← SC-LOBBY-05 only on success
        ▼
  listeners (store only)
        │  onStateChange → seats (incl. connected/reconnectUntil) / started / sessionId
        │  onError → error string
        │  onLeave → _resetRoomState()  ← keeps localStorage token (unexpected drop)
        ▼
  leaveGame() (consented)
        │  clearTouristReconnect(); unsubscribeLobby(); _resetRoomState(); room.leave()
```

### Public actions → SDK

| Intent | Store action | SDK |
|--------|--------------|-----|
| New room | `createGame(options?)` | `client.create(TOURIST_ROOM, options)` |
| Join by id | `joinGame(roomId, options?)` | `client.joinById(roomId, options)` |
| Join or create | `joinGame()` (no id) | `client.joinOrCreate(TOURIST_ROOM, options)` |
| Rejoin after F5 | `rejoinGame(roomId)` | `client.reconnect(token)` then fallback `joinById` |
| Leave (consented) | `leaveGame()` | clear token + `unsubscribeLobby` + `room.leave()` after `_resetRoomState` |
| Game messages | Deferred until rules land | Coordinate with `work-with-game` |

All connect paths go through `_enterRoom`. Do not call `client.create` / `joinById` / `joinOrCreate` / `reconnect` from pages.

## Private Actions Pattern

Three private helpers own the room session (`_enterRoom`, `_attachRoom`, `_resetRoomState`) plus `_leaveTouristRoom` for enter cleanup. Keep this split when extending the store.

### `_enterRoom(connect)`

1. Set `status = 'connecting'`, `error = null`.
2. **`await this._leaveTouristRoom()`** — detach prior tourist room; **keep** lobby subscription during the attempt.
3. `const room = await connect()`.
4. **`this._attachRoom(room)`** — immediately after connect (before any other await) so the first `ROOM_STATE` is not missed.
5. **`await this.unsubscribeLobby()`** — only after success (SC-LOBBY-05); return `room`.
6. On failure: `status = 'idle'`, set `error`, rethrow (lobby stays subscribed on LobbyPage).

```ts
async _enterRoom(connect: () => Promise<Room>) {
  this.status = 'connecting';
  this.error = null;
  try {
    await this._leaveTouristRoom();
    const room = await connect();
    // MUST attach before await unsubscribeLobby — first ROOM_STATE can arrive
    // during that gap; missing the listener leaves seats[] empty on Game.
    this._attachRoom(room);
    await this.unsubscribeLobby();
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

1. Assign `this.room`, `this.roomId = room.roomId`, `this.sessionId = room.sessionId`, `this.status = 'waiting'`.
2. **`saveTouristReconnect(room)`** — persist `{ roomId, token: room.reconnectionToken }` in **`localStorage`** key `ht-tourist-reconnect` (design D3; **never** lobby). `loadTouristReconnect` MAY one-shot migrate the same key from legacy `sessionStorage`.
3. `room.onStateChange` — map `started` / `seats` (incl. connectivity) → Pinia; refresh token (may rotate); also mirror `room.state` once if already present.
4. `room.onError` — set `this.error`.
5. `room.onLeave` — call `_resetRoomState()` **without** clearing the tourist token (unexpected drop / soft disconnect — F5 may still `reconnect`).

**Ordering:** call `_attachRoom` immediately after `connect()` succeeds — **before** any other `await` (including `unsubscribeLobby`).

### `_resetRoomState()`

Clear session fields: `room`, `roomId`, `sessionId`, `started`, `seats`, `status = 'idle'`. Does **not** clear lobby `rooms` / `listing` / `error` and does **not** clear the tourist reconnection token (callers that consent to leave must clear it explicitly).

## Tourist reconnection token (D3)

| Event | Token |
|-------|--------|
| Successful tourist enter / soft reconnect | Save `reconnectionToken` + `roomId` to `localStorage` |
| Consented `leaveGame` / `_leaveTouristRoom` (swap rooms) | **Clear** `localStorage` + legacy `sessionStorage` key |
| Unexpected `onLeave` (drop) | **Keep** storage for remount / F5 / browser reopen |
| Failed `reconnect` (stale / invalid) | **Clear** stale token (both storages), then fresh `joinById` |
| Lobby subscribe | **Never** write lobby token |

Cross-tab: shared `localStorage` — a second tab MAY `reconnect` and take the seat (accepted). Missing or invalid token = fresh `joinById` (SC-PIECE-18), not userId reclaim.

## leaveGame

```ts
async leaveGame() {
  clearTouristReconnect(); // consented leave (D3)
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

- Used for logout / explicit leave (Lobby «Выйти», GamePage «Лобби») — **consented** leave on server (immediate seat remove, no grace).
- Clear tourist token, reset Pinia, then call `leave`.
- **Swallow** closed-room errors — do not surface them as `game.error`.
- `_enterRoom` uses `_leaveTouristRoom` (not `leaveGame`) so a failed enter keeps the lobby list live; `_leaveTouristRoom` also clears the prior tourist token when leaving a live prior room.

## onStateChange Mapping

**Today** (seats / pieces / connectivity — coordinate with `../happy-tourist-server`):

| Server field | Store field |
|--------------|-------------|
| `started` | `started`; also drives `status` (`playing` if started, else `waiting`) |
| `seats` Map (key = `sessionId`) | `seats[]` with `sessionId`, `touristId`, `pieces[]` (`side`/`row`/`col`), **`connected`**, **`reconnectUntil`** |
| (room) `sessionId` | `sessionId` — for `mySeat` / strip×4 / presence self |

- `connected` — `true` when online; `false` during reconnect grace (SC-PIECE-16).
- `reconnectUntil` — unix ms deadline while offline; `0` when online (design D2). Presence countdown uses this, not a local “30” without deadline.
- Mirror via `_mirrorRoomState`; default `connected !== false` if field missing for older peers.

Move messages and turn/progress fields — **deferred** until rules land (`work-with-game` + client board skill).

```ts
// inside _mirrorRoomState / onStateChange
next.push({
  sessionId,
  touristId: Number(seat.touristId),
  pieces,
  connected: seat.connected !== false,
  reconnectUntil: Number(seat.reconnectUntil ?? 0),
});
saveTouristReconnect(room); // token may rotate after soft reconnect
```

`onLeave` always resets Pinia room state (remote kick, disconnect, or peer close) but **keeps** the tourist reconnection token so Game remount can call `rejoinGame`. Consented leave clears the token in `leaveGame` / `_leaveTouristRoom`.

## GamePage Rejoin After Refresh / Soft Fail

Pinia is in-memory. After full page refresh or browser reopen, `game.room` is null but the hash route still has `/game/:roomId`. Soft SDK drops keep the Room object while auto-reconnect runs; if SDK gives up, store `onLeave` clears Pinia but **keeps** the token in `localStorage` within the 30 s server grace.

In `GamePage`:

1. Read `route.params.roomId`.
2. If **`!game.room && roomId`** → `await game.rejoinGame(roomId)`:
   - If stored token matches `roomId` → try `client.reconnect(token)` first.
   - On failure (grace expired / invalid) → **clear stale token**, then fallback `joinById` (spectator / new seat before start).
3. If rejoin throws → `router.replace({ name: 'lobby' })`.
4. If **`!game.room`** and no `roomId` → lobby.
5. Also **`watch(game.room)`**: when a live room becomes null while still on Game (SDK soft-fail), call the same rejoin helper — **unless** consented `leaveGame` is in progress (guard with a local flag so «Лобби» does not immediately `joinById` again).

```ts
// onMounted + watch(game.room lost) → ensureTouristRoom()
// consentedLeaving=true around leaveGame so the watch does not rejoin
async function ensureTouristRoom() {
  if (consentedLeaving || game.room) return;
  const roomId = /* string from route.params.roomId */;
  if (!roomId) {
    await router.replace({ name: 'lobby' });
    return;
  }
  try {
    await game.rejoinGame(roomId);
  } catch {
    await router.replace({ name: 'lobby' });
  }
}
```

```ts
async rejoinGame(roomId: string, options = {}) {
  const saved = loadTouristReconnect();
  if (saved && saved.roomId === roomId) {
    try {
      return await this._enterRoom(() => client.reconnect(saved.token));
    } catch {
      clearTouristReconnect(); // stale / invalid — SC-PIECE-18 fresh join
    }
  }
  return this._enterRoom(() => client.joinById(roomId, options));
}
```

Normal lobby → game navigation already has `game.room` set; skip rejoin.

**Do not** attach room listeners in the page — rejoin only calls `rejoinGame`; `_attachRoom` wires listeners in the store.

## Game messages

**None today** — seating syncs via schema; board/pieces are non-interactive (no `sendMove` / selection UX). When move rules land, add store send helpers + server `onMessage` in lockstep; do not reintroduce legacy draughts `move` `{ from, to }` unless product explicitly revives that contract.

## Do / Don't

| Do | Don't |
|----|--------|
| Enter via `createGame` / `joinGame` / `rejoinGame` → `_enterRoom` | Call `client.create` / `joinById` / `reconnect` from a page |
| Persist tourist token in `localStorage` only | Persist lobby token or use `sessionStorage` for reconnect |
| Clear token on consented `leaveGame` and failed `reconnect` | Clear token on unexpected `onLeave` (blocks F5 / reopen revive) |
| Mirror `connected` / `reconnectUntil` into `seats[]` | Invent client-only offline flags without schema |
| Leave before enter (`_leaveTouristRoom` inside `_enterRoom`; unsubscribe lobby on success) | Attach a second tourist room without leaving the first |
| Wire `onStateChange` / `onError` / `onLeave` in `_attachRoom` | Put room listeners in `GamePage` or components |
| Rejoin with `rejoinGame(roomId)` when Pinia lost room | Assume `game.room` survives refresh; skip reconnect and only `joinById` |
| On rejoin failure → lobby | Leave the user on `/game/:id` without a room |
| Swallow closed-room errors in `leaveGame` | Treat `room.leave()` failure as a user-facing banner |
| Use `TOURIST_ROOM` | Hardcode `'tourist'` in pages or invent a new room name alone |
| Change schema/protocol with `../happy-tourist-server` | Invent synced board/turn/move fields without a rules change |

## Change Checklist

1. Belongs in `stores/game` (listeners + connect), not the page.
2. New connect path still goes through `_enterRoom` (`_leaveTouristRoom` → connect → `_attachRoom` → `unsubscribeLobby`).
3. State fields / connectivity / `move` payload match `../happy-tourist-server`.
4. Consented `leaveGame` clears tourist token; unexpected `onLeave` keeps it; failed `reconnect` clears stale then `joinById`.
5. `GamePage` uses `rejoinGame` on mount and when Pinia loses the room unexpectedly (reconnect → `joinById`); fail → lobby; consented leave guarded against auto-rejoin.
6. Pages only call store actions and bind store state.
7. Lobby reconnect policy stays in `work-with-lobby` (no tourist grace on lobby).
8. Run `npm run lint` / `npm run typecheck` from the client package root; fix failures before claiming done.

## Related

- Broader Colyseus I/O: `.agents/skills/client/colyseus-client/SKILL.md`
- Lobby quiet listing: `.agents/skills/client/work-with-lobby/SKILL.md`
- Presence UI: `.agents/skills/client/work-with-game-board/SKILL.md`
- Auth before rooms: `.agents/skills/client/client-work-with-auth/SKILL.md`
- Client overview: `AGENTS.md` (game session / seats / board)
- Sibling server: `../happy-tourist-server`
