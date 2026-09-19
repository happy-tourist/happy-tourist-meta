---
name: colyseus-client
description: >-
  Use when adding, changing, or reviewing Colyseus client I/O in the
  happy-tourist tourist SPA: Client singleton from src/boot/colyseus.ts,
  LobbyRoom live listing (subscribeLobby), room create/join/leave (incl.
  grilleDensity), room.send move/rescue/push/returnFromFinish/peek/endTurn/say
  messages, private budgets/peekOpen/allJailWarning (infinite=peeks∞ only;
  holes not landable; holdingGrilleKeys), client.auth register/sign-in/sign-out,
  Pinia auth/game stores, or VITE_COLYSEUS_URL / VITE_API_URL env wiring.
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
| Game messages | `sendMove` → `move` (no turn advance); `sendRescue` / `sendPush` / `sendReturnFromFinish`; `sendPeek` / `sendPeekAnswer` / `sendEndTurn`; `sendReady` → `ready`; `sendSay` → `say`; listen `budgets` / `peekOpen` / `allJailWarning` / `say` |
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
| Keep room I/O in Pinia `game` store | Call `room.send` from GamePage — use `sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendSay` |
| Auth via `client.auth.*` in `stores/auth` | Import `@colyseus/auth` on the client (that package is **server-side**) |
| Catch into store `error`; clear loading in `finally` | Leave `listing` / `loading` stuck on reject |
| Pages → stores → `client` | Pages → `client` directly |
| Change contracts with `../happy-tourist-server` | Invent alternate move/turn shapes without lockstep |

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

Local defaults: `.env.development` → `localhost:2567`. Production: `.env.production` / CI vars → `api.happy-tourist.ru` (WSS/HTTPS).

## Where Colyseus I/O lives

| Area | Store | SDK surface |
|------|-------|-------------|
| Auth | `stores/auth.ts` (setup store) | `client.auth.registerWithEmailAndPassword`, `signInWithEmailAndPassword`, `signInAnonymously`, `signInWithProvider('google')`, `signOut`, `onChange` |
| Lobby list | `stores/game.ts` → `subscribeLobby` / `unsubscribeLobby` | `joinOrCreate('lobby', { filter })` + messages `rooms` / `+` / `-` |
| HTTP fallback | `stores/game.ts` → `refreshRooms` | `client.http.get('/rooms/tourist')` (unused by LobbyPage) |
| Room lifecycle | `stores/game.ts` → `createGame` / `joinGame` / `leaveGame` | `client.create` / `joinById` / `joinOrCreate`, `room.leave` |
| Game board UI | `pages/GamePage.vue` | Layout + pieces / strip / presence / grilles / budgets / peek / rescue / push / return icon / say; calls store sends (no direct `room.send`) |
| Live state | `stores/game.ts` → `_attachRoom` | `onStateChange` (`seats`/`phase`/`maxSeats`/`countdownRemaining`/`currentTurnSessionId`/`removedTaskKeys`/`holdingGrilleKeys`); `onMessage('say'|'budgets'|'peekOpen'|'allJailWarning')`; `onError`, `onLeave` |

Allowed dependency direction: `pages` → `stores` / `boot` / `components`. Keep all `client.*` and `room.*` I/O in stores.

### Existing actions (do not invent parallel names)

