---
name: work-with-stores
description: >-
  Instructions for Pinia stores in the happy-tourist Vue 3 client:
  setup vs options defineStore, auth vs game vs theme ownership, local page
  state vs Pinia, Colyseus I/O in stores (budgets peeks∞ / finite steps,
  removed-task holes, grille trap/rescue/return, peek/end-turn), acceptHMRUpdate,
  and Quasar pinia entry. Use when adding, changing, reviewing, or debugging
  Pinia stores, shared game/auth/theme state, or page-to-store wiring.
---

# Work With Stores

Use this skill when deciding where state should live or when changing Pinia stores in the **happy-tourist client** (`happy-tourist.github.io`).

This app uses **Pinia 4** with **three domain stores** (`auth`, `theme`, `game`) plus Quasar’s Pinia entry. Pages use Composition API (`<script setup>`) and call `useAuthStore()` / `useThemeStore()` / `useGameStore()` directly — not Vuex `map*`.

Skills for this client live under `.agents/skills/client/`. Runtime paths below are relative to this repo root.

## Layout

```
src/stores/
  index.ts          # Quasar defineStore → createPinia() (app entry, not a domain store)
  auth.ts           # setup store (Composition API defineStore)
  theme.ts          # setup store — Quasar Dark preference (guest local / registered HTTP)
  game.ts           # options store
  example-store.ts  # Quasar scaffold counter — unused by login/lobby/game
```

Pinia is installed via Quasar store entry `src/stores/index.ts` (`createPinia()`). Domain stores import the Colyseus `client` from `@/boot/colyseus`. Prefer importing `use*Store` from `@/stores/auth` / `@/stores/theme` / `@/stores/game` in pages and router; keep Colyseus calls inside those stores.

### Store styles in this repo

| Store | Style | Why |
|-------|--------|-----|
| `auth` | **Setup** (`defineStore('auth', () => { … })`) | Refs + computed, `onChange` subscription, `whenReady` promise — fits Composition API |
| `theme` | **Setup** (`defineStore('theme', () => { … })`) | Dark preference + `syncFromAuthUser` / `toggle`; registered GET restore + POST save |
| `game` | **Options** (`defineStore('game', { state, getters, actions })`) | Clear room lifecycle, `this.*` mutations, private helpers `_enterRoom` / `_attachRoom` |
| `counter` (`example-store`) | Options | Scaffold only — do not extend for product features |

**When to choose setup vs options**

- Prefer **setup** when the store needs Vue composables heavily (`ref`/`computed`), long-lived subscriptions, or readiness promises (`auth`, `theme`).
- Prefer **options** when the domain is action-centric with shared mutable session state and imperative helpers (`game`).
- Do not convert an existing store style without a concrete reason. Match the neighbor store’s style when extending the same domain.

### Naming conventions

| Layer | Pattern | Examples |
|-------|---------|----------|
| Store id | kebab/camel short id | `'auth'`, `'theme'`, `'game'` |
| Composable | `use*Store` | `useAuthStore`, `useThemeStore`, `useGameStore` |
| State | camelCase | `user`, `roomId`, `status` |
| Getters | camelCase boolean/derived | `isAuthenticated`, `isInRoom` |
| Actions | verb / domain | `login`, `subscribeLobby`, `createGame`, `rejoinGame`, `leaveGame` |
| Internal helpers | `_` prefix (options) | `_enterRoom`, `_leaveTouristRoom`, `_attachRoom`, `_resetRoomState`, `_joinLobbyRoom`, `_quietResubscribeLobby` |
| Exported constants / types | beside the store | `TOURIST_ROOM`, `LOBBY_ROOM`, `AuthUser` |

## State Ownership

### Local page state

Keep state in the page when it belongs to one UI scenario and is not needed elsewhere.

Use local `ref()` / `reactive()` for:
- Form drafts and field values (login email/password/name).
- UI toggles (`isRegister`, `showPassword`).
- Page-local busy flags (`creating`, `joining`) that are not shared.
- Static board layout constants on `GamePage` (tile grid) — geometry is local UI, not Pinia.

