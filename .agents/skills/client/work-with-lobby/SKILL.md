---
name: work-with-lobby
description: >-
  Guides live lobby via LobbyRoom subscribe/unsubscribe, quiet resubscribe
  (no reconnect hold), leave-before-enter tourist, create modal
  (mapId/packId/taskSetIds + maxSeats 1…map.players + grille/catapult density)
  / join-by-id (no Play shortcut), and lobby room card listing pointer
  (`LobbyRoomCardTile` + `taskCount` — topic listing-cards.md). Use when
  changing LobbyPage, LobbyRoomCardTile, game.subscribeLobby /
  unsubscribeLobby / createGame / joinGame, LOBBY_ROOM / TOURIST_ROOM, lobby
  loading flags, game.error + maps/content pickers, or lobby→game navigation
  (section links / logout live in App header — not LobbyPage).
---

# Work With Lobby

Use this skill for the **lobby** in the happy-tourist tourist client
(`happy-tourist.github.io`): live room list via built-in Colyseus `LobbyRoom`,
create (map + seats ≤ map.players + pack/task sets + grilleDensity +
catapultDensity) / join-by-id, then enter the game route. **No** primary
«Играть» / `joinOrCreate` shortcut. Packs / Maps / Support / staff «Модерация»
/ account / session logout live in the **shared App header** (SC-BRAND-11…15)
— LobbyPage MUST NOT duplicate that toolbar.

Stack: Vue 3 `<script setup>`, Quasar 2, Pinia `useGameStore` /
`useAuthStore`, `@colyseus/sdk` 0.18. Sibling server: `../happy-tourist-server`.

**Lobby ≠ tourist reconnect:** listing is fire-and-forget (design D7 /
SC-LOBBY-08). Do **not** persist a lobby reconnection token; do **not** expect
server `allowReconnection` on LobbyRoom. Tourist grace / `localStorage` token
live in `work-with-rooms`.

## Specialized Topics

Read the matching file when the change involves that area (do not load every
file at once):

| Topic | File |
|-------|------|
| Room **card** listing chrome (`LobbyRoomCardTile`, `--pack-card-*`, seats/tourists metric rows, status, set rows + `taskCount`, outline join) | [listing-cards.md](listing-cards.md) |

## Quick Reference

| Topic | Pattern |
|-------|---------|
| Page | `src/pages/LobbyPage.vue` — route `/lobby`, `meta.requiresAuth`; greeting + room list only (no page-local section/account/logout toolbar) |
| Store | `src/stores/game.ts` — `rooms`, `lobbyRoom`, `lobbyWanted`, `listing`, `error`, `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame` |
| Room names | `TOURIST_ROOM = 'tourist'`; `LOBBY_ROOM = 'lobby'` |
| Live list | `subscribeLobby` → `joinOrCreate('lobby', { filter: { name: TOURIST_ROOM } })` + handlers `rooms` / `+` / `-` |
| SDK auto-reconnect | After join: `lobby.reconnection.enabled = false` (avoid reservation churn) |
| Drop while on Lobby | Clear `lobbyRoom`; if `lobbyWanted` → `_quietResubscribeLobby` (no user-facing reservation text) |
| Leave lobby | `unsubscribeLobby` after successful tourist connect (`_enterRoom`); also on LobbyPage unmount and `leaveGame` / logout |
| Create | Modal: **map** picker (option + **selected-item** mini `MapGridPreview` — SC-LOBBY-31; players × tourists in option caption only — SC-LOBBY-29) + **maxSeats** radios `1…map.players` (default `min(2, map.players)` — SC-LOBBY-28) + **pack** + multi-check published sets (exactly one → auto-check — SC-LOBBY-30; labels `taskSetOption` / ordinal `taskSetLabel` **without** set author — SC-LOBBY-24) + **grille**/ **catapult** density few/medium/many (default medium; server seeds **12/22/35%**) → `createGame({ mapId, packId, taskSetIds, maxSeats, grilleDensity, catapultDensity })` |
| Join by id | Card body / outline «Войти» → `joinGame(roomId)` → `client.joinById(roomId)` — card chrome in [listing-cards.md](listing-cards.md) |
| Listing card | `LobbyRoomCardTile` in `.pack-card-grid` (SC-LOBBY-25/26/33/34/35/36; metric rows = set-row rhythm) — details in [listing-cards.md](listing-cards.md) |
| Play shortcut | **Removed** — do not restore «Играть» / bare `joinOrCreate` without product request |
| After enter | `router.push({ name: 'game', params: { roomId } })` |
| Loading | Store `listing` during subscribe connect; page refs `creating`, `joining`, `pickersLoading` |
| Errors | `game.error` + `q-banner` for **real** subscribe fail (SC-LOBBY-07); create pickers surface `maps.error` / `content.error`; filter lobby reconnect / `seat reservation expired` noise (SC-LOBBY-08) |
| Logout | Session logout is in **App header** (`onSessionLogout`); LobbyPage MUST NOT own a logout control |
| I/O boundary | Pages call store actions only; Colyseus stays in Pinia |
| HTTP fallback | `refreshRooms` → `client.http.get('/rooms/tourist')` exists but **LobbyPage must not poll it** |