| Store | Actions / API |
|-------|----------------|
| `auth` | `register`, `login`, `loginAnonymously`, `loginWithGoogle`, `logout`, `whenReady` |
| `game` | `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `rejoinGame`, `leaveGame`, `sendMove`, `sendRescue`, `sendPush`, `sendReturnFromFinish`, `sendPeek`, `sendPeekAnswer`, `sendEndTurn`, `sendReady`, `sendSay` (`refreshRooms` HTTP unused) |

Pages already wired:

| Page | Calls |
|------|-------|
| `LoginPage` | `auth.register` / `login` / `loginAnonymously` / `loginWithGoogle` |
| `LobbyPage` | `subscribeLobby` / `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame`; `auth.logout` |
| `GamePage` | `game.rejoinGame(roomId)` on remount / soft-fail; pieces + grilles + presence + budgets + peek from store; move → `sendMove`; rescue/push/return → `sendRescue` / `sendPush` / `sendReturnFromFinish`; peek → `sendPeek` / `sendPeekAnswer`; end-turn → `sendEndTurn`; say → `sendSay`; `leaveGame` |
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
3. `await connect()`, then **immediately** `_attachRoom(room)` (register `onStateChange` / mirror before any other await), then `await unsubscribeLobby()` on success.
4. On failure: `status = 'idle'`, set `error`, rethrow (lobby subscription remains).

Do **not** `await unsubscribeLobby()` (or any other await) between `connect()` resolve and `_attachRoom` — the first `ROOM_STATE` can arrive in that gap and leave `seats` empty forever.

### State sync (`_attachRoom`)

Mirror seating + turn from schema.

| Field | Meaning |
|-------|---------|
| `phase` | `waiting` \| `countdown` \| `playing` — primary start gate; drives Pinia `status` |
| `maxSeats` / `countdownRemaining` | Capacity + authoritative countdown seconds |
| `started` | Legacy; client mirrors `phase === 'playing'` |
| `seats` Map | Key = `sessionId` → `touristId` + `pieces` (+ `finished`/`trapped`) + connectivity + `ready` + `finishPlace` |
| `currentTurnSessionId` | Synced whose turn; `""` if no seated / all finished → getter `isMyTurn` |
| `removedTaskKeys` | Synced `"r,c"` holes after **correct** peek only; **not landable** (stand-on-hole OK). Incorrect KEEP → no hole |
| `holdingGrilleKeys` | Synced revealed holding grille cells (`"r,c"`); overlay on GamePage |
| `sessionId` | From `room.sessionId` — for `mySeat` / strip / turn check |

Private (not schema) — wire in `_attachRoom`:

| Message | Pinia fields |
|---------|--------------|
| `budgets` `{ steps, peeks, infinite, peekedThisTurn }` | `steps` (always finite), `peeks`, `budgetsInfinite` (peeks∞ only), `peekedThisTurn` (legacy) |
| `peekOpen` `{ side, row, col, reward }` | `openPeek` |
| `allJailWarning` `{}` | `allJailWarning = true` (own seat modal) |

Wire once in the store:

```ts
room.onStateChange((state) => {
  const s = state as TouristRoomState;
  this.sessionId = room.sessionId;
  this.phase = parsePhase(s.phase);
  this.started = this.phase === 'playing';
  this.maxSeats = /* 2|3|4 from s.maxSeats */;
  this.countdownRemaining = /* floor of s.countdownRemaining */;
  this.currentTurnSessionId =
    typeof s.currentTurnSessionId === 'string' ? s.currentTurnSessionId : '';
  // seats[] incl. ready/connectivity/trapped …
  // removedTaskKeys[] / holdingGrilleKeys[] from sync
  this.status = this.phase === 'playing' ? 'playing' : 'waiting';
});

room.onMessage('budgets', (message) => { /* steps / peeks / budgetsInfinite=peeks∞ / peekedThisTurn legacy */ });
room.onMessage('peekOpen', (message) => { /* openPeek */ });
room.onMessage('allJailWarning', () => { this.allJailWarning = true; });
```

`GamePage` may call `rejoinGame(roomId)` if Pinia lost the room after refresh / soft-fail / browser reopen (`localStorage` reconnection token → `reconnect`, clear stale on fail → `joinById`); failed rejoin → navigate to lobby. Board tile geometry is a **client constant** (`work-with-game-board`); seats + phase + connectivity + turn + removed tiles + holding grilles come from sync; own budgets from private messages. Token details: `work-with-rooms`.

## Messages: game actions

| Direction | Name | Payload |
|-----------|------|---------|
| Client → server | `move` | `{ side, row, col }` via `sendMove` when playing + `isMyTurn` (spends step; **no** turn advance; may trap) |
| Client → server | `rescue` | `{ side }` via `sendRescue` |
| Client → server | `push` | `{ pusherSide, targetSessionId, targetSide, row, col }` via `sendPush` (−1 step; relocates target only; no turn advance) |
| Client → server | `returnFromFinish` | `{ side, row, col }` via `sendReturnFromFinish` |
| Client → server | `peek` | `{ side }` via `sendPeek` (reject trapped) |
| Client → server | `peekAnswer` | `{ correct }` via `sendPeekAnswer` |
| Client → server | `endTurn` | empty via `sendEndTurn` |
| Client → server | `ready` | empty via `sendReady` |
| Client → server | `say` | `{ presetId }` via `sendSay` |
| Server → owner | `budgets` | `{ steps, peeks, infinite, peekedThisTurn }` |
| Server → owner | `peekOpen` | `{ side, row, col, reward: 1\|2\|3 }` |
| Server → owner | `allJailWarning` | `{}` |
| Server → clients | `say` | `{ sessionId, presetId, at }` → `sayEvents` (incl. readiness preset from ready) |

```ts
sendMove(side: string, row: number, col: number): boolean {
  if (!this.room || this.phase !== 'playing' || !this.isMyTurn) return false;
  this.room.send('move', { side, row, col });
  return true;
}

sendRescue(side: string): boolean {
  if (!this.room || this.phase !== 'playing' || !this.isMyTurn) return false;
  this.room.send('rescue', { side });
  return true;
}