Examples:
- `LoginPage.vue`: `email`, `password`, `displayName`, `isRegister`, `showPassword`.
- `LobbyPage.vue`: `creating`, `joining` (page spinners); room list and errors come from `game`.
- `GamePage.vue`: `LAYOUT` tiles + piece assets; presence layout / grace tick / `consentedLeaving`; `unfinishedBoardPieces` + disappearing finishers for board overlay; strip×4 from `mySeat` (+ finish icons); local `selectedSide` / `moveAnimating` / legal hints (exclude removed holes) / +N fall anim / place / peek / solo-peeks∞ / dual timer-vs-steps end modals; say picker open state (not Pinia).

### Pinia state

Use a store for shared domain data, realtime session, or anything the router/other pages must see across navigations.

| Store | Owns | Typical consumers |
|-------|------|-------------------|
| **auth** | `user`, `token`, `loading`, `error`, `ready`; `isAuthenticated`, `displayName`; register/login/anonymous/Google/`logout`/`whenReady` | `LoginPage`, router `beforeEach`, `LobbyPage` logout/header, `App.vue` theme sync |
| **theme** | Quasar Dark `preference`, `error`; async `syncFromAuthUser` (GET restore + generation + `clearStoredTheme` when unset; **no** `auth.user` replace after GET), `toggle` (guest `localStorage` `ht-theme`; registered `get` ≠ JWT-only, `post` on toggle may patch `user.theme`) | `App.vue` header toggle + stable auth identity watch |
| **game** | lobby `rooms`/`lobbyRoom`/`lobbyWanted`/`listing`; active `room`/`roomId`/`sessionId`; mirrored `seats` (`GameSeat`: `touristId` + `pieces[]` (+ `finished`/`trapped`) + connectivity + `ready` + `finishPlace` + `timeExpired`) / `phase` / `maxSeats` / `countdownRemaining` / legacy `started` / `currentTurnSessionId` / `turnUntil` / `turnBudgetSeconds` / `removedTaskKeys` / `holdingGrilleKeys`; private `steps`/`peeks`/`budgetsInfinite`/`peekedThisTurn`/`openPeek`/`allJailWarning` from `budgets`/`peekOpen`/`allJailWarning`; getters `mySeat`/`isSeated`/`isMyTurn`/`isPlaying`/`canSendReady`/`canSendEndTurn`/`unfinishedBoardPieces`/`isMySeatFinished`/`isMySeatTimeExpired`/`isSoloBudget`/`myFinishedStripSides`; helpers `isFinishedSeat`/`isFinishedPiece`/`isTimeExpiredSeat`/`isSoloBudgetSeconds`/`turnRemainingSeconds`; `sendMove` / `sendRescue` / `sendReturnFromFinish` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn`; `sendReady`; `sendSay` + ephemeral `sayEvents`; `status`, `error`; subscribe/unsubscribe / create(`maxSeats`+`grilleDensity`)/join/`rejoinGame`/leave; tourist token in `localStorage` | `LobbyPage`, `GamePage` |
| **counter** | scaffold only | none in product flow — ignore unless cleaning scaffold |

### Auth vs theme vs game ownership

- **auth** owns Colyseus Auth only (`client.auth.*`, token sync via `onChange`). It does not create rooms or call Dark/`GET|POST /api/theme`.
- **theme** owns chrome Dark preference and preference HTTP (`client.http.get('/api/theme')` on restore, `post` on toggle). Wired from `App.vue`; does not own auth session or rooms. See `work-with-styles`.
- **game** owns room listing, room lifecycle, tourist reconnect token, seat/turn/start/finish/timer/grille sync (`seats` + piece `finished`/`trapped` / seat `finishPlace` / `timeExpired` + `phase` / `maxSeats` / `countdownRemaining` / `currentTurnSessionId` / `turnUntil` / `turnBudgetSeconds` / `removedTaskKeys` / `holdingGrilleKeys` / `sessionId` from `onStateChange`; legacy `started` mirrors `phase === 'playing'`), private budgets (`onMessage('budgets')` → `steps`/`peeks`/`budgetsInfinite` = peeks∞ only/`peekedThisTurn` legacy; `onMessage('peekOpen')` → `openPeek`; `onMessage('allJailWarning')` → `allJailWarning`), `sendMove` → `room.send('move', …)` only when playing + `isMyTurn` + not finished + not time-expired + `steps > 0` (**does not** advance turn locally — server ends via endTurn/auto/timeout), `sendRescue` / `sendReturnFromFinish`, `sendPeek` / `sendPeekAnswer` / `sendEndTurn` (`canSendEndTurn` hides solo peeks∞), `sendReady` → `room.send('ready')`, and `sendSay` → `room.send('say', { presetId })` plus `onMessage('say')` → `sayEvents` (picker whitelist `hello`|`luck`; readiness preset arrives via ready broadcast; TTL 10s / max 3 live). Create options: `{ maxSeats?, grilleDensity?: 'few'|'medium'|'many' }`. It does not call `client.auth` or theme APIs. Do not put `side`/`row`/`col` on `GameSeat` itself — those live on each `GamePiece`. Selection/hints/`moveAnimating`/+N anim/say picker/place/peek/solo-peeks∞/all-jail/dual end modals stay page-local on `GamePage`.
- Cross-cutting: router awaits `useAuthStore().whenReady()` then enforces `requiresAuth` / `guest`. `App.vue` uses `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `theme.syncFromAuthUser` (GET restore; not JWT `user.theme`-only). Do **not** use `watch(() => [ready, id, anonymous])` (new array each run) or replace `auth.user` after GET — that storms `GET /api/theme` (SC-THEME-10). POST toggle may patch `auth.user.theme` because the watch does not depend on it. Game pages assume auth already passed.
- Room constants: `TOURIST_ROOM = 'tourist'`, `LOBBY_ROOM = 'lobby'`. Board tile geometry + presence + move/peek/grille chrome stay on `GamePage`.

