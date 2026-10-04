---
name: work-with-stores
description: >-
  Instructions for Pinia stores in the happy-tourist Vue 3 client: setup vs
  options defineStore, auth/theme/game/support/content/maps ownership, local
  page state vs Pinia, Colyseus I/O in stores, acceptHMRUpdate. Core in SKILL.md;
  content packs in content.md (compose/tiles SC-PACK-256…263); draft lifecycle
  in pack-lifecycle.md (`hasUnsubmittedChanges`/`canHardDelete` SC-PACK-234/264…275);
  content maps in maps.md; game room I/O in game.md.
  Use when adding, changing, reviewing, or debugging Pinia stores or page-to-store
  wiring.
---

# Work With Stores

Use this skill when deciding where state should live or when changing Pinia stores in the **happy-tourist client** (`happy-tourist.github.io`).

This app uses **Pinia 4** with **six domain stores** (`auth`, `theme`, `game`, `support`, `content`, `maps`) plus Quasar’s Pinia entry. Pages use Composition API (`<script setup>`) and call `use*Store()` directly — not Vuex `map*`.

Skills for this client live under `.agents/skills/client/`. Runtime paths below are relative to this repo root.

## Layout

```
src/stores/
  index.ts          # Quasar defineStore → createPinia() (app entry, not a domain store)
  auth.ts           # setup store; role / isStaff / isAdmin from userdata
  theme.ts          # setup store — Quasar Dark preference
  game.ts           # options store — see game.md
  support.ts        # setup store — tickets + staff queue + admin users
  content.ts        # setup store — packs; see content.md
  maps.ts           # setup store — maps; see maps.md
  example-store.ts  # Quasar scaffold — unused by product
```

Prefer importing `use*Store` from `@/stores/<domain>`; keep Colyseus calls inside those stores.

## Specialized Topics

Read the matching file in this folder when the change involves that area
(do not load every file at once):

| Topic | File |
|-------|------|
| Content packs (`content` store; compose/tiles SC-PACK-256…263) | [content.md](content.md) |
| Pack draft lifecycle (hidden shell, dirty Submit, `canHardDelete`) | [pack-lifecycle.md](pack-lifecycle.md) |
| Content maps (`maps` store) | [maps.md](maps.md) |
| Game / lobby room I/O (`game` options store) | [game.md](game.md) |

### Store styles in this repo

| Store | Style | Why |
|-------|--------|-----|
| `auth` | **Setup** | Refs + computed, `onChange`, `whenReady` |
| `theme` | **Setup** | Dark preference + HTTP restore/toggle |
| `game` | **Options** | Room lifecycle + `this.*` — details in [game.md](game.md) |
| `support` | **Setup** | HTTP tickets/staff/admin; `change_pack`+`packId`; merge after `setUserRole` |
| `content` | **Setup** | Packs HTTP / working copy — [content.md](content.md) |
| `maps` | **Setup** | Maps HTTP / paint — [maps.md](maps.md) |
| `counter` | Options | Scaffold only — do not extend |

**When to choose setup vs options:** prefer **setup** for composables / HTTP CRUD / readiness; prefer **options** for action-centric room sessions (`game`). Do not convert style without a concrete reason.

### Naming conventions

| Layer | Pattern | Examples |
|-------|---------|----------|
| Store id | short id | `'auth'`, `'game'` |
| Composable | `use*Store` | `useAuthStore` |
| Actions | verb / domain | `login`, `subscribeLobby`, `createGame` |
| Internal helpers (options) | `_` prefix | `_enterRoom`, `_attachRoom` |

## State Ownership

### Local page state

Keep in the page when it belongs to one UI scenario: form drafts, UI toggles,
page-only busy flags, board geometry/selection/hints on `GamePage`.

### Pinia state

