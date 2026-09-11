---
name: client-work-with-structure
description: >-
  Use when placing or moving UI in the happy-tourist Vue 3 client: pages
  (*Page.vue), components, Pinia stores, Quasar boot files, dependency
  direction between layers, or deciding whether new UI belongs in pages vs
  components vs stores. No blocks/ or dialogs/ registry layers.
---

# Work With Structure

## Overview

Use this skill when creating or relocating UI under `src`. Feature UI is
layered as **pages → stores / boot / components**. There are **no** `blocks/`
or `dialogs/` registry layers (unlike B2B).

Stack: Vue 3 Composition API (`<script setup>`), TypeScript, Vue Router 5
(hash), Pinia 4, Quasar 2 (`@quasar/app-vite`), vue-i18n, `@colyseus/sdk`.

Path alias: `@/` → `src/`.

Skills for this client live under `.agents/skills/client/` (this package; meta
may also host copies later).

## Core Rules

1. Before adding folders, inspect a nearby peer in the **same layer** and match its file name and import style.
2. Create only the files the feature needs. Do not add empty stubs “for later”.
3. Place UI by layer role (route page vs reusable widget vs store-owned I/O).
4. Respect **allowed dependency direction** (see below). Never invert layers.
5. Import `.vue` / `.ts` by direct path. No per-feature `index.ts` barrels required.
6. Keep `App.vue` minimal: `q-layout` + `q-page-container` + `<router-view />`. Page chrome (headers, logout, banners) lives **inside each page**.
7. Keep Colyseus auth/room I/O inside Pinia stores (`auth`, `game`). Prefer `import { client } from '@/boot/colyseus'` over `$colyseus` in script.
8. Use Quasar auto-imported components (`q-page`, `q-btn`, …). Do not manually register Quasar UI components.
9. Prefer Composition API + `<script setup lang="ts">`. Do not introduce Options API pages.

## Folder Map (`src`)

| Layer | Path | Role |
|-------|------|------|
| Pages | `src/pages/*Page.vue` | Route-level screens; compose stores + Quasar + optional components |
| Components | `src/components/` | Reusable widgets (mostly Quasar scaffold leftovers today) |
| Stores | `src/stores/` | Pinia: `auth`, `game` (+ unused scaffold `example-store`) |
| Boot | `src/boot/` | Quasar boot: `i18n`, `colyseus` (registered in `quasar.config.ts`) |
| Router | `src/router/` | `routes.ts` + guards in `index.ts` (`filenameBasedRouting: false`) |
| i18n | `src/i18n/` | Locale message trees (`en-US`) |
| CSS | `src/css/` | `app.scss`, `quasar.variables.scss` |
| Assets | `src/assets/` | Static assets |

Outside `src`: `public/`, `quasar.config.ts`, `.env.development` / `.env.production`, `.github/workflows/`.

**Not used in this project:** `src/blocks/`, `src/dialogs/`, Vuex, axios BFF layer, `*View` page suffix.

Scaffold leftovers (`EssentialLink.vue`, `example-store.ts`, unused `pages/index*`) — prefer the login / lobby / game flow; do not extend scaffold paths for new features.

## Dependency Direction

Allowed:

```text
pages       →  stores / boot / components / router (params)
components  →  other components (keep lean; prefer props over store)
stores      →  boot/colyseus (client)
boot        →  env / SDK / i18n setup only
App.vue     →  Quasar layout + router-view only
```

Also normal:

- Pages call Pinia actions (`useAuthStore`, `useGameStore`) and read store state.
- Router guards await `useAuthStore().whenReady()` then enforce `requiresAuth` / `guest`.
- Quasar components used in templates without local imports (auto-import).

**Forbidden inversions:**

| Wrong | Why |
|-------|-----|
| `components` → `pages` | Widgets stay page-agnostic |
| `stores` → `pages` / `components` | Data layer must not import UI |
| `boot` → `pages` / `components` | Boot is app setup only |
| Scattering `client.*` across many components | Keep Colyseus I/O in `stores/auth` and `stores/game` |
| Adding `blocks/` or `dialogs/` registry “like B2B” | This app has no those layers |