### Decision checklist

Before adding state to Pinia, ask:
- Do more than one page/component need this data?
- Should it survive leaving the current page (auth session, active room)?
- Is it Colyseus I/O or synced room/auth domain state?

If no → keep it in local `ref()` on the page.

Before keeping state local, ask:
- Is it only used by this page (form draft, selection, one-off spinner)?
- Can it reset without affecting lobby/game/auth elsewhere?

If yes → keep it local.

## Core Patterns

### Keep Colyseus I/O in stores

Pages call store actions; stores call `client` / `room`.

```ts
// Do — in stores/game.ts
await client.joinOrCreate(LOBBY_ROOM, { filter: { name: TOURIST_ROOM } });
await client.create(TOURIST_ROOM, options);
this.room.send('move', { side, row, col }); // only from sendMove when playing + isMyTurn (no local turn advance)
this.room.send('peek', { side }); // sendPeek
this.room.send('peekAnswer', { correct }); // sendPeekAnswer
this.room.send('endTurn'); // sendEndTurn when canSendEndTurn
this.room.send('ready'); // only from sendReady when canSendReady
this.room.send('say', { presetId }); // only from sendSay (hello|luck — not ready)

// Do — in stores/auth.ts
await client.auth.signInWithEmailAndPassword(email, password);
client.auth.onChange((data) => { /* sync token/user/ready */ });
```

Do **not** scatter `client.create` / `client.auth.*` / `client.http` / `room.send` across many components. Import `client` from `@/boot/colyseus` inside stores (and boot), not from random widgets. Theme preference HTTP belongs in `stores/theme.ts` (see `work-with-styles`), not in pages.

### Setup store (`auth`)

```ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<AuthUser | null>(null);
  const token = ref<string | null>(null);
  const ready = ref(false);
  // …
  const isAuthenticated = computed(() => Boolean(token.value && user.value));

  client.auth.onChange((data) => {
    token.value = data.token ?? null;
    user.value = (data.user as AuthUser | null) ?? null;
    ready.value = true;
    // resolve whenReady promise once
  });

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

  return { user, token, /* … */, isAuthenticated, login, whenReady };
});
```

Notes:
- Catch into store `error` string; pages show `q-banner`.
- `whenReady()` gates the router until first `onChange`.
- Token key is SDK-managed (`colyseus-auth-token`); do not invent a parallel token store.

### Options store (`game`)

