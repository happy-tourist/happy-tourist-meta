---
name: colyseus-client
description: >-
  Use when adding, changing, or reviewing Colyseus client I/O in the
  happy-tourist tourist SPA: Client singleton from src/boot/colyseus.ts,
  LobbyRoom live listing (subscribeLobby), room create/join/leave,
  room.send move messages, client.auth register/sign-in/sign-out, Pinia
  auth/game stores, or VITE_COLYSEUS_URL / VITE_API_URL env wiring.
---

# Colyseus Client

Use this skill when adding or changing realtime / HTTP I/O in the **tourist client** (`happy-tourist.github.io`).

Stack: Vue 3 Composition API / `<script setup>`, Quasar 2, Pinia 4, TypeScript, `@colyseus/sdk` 0.18.

Sibling server: `../happy-tourist-server`. Coordinate room name, state schema, and message protocol with that package.

There is **no** axios layer and **no** BFF. Live lobby uses `LobbyRoom` WebSocket messages; gameplay uses room WebSocket messages. HTTP `client.http` is used for registered theme preference (`GET`/`POST` `/api/theme` in `stores/theme`); lobby listing still prefers live LobbyRoom (`GET /rooms/tourist` remains unused fallback).

## Quick Reference

| Topic | Client pattern |
|-------|----------------|
| Client singleton | `export const client = new Client(import.meta.env.VITE_COLYSEUS_URL)` in `src/boot/colyseus.ts` |
| Prefer import | `import { client } from '@/boot/colyseus'` (also `$colyseus` on `globalProperties`) |
| Where I/O lives | Pinia stores only: `stores/auth.ts`, `stores/theme.ts` (preference HTTP), `stores/game.ts` |
| Pages | Call store actions; do not call `client.*` from pages/components |
| Live lobby list | `subscribeLobby` → `joinOrCreate(LOBBY_ROOM, { filter: { name: TOURIST_ROOM } })` + `rooms` / `+` / `-` |
| HTTP fallback | `refreshRooms` → `client.http.get('/rooms/tourist')` — unused by LobbyPage; **not** `getAvailableRooms` |
| Theme preference | `stores/theme.ts` → registered `client.http.get('/api/theme')` restore (≠ JWT-only) + `post('/api/theme', { body: { theme } })` on toggle; guest uses `localStorage` only |
| Room names | `TOURIST_ROOM = 'tourist'`; `LOBBY_ROOM = 'lobby'` in `stores/game.ts` |
| Connect | `client.create` / `joinById` / `joinOrCreate` via game store actions |
| Game messages | Deferred until rules land |
| Auth | `client.auth` — register / signIn / signOut / `onChange`; token key `colyseus-auth-token` |
| Env | `VITE_COLYSEUS_URL`, `VITE_API_URL` (typed in `env.d.ts`) |
| Errors | Store `error` string; pages show `q-banner` |
| Loading | Store flags (`auth.loading`, `game.listing`, `game.status`); reset in `finally` |
| Style | Prefer `async/await` + `try/catch` + `finally` |
| Commands | Agent runs `npm run lint` / `typecheck` / `build` from client package root; may start `quasar dev` for smoke when needed |

## Rules

| Do | Don't |
|----|--------|
| Import `client` from `@/boot/colyseus` | Create a second `Client`, or call `axios` / raw `fetch` for Colyseus |
| Keep Colyseus I/O inside Pinia (`auth`, `theme`, `game`) | Scatter `client.http` / `client.create` / `room.send` across components |
| List rooms via `subscribeLobby` (LobbyRoom `rooms` / `+` / `-`) | Poll `setInterval` + HTTP, or use `client.getAvailableRooms` |
| Restore/save registered theme via `stores/theme` → `GET`/`POST` `/api/theme` | Theme HTTP from page templates; JWT-only restore after reload; save guest theme to the server |
| Use `TOURIST_ROOM` / `LOBBY_ROOM` constants | Hardcode room names in multiple places or invent names without the server |
| Enter rooms via `createGame` / `joinGame` (`_enterRoom`) | Duplicate connect + `onStateChange` wiring in pages |
| Keep room I/O in Pinia `game` store | Invent Game move UX without `work-with-game-board` / server rules |
| Auth via `client.auth.*` in `stores/auth` | Import `@colyseus/auth` on the client (that package is **server-side**) |
| Catch into store `error`; clear loading in `finally` | Leave `listing` / `loading` stuck on reject |
| Pages → stores → `client` | Pages → `client` directly |
| Change contracts with `../happy-tourist-server` | Invent synced board/turn/move shapes without a rules change |