sendPush(
  pusherSide: string,
  targetSessionId: string,
  targetSide: string,
  row: number,
  col: number,
): boolean {
  if (!this.room || this.phase !== 'playing' || !this.isMyTurn) return false;
  this.room.send('push', { pusherSide, targetSessionId, targetSide, row, col });
  return true;
}

sendReturnFromFinish(side: string, row: number, col: number): boolean {
  if (!this.room || this.phase !== 'playing' || !this.isMyTurn) return false;
  this.room.send('returnFromFinish', { side, row, col });
  return true;
}

sendPeek(side: string): boolean {
  if (!this.room || this.phase !== 'playing' || !this.isMyTurn) return false;
  this.room.send('peek', { side });
  return true;
}

sendPeekAnswer(correct: boolean): boolean {
  if (!this.room || typeof correct !== 'boolean') return false;
  this.room.send('peekAnswer', { correct });
  this.openPeek = null;
  return true;
}

sendEndTurn(): boolean {
  if (!this.room || !this.canSendEndTurn) return false;
  this.room.send('endTurn');
  return true;
}

sendReady(): boolean {
  if (!this.room || !this.canSendReady) return false;
  this.room.send('ready');
  return true;
}

sendSay(presetId: SayPresetId): boolean {
  // whitelist hello|luck only (block ready); seated + connected; max 3 live
  this.room.send('say', { presetId });
  return true;
}
```

Do **not** use legacy draughts `{ from, to }`. Pages must not call `room.send` directly. GamePage may start travel animation only when `sendMove` / `sendPush` returns `true`. Peek / rescue / push / return / end-turn / say / ready UI uses store actions only (`work-with-game-board`).

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
// GamePage: LAYOUT + pieces + presence + budgets + peek + say; remount → rejoinGame(roomId)
// Own turn: select → red targets → game.sendMove(side, row, col); eye → sendPeek / sendPeekAnswer
// Multi: skip_next on own avatar (aria game.endTurn) → game.sendEndTurn(); solo hides end-turn
// Own online marker: picker → game.sendSay('hello'|'luck'); bubbles from game.sayEvents
await game.leaveGame(); // consented — clears tourist reconnect token
await router.push({ name: 'lobby' });
```

Do not call `room.send` from the page; keep move/peek/end-turn/say protocol lockstep with server.

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
| Treating GamePage layout as authoritative rules | Board geometry is UI only; seating/turn/moves live on the server |
| Inventing client-local seat assignment | Mirror `seats`/`phase`/`maxSeats`/`currentTurnSessionId` from schema only |
| Hardcoding room name in pages | Use `TOURIST_ROOM` / `LOBBY_ROOM` from `stores/game` |
| Leaving `listing` / `loading` true after error | Always `finally` |
| Skipping `whenReady` in router | Await before `requiresAuth` / `guest` redirects |
| Inventing axios/BFF helpers | Stay on LobbyRoom + room messages (`client.http` only if needed) |
| Calling `room.send('move'|'rescue'|'push'|'returnFromFinish'|'peek'|'endTurn'|'say', …)` from GamePage | Use `game.sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendSay` only |
| Skipping `lint` / `typecheck` after Colyseus client changes | Run `npm run lint` / `typecheck` from client package root; fix failures |

## Checklist for a new or changed Colyseus call

1. Belongs in `stores/auth` or `stores/game` (not a page).
2. Uses shared `client` from `@/boot/colyseus`.
3. Lobby list: LobbyRoom subscribe; rooms: `create({ maxSeats, grilleDensity })` / `joinById` (no Play shortcut); Game `sendMove` → `move`; `sendRescue` / `sendPush` / `sendReturnFromFinish`; `sendPeek` / `sendPeekAnswer` / `sendEndTurn`; `sendReady` → `ready`; `sendSay` → `say` + `onMessage('say'|'budgets'|'peekOpen'|'allJailWarning')`. HTTP only as unused fallback.
4. Room types use `TOURIST_ROOM` / `LOBBY_ROOM`.
5. Loading flag cleared in `finally`; failures set store `error`.
6. State fields / message payloads match `../happy-tourist-server` (lockstep), including `currentTurnSessionId` / `removedTaskKeys` / `holdingGrilleKeys` / piece `trapped` / private budgets.
7. Pages only call store actions and bind store state.
8. Run `npm run lint` / `npm run typecheck` from the client package root (and `quasar dev` if needed for smoke); fix failures before claiming done.

## Related context

- Client overview: `AGENTS.md` in this repo (auth, lobby, game, env, deploy).
- Lobby details: `.agents/skills/client/work-with-lobby/SKILL.md`
- Sibling server: `../happy-tourist-server` — `lobby` + `tourist` + `.enableRealtimeListing()`.