## Where New UI Belongs

Decide in this order:

1. **New route / screen?** → `src/pages/<Name>Page.vue` + route in `src/router/routes.ts`
2. **Reusable control used in 2+ pages or clearly generic?** → `src/components/<Name>.vue` (or small folder if peers do)
3. **Auth / room / move / listing logic?** → Pinia store (`stores/auth.ts` or `stores/game.ts`), not inline in the page beyond thin wiring
4. **App-wide plugin / SDK singleton?** → Quasar boot file under `src/boot/` + register in `quasar.config.ts`
5. **Page-local overlay / dialog?** → Keep in the page (or extract a component) with local `ref` / Quasar dialog props — **do not** invent a global dialogs registry

### Pages vs components — what belongs where

**Put in pages**

- Route entry (`*Page.vue`) and page orchestration (form state, selection, route params).
- Screen chrome for that route (title bar, logout, leave-room controls).
- Markup that exists only on that route (lobby list, board grid, login forms).
- Thin wiring: call store actions, show `store.error` via `q-banner`.

**Put in components**

- Small reusable widgets shared by 2+ pages (or clearly extractable board/cell chrome once reused).
- Prefer extracting only when reuse or page size justifies it — pages may own substantial markup today.

**Put in stores**

- Colyseus auth: register / login / anonymous / logout / `whenReady`.
- Room lifecycle: `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame`, `sendMove`.
- Mirrored room state: `board`, `myColor`, `currentTurn`, `status`, `rooms`, errors.

**Put in boot**

- `i18n` — `createI18n` + `app.use`.
- `colyseus` — `new Client(...)`, export `client`, optional `$colyseus` globalProperty.

### What NOT to put

| Avoid | Prefer |
|-------|--------|
| Global header/footer/dialog host in `App.vue` | Per-page chrome until a real shell is designed |
| Colyseus `client.create` / `send` / `auth.*` in a random component | `stores/auth` or `stores/game` |
| New `*View.vue` naming | `*Page.vue` |
| `src/blocks/` or `src/dialogs/index` registry | Page-local UI or a plain component |
| Empty layer folders “for later” | Add when the first file is needed |
| Extending unused scaffold pages (`pages/index*`) | Login / lobby / game routes only |
| Manual Quasar component registration | Auto-import |
| New axios/API module for Colyseus HTTP | LobbyRoom via store (`subscribeLobby`); `client.http` only as unused fallback |

## Naming

| Kind | Convention | Examples |
|------|------------|----------|
| Page file | PascalCase + `Page` suffix | `LoginPage.vue`, `LobbyPage.vue`, `GamePage.vue` |
| Route `name` | lowercase (existing) | `login`, `lobby`, `game` |
| Component file | PascalCase | `EssentialLink.vue` (scaffold); new: `BoardCell.vue` |
| Store file | kebab or descriptive | `auth.ts`, `game.ts` |
| Boot file | lowercase | `i18n.ts`, `colyseus.ts` |
| Imports | Alias `@/` | `import { client } from '@/boot/colyseus'` |

Pages are **flat files** under `src/pages/` (not `pages/LoginPage/LoginPage.vue` folders), matching current peers.

## Typical Shapes

### Page

```text
src/pages/LobbyPage.vue   # <script setup lang="ts"> + Quasar template
```

Existing pages: `LoginPage`, `LobbyPage`, `GamePage`.

### Component (when extracted)

```text
src/components/BoardCell.vue
```

### Store

```text
src/stores/auth.ts    # setup store
src/stores/game.ts    # options store
src/stores/index.ts   # Quasar Pinia entry
```

### Boot

```text
src/boot/i18n.ts
src/boot/colyseus.ts
```

Registered in `quasar.config.ts` boot array: `i18n`, `colyseus`.

## Real Composition Examples

