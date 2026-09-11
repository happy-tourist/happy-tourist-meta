---
name: work-with-lobby
description: >-
  Guides lobby room list, 5s poll, create / join / joinOrCreate, and navigation
  to /game/:roomId in the happy-tourist checkers client. Use when changing
  LobbyPage, game.refreshRooms / createGame / joinGame, CHECKERS_ROOM listing
  via client.http.get('/rooms/checkers'), lobby loading flags, game.error
  banners, or logout from the lobby.
---

# Work With Lobby

Use this skill for the **lobby** in the happy-tourist checkers client (`happy-tourist.github.io`): room list, poll, create / join / joinOrCreate, then enter the game route.

Stack: Vue 3 `<script setup>`, Quasar 2, Pinia `useGameStore` / `useAuthStore`, `@colyseus/sdk` 0.18.

Sibling server: `../happy-tourist-server`. Coordinate room name (`checkers`) and list metadata with that package.

## Quick Reference

| Topic | Pattern |
|-------|---------|
| Page | `src/pages/LobbyPage.vue` — route `/lobby`, `meta.requiresAuth` |
| Store | `src/stores/game.ts` — `rooms`, `listing`, `error`, `refreshRooms`, `createGame`, `joinGame`, `leaveGame` |
| Room name | `CHECKERS_ROOM = 'checkers'` |
| List rooms | `refreshRooms` → `client.http.get('/rooms/checkers')` — **not** `getAvailableRooms` |
| Poll | `onMounted`: `refreshRooms` + `setInterval(..., 5000)`; `onUnmounted`: `clearInterval` |
| Play | `joinGame()` (no id) → `client.joinOrCreate(CHECKERS_ROOM)` |
| Create | `createGame()` → `client.create(CHECKERS_ROOM)` |
| Join by id | `joinGame(roomId)` → `client.joinById(roomId)` |
| After enter | `router.push({ name: 'game', params: { roomId } })` |
| Loading | Store `listing`; page refs `creating`, `joining` |
| Errors | `game.error` + `q-banner` |
| Logout | `game.leaveGame()` → `auth.logout()` → `replace({ name: 'login' })` |
| I/O boundary | Pages call store actions only; Colyseus stays in Pinia |

## Rules

| Do | Don't |
|----|--------|
| List with `client.http.get(\`/rooms/${CHECKERS_ROOM}\`)` inside `refreshRooms` | Call `client.getAvailableRooms` (removed in SDK 0.16+) |
| Poll every **5s** on lobby mount; clear the timer on unmount | Leave `setInterval` running after leave lobby |
| Play → `joinGame()`; Create → `createGame()`; list click → `joinGame(roomId)` | Invent parallel enter helpers on the page |
| Navigate to `/game/:roomId` only after a successful enter | Stay on lobby with a live room and no route change |
| Bind `:loading="game.listing"` / `creating` / `joining` | Leave buttons clickable during connect |
| Show `game.error` with `q-banner` | Duplicate a second error channel |
| Logout via `useAuthStore().logout()` after `leaveGame()` | Call `client.auth.signOut` from LobbyPage |
| Keep room type / metadata in sync with `../happy-tourist-server` | Change `CHECKERS_ROOM` or list fields without the server |

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Page | `src/pages/LobbyPage.vue` | List UI, poll, Play / Create / Join, logout |
| Store | `src/stores/game.ts` | HTTP list + room enter / leave |
| Auth | `src/stores/auth.ts` | `displayName`, `logout` |
| Client | `src/boot/colyseus.ts` | Shared `Client` (`VITE_COLYSEUS_URL`) |
| Route | `src/router/routes.ts` | `/lobby` → name `lobby`; `/game/:roomId` → name `game` |

List row shape (`RoomAvailable<GameRoomMeta>`): `roomId`, `clients`, `maxClients`, `metadata?.title`, `metadata?.status` (`waiting` \| `playing` \| `finished`).

## Room list — `refreshRooms`

