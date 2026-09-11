---
name: colyseus-client
description: >-
  Use when adding, changing, or reviewing Colyseus client I/O in the
  happy-tourist checkers SPA: Client singleton from src/boot/colyseus.ts,
  client.http lobby listing, room create/join/leave, room.send move messages,
  client.auth register/sign-in/sign-out, Pinia auth/game stores, or
  VITE_COLYSEUS_URL / VITE_API_URL env wiring.
---

# Colyseus Client

Use this skill when adding or changing realtime / HTTP I/O in the **checkers client** (`happy-tourist.github.io`).

Stack: Vue 3 Composition API / `<script setup>`, Quasar 2, Pinia 4, TypeScript, `@colyseus/sdk` 0.18.

Sibling server: `../happy-tourist-server`. Coordinate room name, state schema, and message protocol with that package.

There is **no** axios layer and **no** BFF. HTTP goes through `client.http`; gameplay goes through room WebSocket messages.

## Quick Reference

| Topic | Client pattern |
|-------|----------------|
| Client singleton | `export const client = new Client(import.meta.env.VITE_COLYSEUS_URL)` in `src/boot/colyseus.ts` |
| Prefer import | `import { client } from '@/boot/colyseus'` (also `$colyseus` on `globalProperties`) |
| Where I/O lives | Pinia stores only: `stores/auth.ts`, `stores/game.ts` |
| Pages | Call store actions; do not call `client.*` from pages/components |
| HTTP listing | `client.http.get('/rooms/checkers')` — **not** `getAvailableRooms` (removed in 0.16+) |
| Room name | `CHECKERS_ROOM = 'checkers'` in `stores/game.ts` |
| Connect | `client.create` / `joinById` / `joinOrCreate` via game store actions |
| Moves | `room.send('move', { from, to })` via `sendMove` |
| Auth | `client.auth` — register / signIn / signOut / `onChange`; token key `colyseus-auth-token` |
| Env | `VITE_COLYSEUS_URL`, `VITE_API_URL` (typed in `env.d.ts`) |
| Errors | Store `error` string; pages show `q-banner` |
| Loading | Store flags (`auth.loading`, `game.listing`, `game.status`); reset in `finally` |
| Style | Prefer `async/await` + `try/catch` + `finally` |
| Commands | Agent proposes `npm run lint` / `typecheck` / `quasar dev`; user runs them and replies «готово» |

## Rules

| Do | Don't |
|----|--------|
| Import `client` from `@/boot/colyseus` | Create a second `Client`, or call `axios` / raw `fetch` for Colyseus |
| Keep Colyseus I/O inside Pinia (`auth`, `game`) | Scatter `client.http` / `client.create` / `room.send` across components |
| List rooms with `client.http.get(\`/rooms/${CHECKERS_ROOM}\`)` | Use `client.getAvailableRooms` (removed in SDK 0.16+) |
| Use `CHECKERS_ROOM` constant for room type | Hardcode `'checkers'` in multiple places or invent a new room name without the server |
| Enter rooms via `createGame` / `joinGame` (`_enterRoom`) | Duplicate connect + `onStateChange` wiring in pages |
| Send moves with `sendMove` → `room.send('move', { from, to })` | Invent other message names without coordinating with the server |
| Auth via `client.auth.*` in `stores/auth` | Import `@colyseus/auth` on the client (that package is **server-side**) |
| Catch into store `error`; clear loading in `finally` | Leave `listing` / `loading` stuck on reject |
| Pages → stores → `client` | Pages → `client` directly |
| Change contracts with `../happy-tourist-server` | Assume board/turn/status shape without checking server schema |

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
| Auth | `stores/auth.ts` (setup store) | `client.auth.registerWithEmailAndPassword`, `signInWithEmailAndPassword`, `signInAnonymously`, `signOut`, `onChange` |
| Lobby list | `stores/game.ts` → `refreshRooms` | `client.http.get('/rooms/checkers')` |
| Room lifecycle | `stores/game.ts` → `createGame` / `joinGame` / `leaveGame` | `client.create` / `joinById` / `joinOrCreate`, `room.leave` |
| Moves | `stores/game.ts` → `sendMove` | `room.send('move', { from, to })` |
| Live state | `stores/game.ts` → `_attachRoom` | `room.onStateChange`, `onError`, `onLeave` |

Allowed dependency direction: `pages` → `stores` / `boot` / `components`. Keep all `client.*` and `room.*` I/O in stores.

### Existing actions (do not invent parallel names)

| Store | Actions / API |
|-------|----------------|
| `auth` | `register`, `login`, `loginAnonymously`, `logout`, `whenReady` |
| `game` | `refreshRooms`, `createGame`, `joinGame`, `leaveGame`, `sendMove` |

Pages already wired:

| Page | Calls |
|------|-------|
| `LoginPage` | `auth.register` / `login` / `loginAnonymously` |
| `LobbyPage` | `game.refreshRooms` (poll), `createGame`, `joinGame`, `leaveGame`; `auth.logout` |
| `GamePage` | `game.joinGame(roomId)` on remount, `sendMove`, `leaveGame` |
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

## HTTP: room listing

`getAvailableRooms` was **removed** in Colyseus SDK 0.16+. Lobby uses the server HTTP route:

```ts
const { data } = await client.http.get(`/rooms/${CHECKERS_ROOM}`);
this.rooms = (data ?? []) as RoomAvailable<GameRoomMeta>[];
```

- Path: `/rooms/checkers` via `CHECKERS_ROOM`.
- Response body is the room list (wrapped as `{ data }` from `client.http`).
- `listing` flag + `error` string; always clear `listing` in `finally`.
- `LobbyPage` polls `refreshRooms` on an interval (~5s).