## Rules

| Do | Don't |
|----|--------|
| Subscribe with `joinOrCreate('lobby', { filter: { name: TOURIST_ROOM } })` | Poll `setInterval` + HTTP `GET /rooms/tourist` from LobbyPage |
| Set `lobby.reconnection.enabled = false` after lobby join | Persist lobby `reconnectionToken` in `sessionStorage` / `localStorage` |
| Quiet-resubscribe on drop while `lobbyWanted` | Surface `seat reservation expired` / `FAILED_TO_RECONNECT` as listing `error` |
| Mount → `subscribeLobby`; unmount → `unsubscribeLobby` | Leave lobby WS open after navigate to game or login |
| Call `unsubscribeLobby` after successful `tourist` connect (`_enterRoom`); keep lobby on failed enter | Keep lobby + tourist sockets both live on GamePage |
| Create modal → `createGame({ mapId, packId, taskSetIds, maxSeats, grilleDensity, catapultDensity })`; list click → `joinGame(roomId)` | Invent parallel enter helpers; restore «Играть» / bare `joinOrCreate`; force maxSeats = map.players without picker |
| Navigate to `/game/:roomId` only after a successful enter | Stay on lobby with a live room and no route change |
| Bind listing loading / `creating` / `joining` | Leave buttons clickable during connect |
| Join busy-lock: early-return if `joining`; disable cards / non-clickable while joining (SC-LOBBY-19) | Allow double-join from card + button |
| Render listing as `LobbyRoomCardTile` grid (see [listing-cards.md](listing-cards.md)) | Revive dense `q-list`/`q-item` rows or primary-filled join as sole chrome |
| Clear `rooms = []` on subscribe start / unsubscribe / leave; keep `listing` until fresh `rooms` snapshot (SC-LOBBY-20) | Flash stale rooms after leave/resubscribe |
| Show `game.error` with `q-banner` for real listing failure | Duplicate a second error channel; treat transient reconnect noise as SC-LOBBY-07 |
| Logout via App header (`leaveGame` → `auth.logout` → login); keep Lobby free of section/account/logout chrome | Call `client.auth.signOut` from LobbyPage; revive Lobby-only Packs/Maps/Support/logout toolbar |
| Keep room type / metadata in sync with `../happy-tourist-server` | Change `TOURIST_ROOM` / `LOBBY_ROOM` without the server; add LobbyRoom grace on server |

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Page | `src/pages/LobbyPage.vue` | Subscribe lifecycle, create modal, join; card grid → [listing-cards.md](listing-cards.md); sections/logout → App header |
| Tile | `src/components/LobbyRoomCardTile.vue` | Listing card chrome — [listing-cards.md](listing-cards.md) |
| Store | `src/stores/game.ts` | LobbyRoom subscribe + room enter / leave; `GameRoomMeta.taskSetLabels` include `taskCount` |
| Maps / content | `src/stores/maps.ts` / `src/stores/content.ts` | In-catalog maps + published packs/sets for create pickers |
| Auth | `src/stores/auth.ts` | `displayName`, `logout` |
| Client | `src/boot/colyseus.ts` | Shared `Client` (`VITE_COLYSEUS_URL`) |
| Route | `src/router/routes.ts` | `/lobby` → name `lobby`; `/game/:roomId` → name `game` |

## Live list — `subscribeLobby` / `unsubscribeLobby`