## Client singleton

Implementation: `src/boot/colyseus.ts`.

```ts
import { defineBoot } from '#q-app';
import { Client } from '@colyseus/sdk';

export const client = new Client(import.meta.env.VITE_COLYSEUS_URL);

export default defineBoot(({ app }) => {
  app.config.globalProperties.$colyseus = client;
});
```

- One shared `Client` for the whole SPA.
- Prefer `import { client } from '@/boot/colyseus'` in `<script setup>` / stores.
- `$colyseus` is for Options API / templates only; do not use it as a second entry point for new code.
- Boot is registered in `quasar.config.ts` (`colyseus` boot file).

### Env

| Variable | Role |
|----------|------|
| `VITE_COLYSEUS_URL` | WebSocket / SDK endpoint (`new Client(...)`) |
| `VITE_API_URL` | HTTP base when needed (same host family as Colyseus) |

Local defaults: `.env.development` → `localhost:2567`. Production: `.env.production` / CI vars → `happy-tourist.duckdns.org` (WSS/HTTPS).

## Where Colyseus I/O lives

| Area | Store | SDK surface |
|------|-------|-------------|
| Auth | `stores/auth.ts` (setup store) | `client.auth.registerWithEmailAndPassword`, `signInWithEmailAndPassword`, `signInAnonymously`, `signInWithProvider('google')`, `signOut`, `onChange` |
| Lobby list | `stores/game.ts` → `subscribeLobby` / `unsubscribeLobby` | `joinOrCreate('lobby', { filter })` + messages `rooms` / `+` / `-` |
| HTTP fallback | `stores/game.ts` → `refreshRooms` | `client.http.get('/rooms/tourist')` (unused by LobbyPage) |
| Room lifecycle | `stores/game.ts` → `createGame` / `joinGame` / `leaveGame` | `client.create` / `joinById` / `joinOrCreate`, `room.leave` |
| Game board UI | `pages/GamePage.vue` | Tourist layout + seat pieces / strip (no sendMove) |
| Live state | `stores/game.ts` → `_attachRoom` | `room.onStateChange` (`seats`/`started`), `onError`, `onLeave` |

Allowed dependency direction: `pages` → `stores` / `boot` / `components`. Keep all `client.*` and `room.*` I/O in stores.

### Existing actions (do not invent parallel names)

| Store | Actions / API |
|-------|----------------|
| `auth` | `register`, `login`, `loginAnonymously`, `loginWithGoogle`, `logout`, `whenReady` |
| `game` | `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame` (`refreshRooms` HTTP unused) |

Pages already wired:

| Page | Calls |
|------|-------|
| `LoginPage` | `auth.register` / `login` / `loginAnonymously` / `loginWithGoogle` |
| `LobbyPage` | `subscribeLobby` / `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame`; `auth.logout` |
| `GamePage` | `game.joinGame(roomId)` on remount, pieces from seats, `leaveGame` |
| Router | `auth.whenReady()` before `requiresAuth` / `guest` guards |

## Auth (`client.auth`)

Token is managed by the SDK and persisted under **`colyseus-auth-token`**. Sync UI state from `client.auth.onChange` — do not hand-roll localStorage token logic.

```ts
client.auth.onChange((data) => {
  token.value = data.token ?? null;
  user.value = (data.user as AuthUser | null) ?? null;
  ready.value = true;
  resolveReady?.();
});
```

| Method | Store action |
|--------|--------------|
| `registerWithEmailAndPassword(email, password, options?)` | `register` — `options` e.g. `{ name }` passed to server `onRegisterWithEmailAndPassword` |
| `signInWithEmailAndPassword(email, password)` | `login` |
| `signInAnonymously(options?)` | `loginAnonymously` |
| `signInWithProvider('google')` | `loginWithGoogle` |
| `signOut()` | `logout` |

`whenReady()` resolves after the first `onChange` (session restore). Router `beforeEach` must `await auth.whenReady()` before deciding login vs lobby.

Pattern for auth actions — loading + error + `finally`:

