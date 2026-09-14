---
name: client-work-with-errors
description: >-
  Use when adding, changing, reviewing, or debugging error handling in the
  happy-tourist Vue 3 client — Pinia auth/theme/game `error` strings, try/catch/finally,
  Colyseus `room.onError`, q-banner display (pages + App theme banner), leave/rejoin edge
  cases, or page-level catch that relies on store state.
---

# Work With Errors

Use this skill when working with request / realtime errors in Vue 3 `<script setup>`
pages, Pinia stores, and Colyseus client calls under this package (`happy-tourist.github.io`).

Stack context: Vue 3 Composition API, Pinia (`auth` / `theme` setup stores, `game` options store),
vue-router (hash), Quasar, `@colyseus/sdk` via `src/boot/colyseus`. Language is TypeScript.
Surface errors via store `error: string | null` and `q-banner` on pages (auth/game) or `App.vue` (theme).

There is **no** shared axios layer, **no** response interceptors, **no** Vuex
`GLOBAL_ERROR_*`, **no** Qrator dialogs, and **no** `SHOW_DIALOG` error channel.

## Core Model

Error handling is **store-local**, not a second toast pipeline:

- Auth, theme, and game I/O live in Pinia (`stores/auth`, `stores/theme`, `stores/game`). Pages call store
  actions; they do **not** catch Colyseus/`client` errors for display themselves.
- Failed actions set `error` to a string:
  `e instanceof Error ? e.message : String(e)`.
- Clear `error` at the start of a new attempt (`error = null` / `this.error = null`).
- Loading / listing flags clear in `finally` (auth `loading`, game `listing`).
- Pages bind `q-banner` to `auth.error` or `game.error`.
- Theme save failures bind `q-banner` to `theme.error` in `App.vue` (shared shell).
- Page `catch` blocks are empty (or only navigate) with a comment that the store
  already holds the message — do not duplicate toasts or dialogs.
- Room lifecycle errors also arrive via `room.onError` into `game.error`.

Do not invent Notify plugins / dialog interceptors. Prefer Pinia `error` + `q-banner`.

## Auth Store Pattern

`src/stores/auth.ts` (setup store). Every mutating action follows the same shape:

```ts
loading.value = true;
error.value = null;

try {
  await client.auth./* … */;
} catch (e) {
  error.value = e instanceof Error ? e.message : String(e);
  throw e;
} finally {
  loading.value = false;
}
```

Applies to: `register`, `login`, `loginAnonymously`, `loginWithGoogle`, `logout`.

- Re-throw after setting `error` so the page can skip success navigation.
- `whenReady()` / `client.auth.onChange` are not error channels — they only sync
  session and flip `ready`.

## Game Store Pattern

`src/stores/game.ts` (options store).

### Listing (`subscribeLobby`)

```ts
this.listing = true;
this.error = null;

try {
  const lobby = await client.joinOrCreate(LOBBY_ROOM, {
    filter: { name: TOURIST_ROOM },
  });
  this.lobbyRoom = lobby;
  lobby.onMessage('rooms', (rooms) => { this.rooms = rooms ?? []; });
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
```

Does **not** re-throw — live list stays on LobbyPage; banner shows the message.

### Enter room (`_enterRoom` → `createGame` / `joinGame`)

```ts
this.status = 'connecting';
this.error = null;

try {
  await this._leaveTouristRoom(); // keep lobby live during attempt
  const room = await connect();
  await this.unsubscribeLobby(); // SC-LOBBY-05 — only on success
  this._attachRoom(room);
  return room;
} catch (e) {
  this.status = 'idle';
  this.error = e instanceof Error ? e.message : String(e);
  throw e; // lobby subscription still active on LobbyPage
}
```

Re-throws so Lobby / GamePage can avoid navigation or redirect on failure. Do not call `subscribeLobby` in this `catch` — it would clear `error`.

### Room listener (`_attachRoom`)

```ts
room.onError((_code, message) => {
  this.error = message || 'Room error';
});
```

Realtime room failures update the same `error` field the banners already bind.

### Leave (`leaveGame`)

```ts
const room = this.room;
this._resetRoomState();

if (room) {
  try {
    await room.leave();
  } catch {
    // room may already be closed
  }
}
```

Swallow leave errors on purpose. Do not set `error` here.

### Game messages (later)

No Game `room.send` today (seating syncs via schema; board non-interactive). When move rules land, early-return if
`!this.room`; illegal actions are server-side — do not invent local move UX
error handling unless product requirements change.

## Display Surfaces

### `q-banner` on pages

| Page | Binding | Notes |
|------|---------|--------|
| `LoginPage` | `auth.error` | Inside the form; cleared on mode toggle |
| `LobbyPage` | `game.error` | Above room list |
| `GamePage` | `game.error` | Above the board |

Typical markup:

```vue
<q-banner v-if="auth.error" dense class="bg-negative text-white">
  {{ auth.error }}
</q-banner>
```

```vue
<q-banner v-if="game.error" class="bg-negative text-white q-mb-md" dense rounded>
  {{ game.error }}
</q-banner>
```