**Auth page** — `LoginPage` uses `useAuthStore()` for register / login / anonymous; shows `error` banner; no Colyseus calls outside the store.

**Lobby** — `LobbyPage` uses `useGameStore().subscribeLobby` / `createGame` / `joinGame` and `useAuthStore` for display/logout.

**Game** — `GamePage` binds board from `useGameStore`, sends moves via `sendMove`, rejoins by `roomId` on refresh; local move highlights are UI-only.

**App shell** — `App.vue` only hosts `q-layout` → `router-view`; no header/footer/dialog host.

**Boot** — `colyseus.ts` exports singleton `client`; stores import it.

## Creating New Pieces — Checklist

**New page**

1. `src/pages/<Name>Page.vue` with `<script setup lang="ts">`.
2. Add route in `src/router/routes.ts` (lazy `() => import('@/pages/...')`), set `meta.requiresAuth` or `meta.guest` as needed.
3. Wire UI to Pinia; keep Colyseus I/O in stores.
4. Keep shell chrome in the page (not `App.vue`) unless introducing a deliberate shared layout.

**New component**

1. `src/components/<Name>.vue`.
2. Import from pages (or other components); do not import pages/stores unless clearly justified — prefer props/emits.
3. Do not call Colyseus directly; emit events or accept callbacks from the page/store consumer.

**New store logic**

1. Extend `stores/auth.ts` or `stores/game.ts` (or add a focused store if domain is new).
2. Import `client` from `@/boot/colyseus`.
3. Surface `error` / loading flags for pages to display.

**New boot file**

1. `src/boot/<name>.ts`.
2. Register in `quasar.config.ts` boot list.
3. Export symbols for stores/pages to import; avoid putting feature UI there.

## Domain Anchors

| Domain | Page | Store / boot |
|--------|------|----------------|
| Auth | `LoginPage` | `stores/auth` + `boot/colyseus` |
| Lobby / rooms | `LobbyPage` | `stores/game.subscribeLobby` / create / join |
| Game session | `GamePage` | `stores/game` room state + `sendMove` |
| Shell | `App.vue` | layout + `router-view` only |

Routes (from `src/router/routes.ts`):

| Path | Name | Meta |
|------|------|------|
| `/` → `/lobby` | — | redirect |
| `/login` | `login` | `guest` |
| `/lobby` | `lobby` | `requiresAuth` |
| `/game/:roomId` | `game` | `requiresAuth` |
| `/:catchAll(.*)*` | — | → `/lobby` |

Router mode: **hash** (`/#/lobby`, `/#/game/...`).

Board cell values (server truth): `0` empty, `1` white, `2` black, `3` white king, `4` black king.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Naming a screen `*View.vue` | Use `*Page.vue` |
| Adding `blocks/` or a dialogs registry | Keep UI in page/component |
| Putting room/auth SDK calls in the page body | Move to `stores/auth` / `stores/game` |
| Importing a page from a component | Invert: page imports the component |
| Growing `App.vue` with feature chrome | Keep chrome in pages |
| Using scaffold `pages/index*` for new routes | Add `*Page.vue` + `routes.ts` entry |
| Reintroducing Vuex or axios for Colyseus | Pinia + `client` / `client.http` |
| Manual Quasar imports for auto-imported tags | Use `q-*` in template as-is |

## Related Skills

- (Add as created) page routing / guards → `work-with-pages`
- (Add as created) Pinia auth/game → `work-with-stores`
- Sibling server contracts → `../happy-tourist-server` (coordinate room name `checkers`, move payload, state shape)

## Verification

For structure-only placement tasks, confirm:

- [ ] Correct layer (`pages` / `components` / `stores` / `boot`)
- [ ] `*Page.vue` naming (not `*View`)
- [ ] No new `blocks/` or `dialogs/` registry
- [ ] Dependency direction respected
- [ ] Colyseus I/O stays in stores
- [ ] Route registered in `routes.ts` with correct meta
- [ ] `App.vue` still shell-only unless a shared layout was explicitly requested