```ts
async refreshRooms() {
  this.listing = true;
  this.error = null;
  try {
    // Requires server GET /rooms/:roomName (getAvailableRooms removed in 0.16+)
    const { data } = await client.http.get(`/rooms/${CHECKERS_ROOM}`);
    this.rooms = (data ?? []) as RoomAvailable<GameRoomMeta>[];
  } catch (e) {
    this.error = e instanceof Error ? e.message : String(e);
    this.rooms = [];
  } finally {
    this.listing = false;
  }
}
```

- Path resolves to `GET /rooms/checkers`.
- Manual refresh button also calls `game.refreshRooms()` with `:loading="game.listing"`.

## Poll (LobbyPage)

```ts
onMounted(() => {
  void game.refreshRooms();
  pollTimer = setInterval(() => void game.refreshRooms(), 5000);
});

onUnmounted(() => {
  if (pollTimer) clearInterval(pollTimer);
});
```

Always clear the interval on unmount so lobby polling stops after navigate to game or login.

## Enter flows

| UI | Page handler | Store | SDK |
|----|--------------|-------|-----|
| «Играть» (Play) | `onPlay` | `joinGame()` | `joinOrCreate(CHECKERS_ROOM)` |
| «Создать игру» (Create) | `onCreate` | `createGame()` | `create(CHECKERS_ROOM)` |
| List row / «Войти» | `onJoin(roomId)` | `joinGame(roomId)` | `joinById(roomId)` |

All go through `_enterRoom`: clear prior room → connect → `_attachRoom` → return `Room`. Page then navigates:

```ts
const room = await game.joinGame(); // or createGame / joinGame(id)
await router.push({ name: 'game', params: { roomId: room.roomId } });
```

Page `catch` is empty on purpose — failure already sets `game.error`.

## Loading flags

| Flag | Where | Covers |
|------|-------|--------|
| `game.listing` | store | `refreshRooms` (poll + manual) |
| `joining` | LobbyPage `ref` | Play + join-by-id |
| `creating` | LobbyPage `ref` | Create |

Reset page refs in `finally`. Store clears `listing` in `finally`.

## Errors And Logout

- API / connect failures → `game.error` string → `q-banner` (`bg-negative`).
- Logout: `await game.leaveGame()` → `await auth.logout()` → `router.replace({ name: 'login' })`.

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

1. Listing uses `refreshRooms` → `client.http.get('/rooms/checkers')`, not `getAvailableRooms`.
2. Lobby polls every 5s on mount; interval cleared on unmount.
3. Play / Create / Join map to `joinGame()` / `createGame()` / `joinGame(roomId)`.
4. Successful enter navigates to `/game/:roomId` (route name `game`).
5. `listing` / `creating` / `joining` bound on buttons; cleared in `finally`.
6. Failures surface as `game.error` + `q-banner`.
7. Logout goes through auth store (after `leaveGame`).
8. Room name and list metadata match `../happy-tourist-server`.
9. No `client.*` calls from LobbyPage — only store actions.
10. Propose `npm run lint` / `typecheck`; wait for user «готово».

## Common mistakes

| Mistake | Fix |
|---------|-----|
| `client.getAvailableRooms('checkers')` | `client.http.get('/rooms/checkers')` via `refreshRooms` |
| Poll without `clearInterval` | Clear in `onUnmounted` |
| Play calling `createGame` | Play = `joinGame()` → `joinOrCreate` |
| Join without `roomId` when clicking a list row | Pass `room.roomId` into `joinGame(roomId)` |
| Enter success but no navigation | `push({ name: 'game', params: { roomId } })` |
| Calling `client.create` / `joinById` in the page | Use `createGame` / `joinGame` on the store |
| Ignoring `game.error` | Show `q-banner` |
| Hardcoding a new room name in the client only | Align `CHECKERS_ROOM` + server registration + `GET /rooms/:roomName` |

## Related

- Broader Colyseus I/O: `.agents/skills/client/colyseus-client/SKILL.md`
- Auth / logout details: `.agents/skills/client/client-work-with-auth/SKILL.md`
- Overview: `AGENTS.md` (Lobby / rooms)
- Server: `../happy-tourist-server` — room type `checkers`, HTTP `GET /rooms/:roomName`