| Store | Owns | Typical consumers |
|-------|------|-------------------|
| **auth** | session / email flows / roles | Login*, Account, router, App |
| **theme** | Dark preference + `/api/theme` | App header (`work-with-styles/theme.md`) |
| **game** | lobby + tourist room — [game.md](game.md) | Lobby, Game, App leave |
| **support** | tickets / staff / admin users | Support*, AdminUsers |
| **content** | packs — [content.md](content.md) | Content* pages |
| **maps** | maps — [maps.md](maps.md) | MapsList / MapEditor |

### Auth vs theme vs game

- **auth** — Colyseus Auth only; no rooms / no Dark HTTP.
- **theme** — chrome Dark; wired from App; see `work-with-styles/theme.md`.
- **game** — rooms + sync + send* — [game.md](game.md).
- Cross-cutting: router `whenReady()`; App stable multi-source watch →
  `theme.syncFromAuthUser` (SC-THEME-10 — no `watch(() => […])`, no replace
  `auth.user` after GET).

### Decision checklist

Shared across pages / survives navigation / Colyseus I/O → Pinia. Form draft /
selection / one-off spinner → local `ref`.

## Core Patterns

### Keep Colyseus I/O in stores

Pages call store actions; stores call `client` / `room`. Do **not** scatter
`client.create` / `client.auth.*` / `client.http` / `room.send` across widgets.
Theme HTTP belongs in `stores/theme.ts`. Game `room.send` helpers → [game.md](game.md).

### Setup store (`auth`) sketch

```ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<AuthUser | null>(null);
  const token = ref<string | null>(null);
  const ready = ref(false);
  const isAuthenticated = computed(() => Boolean(token.value && user.value));
  client.auth.onChange((data) => {
    token.value = data.token ?? null;
    user.value = (data.user as AuthUser | null) ?? null;
    ready.value = true;
  });
  async function login(email: string, password: string) { /* catch → error */ }
  return { user, token, ready, isAuthenticated, login, whenReady };
});
```

Catch into store `error`; pages show `q-banner`. `whenReady()` gates the router.
Token key is SDK-managed (`colyseus-auth-token`).

### Options store (`game`)

See [game.md](game.md) — do not duplicate the full options sketch in the core.

### HMR: always `acceptHMRUpdate`

Every domain store ends with:

```ts
if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useAuthStore, import.meta.hot));
}
```

### Quasar Pinia entry (`stores/index.ts`)

Only `createPinia()` (+ optional plugins). New product stores are separate files.

## Using Stores In Pages / Router

```ts
const auth = useAuthStore();
const game = useGameStore();
await auth.login(email.value, password.value);
await game.subscribeLobby();
```

Router: `await useAuthStore().whenReady()`, then honor meta. Dependency:
`pages` → `stores` / `boot` / `components`.

## Adding A New Store Or Field

1. Prefer extending an existing domain store over a seventh store.
2. Pick setup vs options; add `acceptHMRUpdate`.
3. Put async Colyseus/HTTP in actions; wire pages with `use*Store()`.
4. Do not build on `example-store` / `counter`.

## Do / Don't

**Do**

- Keep Auth/room I/O in `auth` / `game`; tickets/packs in `support` / `content`.
- Local `ref` for forms, layout, page-only spinners.
- Content packs/maps: follow [content.md](content.md) / [maps.md](maps.md).
- Game mirror / D13 / send* lockstep: [game.md](game.md).

**Don't**

- Call `client.*` / `room.send` from random components / GamePage.
- Put board geometry or selection into Pinia without cross-page need.
- Assume `sendMove` / `sendPush` ends the turn.
- Fold packs into `support` or room lifecycle into `auth`.
- Pre-clear task slots before cascade save — [content.md](content.md).
- After theme GET, replace `auth.user` only to set `theme` (SC-THEME-10).

## Common Mistakes

- Scattering Colyseus calls across pages.
- Vuex `mapState` — this client is Pinia + `<script setup>`.
- Login form fields in `auth`; cell selection in `game`.
- LobbyPage HTTP poll — use `subscribeLobby` (`work-with-lobby`).
- Skipping `whenReady()`; attaching room listeners only in a page.
