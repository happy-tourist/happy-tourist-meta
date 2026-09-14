---
name: work-with-lobby
description: >-
  Guides live lobby room list via LobbyRoom subscribe/unsubscribe, leave policy
  before enter tourist, create / join / joinOrCreate, and navigation to
  /game/:roomId in the happy-tourist tourist client. Use when changing
  LobbyPage, game.subscribeLobby / unsubscribeLobby / createGame / joinGame,
  LOBBY_ROOM / TOURIST_ROOM listing, lobby loading flags, game.error banners,
  or logout from the lobby.
---

# Work With Lobby

Use this skill for the **lobby** in the happy-tourist tourist client (`happy-tourist.github.io`): live room list via built-in Colyseus `LobbyRoom`, create / join / joinOrCreate, then enter the game route.

Stack: Vue 3 `<script setup>`, Quasar 2, Pinia `useGameStore` / `useAuthStore`, `@colyseus/sdk` 0.18.

Sibling server: `../happy-tourist-server`. Coordinate room name (`tourist`), `lobby` registration, and list metadata with that package.

## Quick Reference

| Topic | Pattern |
|-------|---------|
| Page | `src/pages/LobbyPage.vue` — route `/lobby`, `meta.requiresAuth` |
| Store | `src/stores/game.ts` — `rooms`, `lobbyRoom`, `listing`, `error`, `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame` |
| Room names | `TOURIST_ROOM = 'tourist'`; `LOBBY_ROOM = 'lobby'` |
| Live list | `subscribeLobby` → `client.joinOrCreate(LOBBY_ROOM, { filter: { name: TOURIST_ROOM } })` + handlers `rooms` / `+` / `-` |
| Leave lobby | `unsubscribeLobby` after successful tourist connect (`_enterRoom`); also on LobbyPage unmount and `leaveGame` / logout |
| Play | `joinGame()` (no id) → `client.joinOrCreate(TOURIST_ROOM)` |
| Create | `createGame()` → `client.create(TOURIST_ROOM)` |
| Join by id | `joinGame(roomId)` → `client.joinById(roomId)` |
| After enter | `router.push({ name: 'game', params: { roomId } })` |
| Loading | Store `listing` during subscribe connect; page refs `creating`, `joining` |
| Errors | `game.error` + `q-banner` (subscribe fail / lobby `onError`) |
| Logout | `game.leaveGame()` → `auth.logout()` → `replace({ name: 'login' })` |
| I/O boundary | Pages call store actions only; Colyseus stays in Pinia |
| HTTP fallback | `refreshRooms` → `client.http.get('/rooms/tourist')` exists but **LobbyPage must not poll it** |

## Rules

| Do | Don't |
|----|--------|
| Subscribe with `joinOrCreate('lobby', { filter: { name: TOURIST_ROOM } })` | Poll `setInterval` + HTTP `GET /rooms/tourist` from LobbyPage |
| Mount → `subscribeLobby`; unmount → `unsubscribeLobby` | Leave lobby WS open after navigate to game or login |
| Call `unsubscribeLobby` after successful `tourist` connect (`_enterRoom`); keep lobby on failed enter | Keep lobby + tourist sockets both live on GamePage |
| Play → `joinGame()`; Create → `createGame()`; list click → `joinGame(roomId)` | Invent parallel enter helpers on the page |
| Navigate to `/game/:roomId` only after a successful enter | Stay on lobby with a live room and no route change |
| Bind listing loading / `creating` / `joining` | Leave buttons clickable during connect |
| Show `game.error` with `q-banner` | Duplicate a second error channel |
| Logout via `useAuthStore().logout()` after `leaveGame()` | Call `client.auth.signOut` from LobbyPage |
| Keep room type / metadata in sync with `../happy-tourist-server` | Change `TOURIST_ROOM` / `LOBBY_ROOM` without the server |

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Page | `src/pages/LobbyPage.vue` | List UI, subscribe lifecycle, Play / Create / Join, logout |
| Store | `src/stores/game.ts` | LobbyRoom subscribe + room enter / leave |
| Auth | `src/stores/auth.ts` | `displayName`, `logout` |
| Client | `src/boot/colyseus.ts` | Shared `Client` (`VITE_COLYSEUS_URL`) |
| Route | `src/router/routes.ts` | `/lobby` → name `lobby`; `/game/:roomId` → name `game` |

List row shape (`RoomAvailable<GameRoomMeta>`): `roomId`, `clients`, `maxClients`, `metadata?.title`, `metadata?.status` (`waiting` \| `playing` \| `finished`).

## Live list — `subscribeLobby` / `unsubscribeLobby`

```ts
async subscribeLobby() {
  await this.unsubscribeLobby();
  this.listing = true;
  this.error = null;

  try {
    const lobby = await client.joinOrCreate(LOBBY_ROOM, {
      filter: { name: TOURIST_ROOM },
    });
    this.lobbyRoom = lobby;

    lobby.onMessage('rooms', (rooms) => {
      this.rooms = rooms ?? [];
    });
    lobby.onMessage('+', ([roomId, room]) => {
      /* upsert into this.rooms */
    });
    lobby.onMessage('-', (roomId) => {
      this.rooms = this.rooms.filter((r) => r.roomId !== roomId);
    });
    lobby.onError((_code, message) => {
      this.error = message || 'Lobby error';
    });
  } catch (e) {
    this.error = e instanceof Error ? e.message : String(e);
    this.rooms = [];
    this.lobbyRoom = null;
  } finally {
    this.listing = false;
  }
}

async unsubscribeLobby() {
  const lobby = this.lobbyRoom;
  this.lobbyRoom = null;
  if (lobby) {
    try {
      await lobby.leave();
    } catch {
      /* already closed */
    }
  }
}
```