```vue
<!-- App.vue — theme preference save failure -->
<q-banner v-if="theme.error" dense rounded class="bg-negative text-white q-ma-md">
  {{ theme.error }}
  <template #action>
    <q-btn flat dense label="OK" @click="theme.error = null" />
  </template>
</q-banner>
```

Use Quasar `bg-negative text-white`; keep dense. Do not add Notify plugins or
`$q.dialog` for these flows. Auth/game banners stay on pages; theme save errors
use the shared `App.vue` banner only.

### Clearing

- Stores clear `error` when starting a new attempt.
- Login clears on register/login toggle: `auth.error = null` in `toggleMode`.
- Do not leave stale banners after a successful action that did not clear `error`
  (enter/list/auth actions already clear at start).

## When Store Catch Vs Page Catch

| Situation | Pattern |
|-----------|---------|
| Auth register / login / anonymous / Google / logout | Store sets `error`, re-throws; page `catch { /* error already in store */ }` and skips redirect |
| Lobby create / join / play | Store sets `error`, re-throws; page `catch` + local `creating`/`joining` in `finally` |
| Lobby room list subscribe | Store catch sets `error`, empties `rooms`, **no** re-throw |
| GamePage mount rejoin | `joinGame(roomId)` fail → `router.replace({ name: 'lobby' })` (error may still be in store for lobby banner) |
| GamePage missing room and no `roomId` | Redirect lobby without setting a new error |
| Leave room (user or `_enterRoom` cleanup) | Swallow leave errors |
| Room `onError` while seated | Store only — banner on GamePage |
| Theme toggle / restore `GET`/`POST /api/theme` fail | `stores/theme` sets `error`; banner in `App.vue` |
| Best-effort side effect | Prefer store pattern above; avoid empty catch that hides failures without store `error` or intentional swallow |

## Patterns And Examples

### 1. Auth action (store) + page navigation

```ts
// stores/auth — login
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
```

```ts
// LoginPage — onSubmit
try {
  await auth.login(email.value, password.value);
  await goAfterLogin();
} catch {
  // error already in store
}
```

### 2. Lobby join with local button loading

```ts
joining.value = true;
try {
  await game.joinGame(roomId);
  await router.push({ name: 'game', params: { roomId } });
} catch {
  // error in store
} finally {
  joining.value = false;
}
```

Store owns the message; page owns button `loading` via `joining` / `creating`.

### 3. List refresh (no re-throw)

Use `subscribeLobby` as-is: catch → `error` + `rooms = []` → `listing = false` in
`finally`. Poll interval in LobbyPage should keep calling it.

### 4. GamePage rejoin failure → lobby

```ts
onMounted(async () => {
  const roomId = /* from route.params.roomId */;
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

Do not stay on `/game/:roomId` with an empty board after a failed rejoin.

### 5. Intentional swallow on leave

Only `leaveGame` (and similar “room already gone”) may use empty `catch` without
setting `error`. Document with a short comment.

## Colyseus / HTTP Notes

- Shared client: `src/boot/colyseus.ts` — `new Client(import.meta.env.VITE_COLYSEUS_URL)`.
- Auth: `client.auth.*` (SDK), not `@colyseus/auth` in the browser.
- Lobby HTTP: `client.http.get('/rooms/tourist')` — treat like any other promise;
  normalize with `e instanceof Error ? e.message : String(e)`.
- Keep Colyseus I/O in stores; pages should not call `client.*` for errors/display.

## Common Mistakes

- Adding axios interceptors, Vuex `GLOBAL_ERROR_*`, Qrator, or `SHOW_DIALOG` —
  none of that exists in this client.
- Showing errors only with `console.error` / Notify while leaving `auth.error` /
  `game.error` / `theme.error` unset — banners will stay empty.
- Catching in the page and setting a second local `error` ref that duplicates the store.
- Forgetting to clear `error` before a new attempt (stale banner).
- Skipping `finally` for `loading` / `listing` (or page `joining` / `creating`).
- Setting `game.error` inside `leaveGame` when the room is already closed.
- Swallowing join/auth failures without store `error` **and** without navigation —
  user sees no feedback.
- Staying on GamePage after failed rejoin instead of redirecting to lobby.
- Putting `client.create` / `joinById` / `http.get` / theme POST try/catch in a page
  instead of the owning store (`game` / `theme`).
- Showing theme save failures only on a page while leaving `theme.error` unset —
  use the shared `App.vue` banner.

## Key Files

| Path | Role |
|------|------|
| `src/stores/auth.ts` | Auth actions: `error` / `loading` / re-throw |
| `src/stores/theme.ts` | Theme toggle/save: `error` → `App.vue` banner |
| `src/stores/game.ts` | Rooms, `_enterRoom`, `room.onError`, `leaveGame` swallow |
| `src/App.vue` | Shared `theme.error` `q-banner` |
| `src/pages/LoginPage.vue` | `q-banner` + `auth.error`; clear on toggle |
| `src/pages/LobbyPage.vue` | `q-banner` + `game.error`; join/create catch |
| `src/pages/GamePage.vue` | `q-banner` + rejoin fail → lobby |
| `src/boot/colyseus.ts` | Shared `Client` instance |
| `src/router/index.ts` | Auth `whenReady` gate (not an error UI) |