```ts
async subscribeLobby() {
  await this.unsubscribeLobby();
  this.lobbyWanted = true;
  this.listing = true;
  this.rooms = []; // SC-LOBBY-20 — no stale list until snapshot
  this.error = null;

  try {
    this._attachLobbyRoom(await this._joinLobbyRoom());
  } catch (e) {
    const msg = e instanceof Error ? e.message : String(e);
    if (isLobbyReconnectNoise(msg)) {
      // One quiet retry; still-noise → generic "Список комнат недоступен".
    } else {
      this.error = msg; // SC-LOBBY-07
      this.rooms = [];
      this.lobbyRoom = null;
    }
  } finally {
    this.listing = false;
  }
}

async _joinLobbyRoom() {
  const lobby = await client.joinOrCreate(LOBBY_ROOM, {
    filter: { name: TOURIST_ROOM },
  });
  lobby.reconnection.enabled = false; // D7
  return lobby;
}

async unsubscribeLobby() {
  this.lobbyWanted = false;
  this.rooms = []; // SC-LOBBY-20
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

### Quiet drop / resubscribe (D7 / SC-LOBBY-08)

- `lobbyWanted` — `true` while LobbyPage wants a live list; cleared by `unsubscribeLobby`.
- `lobby.onLeave` → null `lobbyRoom`; if `lobbyWanted` → `_quietResubscribeLobby`.
- `isLobbyReconnectNoise(message)` filters: `seat reservation expired`,
  `failed_to_reconnect`, `reconnection failed`, `reconnection token`
  (case-insensitive).
- Filter keeps only `tourist` rooms. Server: `lobby: defineRoom(LobbyRoom)` +
  `tourist: …enableRealtimeListing()`; **no** LobbyRoom `allowReconnection`.
- `refreshRooms` HTTP remains unused fallback — do not poll from LobbyPage.

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

`_enterRoom` detaches any prior tourist room via `_leaveTouristRoom()` (lobby
stays subscribed during the attempt), then `connect()`, then
`unsubscribeLobby()` **only on success** (SC-LOBBY-05). Failed enter keeps the
live list. GamePage has only the `tourist` socket. Logout / leave uses
`leaveGame()` → `unsubscribeLobby` + leave tourist (tourist reconnect token —
see `work-with-rooms`).

## Enter flows

| UI | Page handler | Store | SDK |
|----|--------------|-------|-----|
| «Создать игру» → modal confirm | `onConfirmCreate` | `createGame({ mapId, packId, taskSetIds, maxSeats, grilleDensity, catapultDensity })` | `create(TOURIST_ROOM, { … })` |
| Card / outline «Войти» | `onJoin(roomId)` | `joinGame(roomId)` | `joinById(roomId)` |

All go through `_enterRoom` → navigate `push({ name: 'game', params: { roomId } })`.
Page `catch` empty on purpose — failure sets `game.error`.

## Loading flags

| Flag | Where | Covers |
|------|-------|--------|
| `game.listing` | store | `subscribeLobby` connect window |
| `joining` | LobbyPage `ref` | join-by-id |
| `creating` | LobbyPage `ref` | Create modal confirm |
| `pickersLoading` | LobbyPage `ref` | maps/packs list load for create modal |

Reset page refs in `finally`.

## Errors And Logout

- **SC-LOBBY-07:** primary subscribe fail, or listing still unavailable after
  quiet resubscribe with a non-noise error → `game.error` → `q-banner`.
- **SC-LOBBY-08:** transient drop / reservation noise → quiet clear + resubscribe;
  do **not** put that text in `game.error`.
- Create pickers: `maps.error` / `content.error` (+ i18n); no third error channel.
- Logout: `await game.leaveGame()` → `await auth.logout()` → login. Listing
  error must not block logout.

## Patterns

Create: validate map/pack/≥1 set/`maxSeats` → `createGame({ mapId, packId, taskSetIds, maxSeats, grilleDensity, catapultDensity })` → `push({ name: 'game', params: { roomId } })` (defaults: maxSeats `min(2, map.players)` SC-LOBBY-28; densities `medium`; single published set auto-check SC-LOBBY-30). Join: `joinGame(roomId)` → same navigate. Bind `creating`/`joining` in `finally`. No `client.*` in the page.

## Checklist

1. LobbyRoom subscribe (not HTTP poll); `reconnection.enabled = false`; quiet resubscribe + noise filter.
2. Mount/unmount/successful enter unsubscribe; failed enter keeps list; enter → `/game/:roomId`.
3. Create/join via store only; cards per [listing-cards.md](listing-cards.md); metadata `taskCount` SC-LOBBY-32.
4. SC-LOBBY-07 only for real listing fail; run client lint / typecheck / Lobby* vitest.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Poll HTTP / persist lobby token / surface reservation noise | Live subscribe + D7 quiet resubscribe |
| Lobby WS on GamePage / Play shortcut / `client.*` in page | `unsubscribeLobby` on success; store actions only |
| Dense list / author on card / ignore `taskCount` | [listing-cards.md](listing-cards.md) |

## Related

- Card chrome: [listing-cards.md](listing-cards.md) + `work-with-styles/pack-cards.md`
- Colyseus I/O / tourist reconnect / auth: `colyseus-client`, `work-with-rooms`, `client-work-with-auth`
- Vitest: `work-with-test` (`LobbyRoomCardTile` + `LobbyCreateWire`)
- Server: `lobby` + `tourist` + `.enableRealtimeListing()` + listing `taskCount`