- Filter keeps only `tourist` rooms in the live list (design D3).
- Server must register `lobby: defineRoom(LobbyRoom)` and `tourist: …enableRealtimeListing()`.
- `refreshRooms` HTTP remains as unused fallback; do not wire it back to LobbyPage poll.

## LobbyPage lifecycle

```ts
onMounted(() => {
  void game.subscribeLobby();
});

onUnmounted(() => {
  void game.unsubscribeLobby();
});
```

No `setInterval`. Returning to `/lobby` remounts and resubscribes (SC-LOBBY-06).

## Leave policy (enter game)

`_enterRoom` detaches any prior tourist room via `_leaveTouristRoom()` (lobby stays subscribed during the attempt), then `connect()`, then `unsubscribeLobby()` **only on success** (SC-LOBBY-05 / design D2). Failed enter keeps the live list on LobbyPage (SC-LOBBY-01). GamePage therefore has only the `tourist` socket — no background lobby WS. Logout / leave still uses `leaveGame()` → `unsubscribeLobby` + leave tourist.

## Enter flows

| UI | Page handler | Store | SDK |
|----|--------------|-------|-----|
| «Играть» (Play) | `onPlay` | `joinGame()` | `joinOrCreate(TOURIST_ROOM)` |
| «Создать игру» (Create) | `onCreate` | `createGame()` | `create(TOURIST_ROOM)` |
| List row / «Войти» | `onJoin(roomId)` | `joinGame(roomId)` | `joinById(roomId)` |

All go through `_enterRoom`: clear prior tourist room → connect → `unsubscribeLobby` on success → `_attachRoom` → return `Room`. Page then navigates:

```ts
const room = await game.joinGame(); // or createGame / joinGame(id)
await router.push({ name: 'game', params: { roomId: room.roomId } });
```

Page `catch` is empty on purpose — failure already sets `game.error`.

## Loading flags

| Flag | Where | Covers |
|------|-------|--------|
| `game.listing` | store | `subscribeLobby` connect window |
| `joining` | LobbyPage `ref` | Play + join-by-id |
| `creating` | LobbyPage `ref` | Create |

Reset page refs in `finally`. Store clears `listing` in `finally` of subscribe.

## Errors And Logout

- Subscribe / lobby `onError` / connect failures → `game.error` string → `q-banner` (`bg-negative`) (SC-LOBBY-07).
- Logout: `await game.leaveGame()` → `await auth.logout()` → `router.replace({ name: 'login' })`.
- Listing error must not block logout.

## Patterns

### Play (joinOrCreate)

```ts
joining.value = true;
try {
  const room = await game.joinGame();
  await router.push({ name: 'game', params: { roomId: room.roomId } });
} catch {
  // error in store
} finally {
  joining.value = false;
}
```

### Create

```ts
creating.value = true;
try {
  const room = await game.createGame();
  await router.push({ name: 'game', params: { roomId: room.roomId } });
} catch {
} finally {
  creating.value = false;
}
```

### Join listed room

```ts
await game.joinGame(roomId);
await router.push({ name: 'game', params: { roomId } });
```

## Checklist

1. Listing uses LobbyRoom `subscribeLobby` (filter `name: tourist`), not LobbyPage HTTP poll.
2. Lobby mounts subscribe; unmount / logout / successful enter unsubscribe lobby WS; failed enter keeps subscription.
3. Play / Create / Join map to `joinGame()` / `createGame()` / `joinGame(roomId)`.
4. Successful enter navigates to `/game/:roomId` (route name `game`) with no active lobby subscription.
5. `listing` / `creating` / `joining` bound; cleared in `finally`.
6. Failures surface as `game.error` + `q-banner`; logout still works.
7. Logout goes through auth store (after `leaveGame`).
8. Room name and list metadata match `../happy-tourist-server` (`lobby` + `tourist` + realtime listing).
9. No `client.*` calls from LobbyPage — only store actions.
10. Run `npm run lint` / `typecheck` from the client package root; fix failures before claiming done.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| `setInterval` + `refreshRooms` on LobbyPage | `subscribeLobby` / `unsubscribeLobby` only |
| `client.getAvailableRooms('tourist')` | Live LobbyRoom messages; HTTP only as unused fallback |
| Keeping lobby WS on GamePage | `unsubscribeLobby` after successful connect in `_enterRoom`; `leaveGame` on logout/leave |
| Play calling `createGame` | Play = `joinGame()` → `joinOrCreate` |
| Join without `roomId` when clicking a list row | Pass `room.roomId` into `joinGame(roomId)` |
| Enter success but no navigation | `push({ name: 'game', params: { roomId } })` |
| Calling `client.create` / `joinById` in the page | Use `createGame` / `joinGame` on the store |
| Ignoring `game.error` | Show `q-banner` |
| Hardcoding a new room name in the client only | Align `TOURIST_ROOM` / `LOBBY_ROOM` + server registration |

## Related

- Broader Colyseus I/O: `.agents/skills/client/colyseus-client/SKILL.md`
- Auth / logout details: `.agents/skills/client/client-work-with-auth/SKILL.md`
- Overview: `AGENTS.md` (Lobby / rooms)
- Server: `../happy-tourist-server` — `lobby` + `tourist` + `.enableRealtimeListing()`
