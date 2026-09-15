---
name: work-with-pages
description: >-
  Use when creating or changing Vue page views under src/pages/*Page.vue, routes in
  src/router/routes.ts, router guards in src/router/index.ts, App.vue shell wiring,
  hash-mode deep links, meta.guest / meta.requiresAuth, or navigation between login,
  lobby, and game in this Quasar Vue 3 client.
---

# Work With Pages

Use this skill when adding or changing pages in the happy-tourist client (`happy-tourist.github.io`).

Stack: **Vue 3** Composition API (`<script setup>`), **TypeScript**, Vue Router **5** (**hash** mode), Quasar 2, Pinia.

Skills for this client live under `.agents/skills/client/`.

## Hard Restrictions

When the task is only about creating or wiring a page:

- Do not invent or expand i18n catalogs unless the user explicitly asks.
- Do not write tests unless the user explicitly asks for page tests.
- Do not install packages.
- Do not start long-lived dev servers unless needed for verification; when verifying, run `npm run lint` / `typecheck` from the client package root.
- Do not fix IDE diagnostics.

## Standard Page Wiring

1. Add the page as a single file `src/pages/<Name>Page.vue` (optional scoped `<style>` in the same file).
2. Register a route in `src/router/routes.ts` with `path`, `name`, lazy `component`, and `meta` when needed.
3. Do **not** enable filename-based routing — `quasar.config.ts` keeps `filenameBasedRouting: false`; routes stay manual.
4. Keep the global shell in `App.vue` — `q-layout` → shared theme `q-header` → `q-page-container` → theme `q-banner` + `<router-view />`. Route chrome (logout, leave-room, page banners) lives inside the page; do not duplicate the theme toggle.
5. Prefer: `pages` → `stores` / `boot` / `components`. Keep Colyseus I/O in Pinia (`auth`, `theme`, `game`), not scattered across new pages.
6. Wrap page content in Quasar `q-page` (match nearby pages).

If the route path or page name is unclear, ask the user before editing.

## Existing Routes

From `src/router/routes.ts` (hash mode via `createWebHashHistory` when `vueRouterMode: 'hash'`):

| Path | Name | Page | Notes |
|------|------|------|-------|
| `/` | — | — | redirect → `/lobby` |
| `/login` | `login` | `LoginPage` | `meta.guest` |
| `/lobby` | `lobby` | `LobbyPage` | `meta.requiresAuth` |
| `/game/:roomId` | `game` | `GamePage` | `meta.requiresAuth`; param `roomId` |
| `/:catchAll(.*)*` | — | — | redirect → `/lobby`; keep last |

Deep links on GitHub Pages use the hash form: `/#/lobby`, `/#/game/<roomId>`, `/#/login`.

## Page Component

Flat file with the `*Page` suffix (no per-page folder required):

```text
src/pages/
|-- LoginPage.vue
|-- LobbyPage.vue
`-- GamePage.vue
```

- Use `<script setup lang="ts">`.
- Path alias: `@/` → `src/`.
- Root element: `q-page` (optionally with Quasar utility classes like nearby pages).

Placeholder when UI is unspecified:

```vue
<template>
  <q-page class="q-pa-md">
    TODO
  </q-page>
</template>

<script setup lang="ts">
// page logic
</script>
```

## Page Composition

**Page owns the route view; stores own realtime/auth I/O.**

Allowed dependency direction:

- `pages` → `stores` / `boot` / `components`
- Prefer importing `client` from `@/boot/colyseus` only inside stores, not from many pages

Examples:

- `LoginPage` — form UI; calls `useAuthStore()` (`register` / `login` / `loginAnonymously`), then `router.replace`.
- `LobbyPage` — room list / create / join via `useGameStore()`; navigates to `game` with `roomId`.
- `GamePage` — board + presence UI (blue ring on current-turn seat); preset say bubbles/picker on own online marker (`sendSay` / `sayEvents`); ready/countdown UX; header turn text («Ваш ход» / «Ход соперника» / «Ход игрока»); syncs via `useGameStore()`; `rejoinGame(roomId)` on mount / soft-fail / browser reopen (`localStorage` token); exit via i18n `game.leave*` — seated ∧ `playing` → `q-dialog` confirm, else immediate `leaveGame` → `lobby`.

Do not put a second app shell (global layout host) inside a page — `App.vue` already mounts `router-view`.

## App Shell (`App.vue`)

Every route renders inside:

```vue
<q-layout view="hHh lpR fFf">
  <q-header bordered>
    <q-toolbar>
      <q-space />
      <q-btn flat round dense :icon="…" aria-label="Toggle theme" @click="onToggleTheme" />
    </q-toolbar>
  </q-header>

  <q-page-container>
    <q-banner v-if="theme.error" …>{{ theme.error }}</q-banner>
    <router-view />
  </q-page-container>
</q-layout>
```

Shared theme toggle + `theme.error` banner live here (`useThemeStore`). Wire restore with a stable multi-source watch — `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `syncFromAuthUser` (registered → `GET /api/theme`, guest → `localStorage`) — not JWT `user.theme` alone, and not `watch(() => […])` (new array each run). Do **not** replace `auth.user` after GET (theme lives in the theme store; SC-THEME-10). Do not duplicate a layout wrapper or per-page theme control when adding pages. Route-specific headers/actions stay in the page.

## Router Patterns

Two modules:

- `src/router/routes.ts` — route table (`RouteRecordRaw[]`)
- `src/router/index.ts` — `defineRouter`, history factory, global `beforeEach`

Match nearby style (lazy imports):

```ts
{
  path: '/example',
  name: 'example',
  component: () => import('@/pages/ExamplePage.vue'),
  meta: { requiresAuth: true },
},
```

For a new route:

- Add the record in `src/router/routes.ts` next to the other pages.
- Use a lowercase route `name` consistent with existing names (`login`, `lobby`, `game`).
- Set `meta.requiresAuth` for authenticated screens, or `meta.guest` for login-only screens.
- Keep the catch-all `/:catchAll(.*)*` redirect **last**.
- Do not switch `vueRouterMode` away from `hash` unless the user explicitly asks (GitHub Pages deep links).

### Auth `beforeEach`

`src/router/index.ts` always waits for auth readiness, then enforces meta:

```ts
Router.beforeEach(async (to) => {
  const auth = useAuthStore();
  await auth.whenReady();

  if (to.meta.requiresAuth && !auth.isAuthenticated) {
    return '/login';
  }

  if (to.meta.guest && auth.isAuthenticated) {
    return '/lobby';
  }
});
```

Do not remove `auth.whenReady()` or the `guest` / `requiresAuth` redirects unless the user explicitly asks.

## Navigation

Prefer named routes (as in lobby / game):

```ts
await router.push({ name: 'game', params: { roomId } });
await router.replace({ name: 'lobby' });
await router.replace({ name: 'login' });
```

Path strings also appear (login redirect / guard returns):

```ts
await router.replace('/lobby');
// guard:
return '/login';
```

Browser URL shape under hash mode: `/#/lobby`, `/#/game/<roomId>`.

If an old path changes, keep a redirect in `routes.ts`:

```ts
{ path: '/old-path', redirect: { name: 'lobby' } },
```

## Domain Map

| Domain | Page | Typical stores / notes |
|--------|------|------------------------|
| Auth | `LoginPage` | `stores/auth`; `meta.guest` |
| Theme (chrome Dark) | `App.vue` header | `stores/theme` + `boot/theme` |
| Lobby / rooms | `LobbyPage` | `stores/game.subscribeLobby`, create/join; `meta.requiresAuth` |
| Game session | `GamePage` | `stores/game` leave/`rejoinGame`/`sendMove`/`sendSay`; board + presence turn ring + say bubbles + seat pieces / strip; route param `roomId`; `meta.requiresAuth` |

Room name `tourist` + live lobby align with `../happy-tourist-server`; Game mirrors seats/`currentTurnSessionId` and renders pieces + local move chrome + ephemeral say bubbles.

## Verification and Final Response

For page-only tasks, verify by manually reviewing the changed page and `src/router/routes.ts` (and `src/router/index.ts` if guards changed). In the final response, state which files changed and explicitly mention that tests, scripts, package installs, and IDE diagnostics were not run or used when this skill forbids them.