Coordinate any listing-shape changes with `../happy-tourist-server` (`GET /rooms/:roomName`).

## Rooms: create / join / leave

Room type constant:

```ts
export const CHECKERS_ROOM = 'checkers';
```

| Intent | Store | SDK |
|--------|-------|-----|
| New room | `createGame(options?)` | `client.create(CHECKERS_ROOM, options)` |
| Join by id | `joinGame(roomId, options?)` | `client.joinById(roomId, options)` |
| Join or create | `joinGame()` (no id) | `client.joinOrCreate(CHECKERS_ROOM, options)` |
| Leave | `leaveGame()` | `room.leave()` (errors swallowed if already closed) |

All connect paths go through `_enterRoom`:

1. Set `status = 'connecting'`, clear `error`.
2. `await leaveGame()` to detach any previous room.
3. `await connect()` then `_attachRoom(room)`.
4. On failure: `status = 'idle'`, set `error`, rethrow.

### State sync (`_attachRoom`)

Expected server state fields:

| Field | Meaning |
|-------|---------|
| `board` | `CellValue[][]` — `0` empty, `1` white, `2` black, `3` white king, `4` black king |
| `currentTurn` | `'white' \| 'black'` |
| `status` | `'waiting' \| 'playing' \| 'finished'` |
| `players[sessionId].color` | Local player's color |

Wire once in the store:

```ts
room.onStateChange((state) => { /* map board, turn, status, myColor */ });
room.onError((_code, message) => { this.error = message || 'Room error'; });
room.onLeave(() => { this._resetRoomState(); });
```

`GamePage` may call `joinGame(roomId)` again if Pinia lost the room after refresh; failed rejoin → navigate to lobby. Local move highlights on the page are **UI hints only** — board truth is server state.

## Messages: moves

```ts
sendMove(from: { row: number; col: number }, to: { row: number; col: number }) {
  if (!this.room) return;
  this.room.send('move', { from, to });
}
```

- Message type: `'move'`.
- Payload: `{ from: { row, col }, to: { row, col } }`.
- Gate UI with getter `canMove` (`status === 'playing'` and `currentTurn === myColor`).
- Do not add new message types without updating the server room handler.

## Loading and errors

| Flag | Where | Use |
|------|-------|-----|
| `auth.loading` | setup store ref | Auth form submit |
| `auth.error` | setup store ref | Login `q-banner` |
| `game.listing` | options store | Lobby refresh button / list |
| `game.error` | options store | Lobby / game `q-banner` |
| `game.status` | options store | connecting / waiting / playing / finished / idle |

- Prefer `try/catch/finally` so loading flags never stick.
- No global Quasar loading overlay for Colyseus I/O today — keep store/page-local flags.
- Do not add empty `catch` blocks “just in case”; catch when the store or page must show `error` (or swallow known-closed `room.leave`).

## Patterns

### Lobby list refresh (HTTP)

```ts
async refreshRooms() {
  this.listing = true;
  this.error = null;
  try {
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

From `LobbyPage`: `onMounted` → `refreshRooms` + `setInterval`; clear interval on unmount.

### Enter room from lobby

```ts
const room = await game.createGame();
// or: await game.joinGame(roomId)
// or: await game.joinGame()  // joinOrCreate
await router.push({ name: 'game', params: { roomId: room.roomId } });
```

Catch at the page only if you need extra UI beyond `game.error`.

### Send move from GamePage

```ts
if (!game.canMove) return;
game.sendMove(selected.value, { row, col });
```

Do not optimistically rewrite `game.board` as source of truth; wait for `onStateChange`.

### Auth form submit

```ts
await auth.login(email, password);
await router.push({ name: 'lobby' });
```

Show `auth.error` in a `q-banner`. Router already blocks until `whenReady()`.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| `client.getAvailableRooms('checkers')` | `client.http.get('/rooms/checkers')` |
| `import … from '@colyseus/auth'` in the SPA | Use `client.auth` from `@colyseus/sdk` |
| Calling `client.create` / `room.send` in a page | Add/extend actions on `useGameStore` |
| Second `new Client(...)` | Reuse singleton from `@/boot/colyseus` |
| Treating local highlight path as game rules | Server validates moves; client only sends `{ from, to }` |
| Hardcoding room name in pages | Use `CHECKERS_ROOM` from `stores/game` |
| Leaving `listing` / `loading` true after error | Always `finally` |
| Skipping `whenReady` in router | Await before `requiresAuth` / `guest` redirects |
| Inventing axios/BFF helpers | Stay on `client.http` + room messages |
| Agent running `lint` / `dev` without waiting | Propose command; wait for user «готово» |

## Checklist for a new or changed Colyseus call

1. Belongs in `stores/auth` or `stores/game` (not a page).
2. Uses shared `client` from `@/boot/colyseus`.
3. HTTP: `client.http.*` with server path; rooms: `create` / `joinById` / `joinOrCreate`; messages: `room.send`.
4. Room type uses `CHECKERS_ROOM` (`'checkers'`).
5. Loading flag cleared in `finally`; failures set store `error`.
6. State fields / message payload match `../happy-tourist-server`.
7. Pages only call store actions and bind store state.
8. Propose `npm run lint` / `npm run typecheck` (and `quasar dev` if needed); wait for «готово».

## Related context

- Client overview: `AGENTS.md` in this repo (auth, lobby, game, env, deploy).
- Skills path (temporary): `.agents/skills/client/` here until `happy-tourist-meta`.
- Sibling server: `../happy-tourist-server` — rooms, auth handlers, `GET /rooms/:roomName`.