```ts
async function login(email: string, password: string) {
  loading.value = true;
  error.value = null;
  try {
    await client.auth.signInWithEmailAndPassword(email, password);
  } catch (e) {
    error.value = e instanceof Error ? e.message : String(e);
    throw e;
  } finally {
    loading.value = false;
  }
}
```

## Live lobby listing (LobbyRoom)

Primary path — `subscribeLobby` / `unsubscribeLobby` (see `work-with-lobby`):

```ts
const lobby = await client.joinOrCreate(LOBBY_ROOM, {
  filter: { name: TOURIST_ROOM },
});
lobby.onMessage('rooms', (rooms) => { this.rooms = rooms ?? []; });
lobby.onMessage('+', ([roomId, room]) => { /* upsert */ });
lobby.onMessage('-', (roomId) => { /* remove */ });
```

- Room constants: `LOBBY_ROOM = 'lobby'`, `TOURIST_ROOM = 'tourist'`.
- `listing` flag during subscribe connect; clear in `finally`.
- LobbyPage: `onMounted` → `subscribeLobby`; `onUnmounted` → `unsubscribeLobby`. **No** `setInterval` poll.
- After successful tourist connect, `_enterRoom` calls `unsubscribeLobby` (failed enter keeps lobby live).

### HTTP fallback (unused by LobbyPage)

`getAvailableRooms` was **removed** in Colyseus SDK 0.16+. `refreshRooms` still exists as unused HTTP fallback:

```ts
const { data } = await client.http.get(`/rooms/${TOURIST_ROOM}`);
this.rooms = (data ?? []) as RoomAvailable<GameRoomMeta>[];
```

Do **not** wire LobbyPage back to HTTP poll. Coordinate listing-shape / server registration with `../happy-tourist-server` (`lobby` + `tourist` + `.enableRealtimeListing()`).

## Rooms: create / join / leave

Room type constants:

```ts
export const TOURIST_ROOM = 'tourist';
export const LOBBY_ROOM = 'lobby';
```

| Intent | Store | SDK |
|--------|-------|-----|
| New room | `createGame(options?)` | `client.create(TOURIST_ROOM, options)` |
| Join by id | `joinGame(roomId, options?)` | `client.joinById(roomId, options)` |
| Join or create | `joinGame()` (no id) | `client.joinOrCreate(TOURIST_ROOM, options)` |
| Leave | `leaveGame()` | `room.leave()` (errors swallowed if already closed) |

All connect paths go through `_enterRoom`:

1. Set `status = 'connecting'`, clear `error`.
2. `await _leaveTouristRoom()` to detach any previous tourist room (lobby stays live during the attempt).
3. `await connect()`, then `unsubscribeLobby()` on success, then `_attachRoom(room)`.
4. On failure: `status = 'idle'`, set `error`, rethrow (lobby subscription remains).

### State sync (`_attachRoom`)

**Today** — mirror seating from schema. Move messages / turn fields deferred until rules land.

| Field | Meaning |
|-------|---------|
| `started` | Fourth seat assigned → `true`; drives Pinia `status` `playing` / `waiting` |
| `seats` Map | Key = `sessionId` → `touristId`, `side`, `row`, `col` |
| `sessionId` | From `room.sessionId` — for `mySeat` / strip |

Wire once in the store:

```ts
room.onStateChange((state) => {
  const s = state as TouristRoomState;
  this.sessionId = room.sessionId;
  this.started = Boolean(s.started);
  const next: GameSeat[] = [];
  s.seats?.forEach((seat, sessionId) => {
    next.push({
      sessionId,
      touristId: Number(seat.touristId),
      side: String(seat.side),
      row: Number(seat.row),
      col: Number(seat.col),
    });
  });
  this.seats = next;
  this.status = this.started ? 'playing' : 'waiting';
});
room.onError((_code, message) => { this.error = message || 'Room error'; });
room.onLeave(() => { this._resetRoomState(); });
```

`GamePage` may call `joinGame(roomId)` again if Pinia lost the room after refresh; failed rejoin → navigate to lobby. Board tile geometry is a **client constant** (`work-with-game-board`); seats come from sync.

## Messages: game actions

**None today** for seating (schema sync only). When move rules land, add `room.send(...)` helpers in the game store with server `onMessage` lockstep — do not treat legacy draughts `move` `{ from, to }` as current product canon.