```ts
export const useGameStore = defineStore('game', {
  state: () => ({
    rooms: [],
    lobbyRoom: null,
    lobbyWanted: false,
    room: null,
    roomId: null,
    sessionId: null,
    phase: 'waiting',
    maxSeats: 2,
    countdownRemaining: 0,
    started: false, // legacy mirror of phase === 'playing'
    currentTurnSessionId: '',
    turnUntil: 0,
    turnBudgetSeconds: 0,
    seats: [],
    removedTaskKeys: [],
    steps: 0,
    peeks: 0,
    budgetsInfinite: false,
    peekedThisTurn: false,
    openPeek: null,
    status: 'idle',
    error: null,
    listing: false,
  }),
  getters: {
    isInRoom: (state) => Boolean(state.room),
    mySeat: (state) =>
      state.seats.find((s) => s.sessionId === state.sessionId) ?? null,
    isSeated: (state) =>
      Boolean(state.sessionId && state.seats.some((s) => s.sessionId === state.sessionId)),
    isMyTurn: (state) =>
      Boolean(
        state.sessionId &&
        state.currentTurnSessionId &&
        state.sessionId === state.currentTurnSessionId &&
        state.seats.some((s) => s.sessionId === state.sessionId),
      ),
    isPlaying: (state) => state.phase === 'playing',
    canSendReady: (state) => { /* waiting + ≥2 + under maxSeats + own !ready */ },
    canSendEndTurn: (state) => { /* playing + isMyTurn + !budgetsInfinite + !finished + !timeExpired */ },
  },
  actions: {
    async subscribeLobby() { /* lobbyWanted + joinOrCreate lobby + quiet resubscribe */ },
    async unsubscribeLobby() { /* lobbyWanted=false; leave lobbyRoom */ },
    async createGame(options: { maxSeats?: 2 | 3 | 4; grilleDensity?: 'few' | 'medium' | 'many' } = {}) {
      return this._enterRoom(() => client.create(TOURIST_ROOM, options));
    },
    async rejoinGame(roomId, options = {}) {
      // localStorage reconnect(token) → clear stale on fail → fallback joinById
    },
    sendMove(side, row, col) {
      if (
        !this.room ||
        this.phase !== 'playing' ||
        !this.isMyTurn ||
        this.isMySeatFinished ||
        this.isMySeatTimeExpired
      ) {
        return false;
      }
      this.room.send('move', { side, row, col });
      return true;
    },
    sendPeek(side) { /* playing + isMyTurn → room.send('peek', { side }) */ },
    sendPeekAnswer(correct) { /* room.send('peekAnswer', { correct }); openPeek = null */ },
    sendEndTurn() {
      if (!this.room || !this.canSendEndTurn) return false;
      this.room.send('endTurn');
      return true;
    },
    sendReady() {
      if (!this.room || !this.canSendReady) return false;
      this.room.send('ready');
      return true;
    },
    sendSay(presetId) {
      // whitelist hello|luck only (block ready); seated + connected; max 3 live
      this.room.send('say', { presetId });
      return true;
    },
  },
});
```

Notes:
- Live lobby listing uses `subscribeLobby` / LobbyRoom messages — not LobbyPage HTTP poll. `refreshRooms` HTTP remains unused fallback. Set `lobby.reconnection.enabled = false`; filter reservation/reconnect noise (see `work-with-lobby`).
- Mirror `seats` (incl. connectivity + `ready` + `finishPlace` + `timeExpired` + piece `trapped`) / `phase` / `maxSeats` / `countdownRemaining` / `currentTurnSessionId` / `turnUntil` / `turnBudgetSeconds` / `removedTaskKeys` / `holdingGrilleKeys` / `sessionId` in the store; listen `budgets` / `peekOpen` / `allJailWarning` privately. Keep tile geometry + presence + selection/hints + peek/rescue/return/end-turn chrome + say/timeout/place/solo/all-jail modals on `GamePage` (not Pinia). Ephemeral `sayEvents` stay in the store (room I/O).
- Persist tourist `reconnectionToken` in `localStorage` (`ht-tourist-reconnect`); clear on consented `leaveGame` / `_leaveTouristRoom` and after failed `reconnect`; keep on unexpected `onLeave`; cross-tab steal OK (see `work-with-rooms`).
- `leaveGame` unsubscribes lobby and swallows leave errors (room may already be closed). `GamePage` calls `rejoinGame(roomId)` on mount / soft-fail (reconnect → `joinById`).