## Loading and errors

| Flag | Where | Use |
|------|-------|-----|
| `auth.loading` | setup store ref | Auth form submit |
| `auth.error` | setup store ref | Login `q-banner` |
| `game.listing` | options store | Lobby subscribe connect window |
| `game.error` | options store | Lobby / game `q-banner` |
| `game.status` | options store | connecting / waiting / playing / finished / idle |

- Prefer `try/catch/finally` so loading flags never stick.
- No global Quasar loading overlay for Colyseus I/O today — keep store/page-local flags.
- Do not add empty `catch` blocks “just in case”; catch when the store or page must show `error` (or swallow known-closed `room.leave`).

## Patterns

### Live lobby subscribe

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
    lobby.onMessage('rooms', (rooms) => { this.rooms = rooms ?? []; });
    lobby.onMessage('+', ([roomId, room]) => { /* upsert into this.rooms */ });
    lobby.onMessage('-', (roomId) => {
      this.rooms = this.rooms.filter((r) => r.roomId !== roomId);
    });
  } catch (e) {
    this.error = e instanceof Error ? e.message : String(e);
    this.rooms = [];
    this.lobbyRoom = null;
  } finally {
    this.listing = false;
  }
}
```

From `LobbyPage`: `onMounted` → `subscribeLobby`; `onUnmounted` → `unsubscribeLobby`. No poll interval.

### Enter room from lobby

```ts
const room = await game.createGame();
// or: await game.joinGame(roomId)
// or: await game.joinGame()  // joinOrCreate
await router.push({ name: 'game', params: { roomId: room.roomId } });
```

Catch at the page only if you need extra UI beyond `game.error`.

### Board + seats on GamePage

```ts
// GamePage renders LAYOUT tiles + pieces from game.seats; no room.send for board UX today
await game.leaveGame();
await router.push({ name: 'lobby' });
```

Do not invent client-only move protocols; wait for product rules + server lockstep.

### Auth form submit

```ts
await auth.login(email, password);
await router.push({ name: 'lobby' });
```

Show `auth.error` in a `q-banner`. Router already blocks until `whenReady()`.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| `client.getAvailableRooms('tourist')` or LobbyPage HTTP poll | Live LobbyRoom `subscribeLobby`; HTTP only as unused `refreshRooms` fallback |
| `import … from '@colyseus/auth'` in the SPA | Use `client.auth` from `@colyseus/sdk` |
| Calling `client.create` / `room.send` in a page | Add/extend actions on `useGameStore` |
| Second `new Client(...)` | Reuse singleton from `@/boot/colyseus` |
| Treating GamePage layout as authoritative rules | Board geometry is UI only; seating/rules live on the server |
| Inventing client-local seat assignment | Mirror `seats`/`started` from schema only |
| Hardcoding room name in pages | Use `TOURIST_ROOM` / `LOBBY_ROOM` from `stores/game` |
| Leaving `listing` / `loading` true after error | Always `finally` |
| Skipping `whenReady` in router | Await before `requiresAuth` / `guest` redirects |
| Inventing axios/BFF helpers | Stay on LobbyRoom + room messages (`client.http` only if needed) |
| Skipping `lint` / `typecheck` after Colyseus client changes | Run `npm run lint` / `typecheck` from client package root; fix failures |

## Checklist for a new or changed Colyseus call

1. Belongs in `stores/auth` or `stores/game` (not a page).
2. Uses shared `client` from `@/boot/colyseus`.
3. Lobby list: LobbyRoom subscribe; rooms: `create` / `joinById` / `joinOrCreate`; Game `room.send` only when rules exist. HTTP only as unused fallback.
4. Room types use `TOURIST_ROOM` / `LOBBY_ROOM`.
5. Loading flag cleared in `finally`; failures set store `error`.
6. Future state fields / message payloads match `../happy-tourist-server` (lockstep).
7. Pages only call store actions and bind store state.
8. Run `npm run lint` / `npm run typecheck` from the client package root (and `quasar dev` if needed for smoke); fix failures before claiming done.

## Related context

- Client overview: `AGENTS.md` in this repo (auth, lobby, game, env, deploy).
- Lobby details: `.agents/skills/client/work-with-lobby/SKILL.md`
- Sibling server: `../happy-tourist-server` — `lobby` + `tourist` + `.enableRealtimeListing()`.