### HMR: always `acceptHMRUpdate`

Every domain store ends with:

```ts
import { defineStore, acceptHMRUpdate } from 'pinia';

// … defineStore …

if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useAuthStore, import.meta.hot));
}
```

Use the same store composable passed to `defineStore`. Add this when creating a new store file.

### Quasar Pinia entry (`stores/index.ts`)

Do not put domain state here. Only `createPinia()` (+ optional plugins) and `PiniaCustomProperties` typing. New product stores are separate files under `src/stores/`.

## Using Stores In Pages / Router

Composition API only for product pages:

```ts
import { useAuthStore } from '@/stores/auth';
import { useGameStore } from '@/stores/game';

const auth = useAuthStore();
const game = useGameStore();

await auth.login(email.value, password.value);
await game.subscribeLobby();
```

Router: await `useAuthStore().whenReady()`, then honor `meta.requiresAuth` / `meta.guest`. Prefer store actions over importing `client` in the router.

Dependency direction: `pages` → `stores` / `boot` / `components`. Keep Colyseus I/O in Pinia.

## Adding A New Store Or Field

1. Prefer extending `auth` or `game` over a third domain store unless the concern is clearly separate.
2. Pick setup vs options deliberately (see table above); add `acceptHMRUpdate`.
3. Put defaults in setup `ref()` initial values or options `state()`.
4. Add getters for derived flags (`isAuthenticated`, `isInRoom`) instead of recomputing in every page.
5. Put async Colyseus/HTTP work in store actions; set `error` / loading flags there.
6. Wire pages with `use*Store()`; keep form drafts and board layout local.
7. Do not build product features on `example-store` / `counter`.

## Do / Don't

**Do**
- Keep Colyseus Auth and room I/O inside `auth` / `game`.
- Use local `ref` for form drafts, layout constants, and page-only spinners.
- Sync auth from `client.auth.onChange`; gate routes with `whenReady()`.
- Map only needed room fields from `onStateChange` (incl. `currentTurnSessionId` / `removedTaskKeys` / `holdingGrilleKeys` / piece `trapped`); keep `sendMove` / `sendRescue` / `sendReturnFromFinish` / `sendPeek` / `sendEndTurn` / `sendSay` lockstep with server `onMessage`.
- Add `acceptHMRUpdate` to every new store file.
- Coordinate room name / state schema / messages with `../happy-tourist-server`.

**Don't**
- Call `client.auth.*`, `client.create` / `joinById`, or `room.send` from random components / GamePage (use `sendMove` / `sendRescue` / `sendReturnFromFinish` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendSay`).
- Put tourist board tile geometry or selection/hints into Pinia without a cross-page need.
- Assume `sendMove` advances the turn — end-turn / auto / timeout do.
- Put domain state in `stores/index.ts` or grow the unused `counter` scaffold.
- Reintroduce legacy draughts `board` / `{ from, to }` as current product canon.
- Mix auth session concerns into `game` or room lifecycle into `auth`.
- Forget HMR `acceptHMRUpdate` on new stores.
- After theme GET, replace `auth.user` only to set `theme` — feeds the App
  restore watch and storms preference HTTP; keep applied theme in `theme`
  store; use a stable multi-source `watch([...sources])`.

## Common Mistakes

- Scattering Colyseus calls across pages instead of store actions.
- Adding Vuex-style modules/`mapState` — this client is Pinia + `<script setup>`.
- Putting login form fields into `auth` state.
- Putting cell selection / legal hints into `game` state (keep page-local on `GamePage`).
- Using `getAvailableRooms` or LobbyPage HTTP poll — use `subscribeLobby` (LobbyRoom); HTTP `refreshRooms` is unused fallback.
- Assuming `@colyseus/auth` is the client API — browser auth is `client.auth` from `@colyseus/sdk`.
- Skipping `whenReady()` and racing protected routes before token restore.
- Leaving room attach listeners only in a page so refresh/rejoin breaks — attach in `game._attachRoom`.
