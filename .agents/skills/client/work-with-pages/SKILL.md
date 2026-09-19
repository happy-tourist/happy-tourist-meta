---
name: work-with-pages
description: >-
  Use when creating or changing Vue page views under src/pages/*Page.vue, routes in
  src/router/routes.ts, router guards in src/router/index.ts, App.vue shell wiring,
  hash-mode deep links, meta.guest / meta.requiresAuth, or navigation between login,
  lobby (create maxSeats + grilleDensity + catapultDensity), and game (top opponents presence, seated
  strip HUD row/2×2, budgets/end-turn icon on avatar, return strip icon no modal, push icons,
  nearest-center finish, grille trap/rescue + catapult land→overlay→fling + deferred grille drops + board-busy lock) in this Quasar Vue 3 client.
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
4. Keep the global shell in `App.vue` — `q-layout` → shared `q-header` → `q-page-container` → theme `q-banner` + `<router-view />`. Theme toggle is always in the header. On **Game** route only, the same header also owns icon-only leave (left) + centered match status; leave confirm + `leaveGame` orchestration live in `App.vue` — not a page-local game header. Login/Lobby keep theme-only chrome. Do not duplicate the theme toggle per page.
5. Prefer: `pages` → `stores` / `boot` / `components`. Keep Colyseus I/O in Pinia (`auth`, `theme`, `game`), not scattered across new pages.
6. Wrap page content in Quasar `q-page` (match nearby pages).

If the route path or page name is unclear, ask the user before editing.

## Existing Routes

From `src/router/routes.ts` (hash mode via `createWebHashHistory` when `vueRouterMode: 'hash'`):

| Path | Name | Page | Notes |
|------|------|------|-------|
| `/` | — | — | redirect → `/lobby` |
| `/login` | `login` | `LoginPage` | `meta.guest` |
| `/forgot-password` | `forgot-password` | `ForgotPasswordPage` | `meta.guest`; request reset mail |
| `/confirm-email` | `confirm-email` | `ConfirmEmailPage` | **public** (no `guest` — logged-in confirm must run); auto JSON on mount → lobby |
| `/reset-password` | `reset-password` | `ResetPasswordPage` | **public**; SPA form → JSON → login |
| `/lobby` | `lobby` | `LobbyPage` | `meta.requiresAuth` |
| `/account` | `account` | `AccountPage` | `meta.requiresAuth`; cabinet (registered only — anonymous → lobby) |
| `/game/:roomId` | `game` | `GamePage` | `meta.requiresAuth`; param `roomId` |
| `/:catchAll(.*)*` | — | — | redirect → `/lobby`; keep last |

Deep links on GitHub Pages use the hash form: `/#/lobby`, `/#/account`, `/#/forgot-password`, `/#/confirm-email`, `/#/reset-password`, `/#/game/<roomId>`, `/#/login`.

## Page Component

Flat file with the `*Page` suffix (no per-page folder required):

```text
src/pages/
|-- LoginPage.vue
|-- ForgotPasswordPage.vue
|-- ConfirmEmailPage.vue
|-- ResetPasswordPage.vue
|-- AccountPage.vue
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

- `LoginPage` — form UI; calls `useAuthStore()` (`register` / `login` / `loginAnonymously` / Google); link to forgot-password; then `router.replace`.
- `ForgotPasswordPage` — email → `auth.forgotPassword`; success / `email_not_found` banners; back to login.
- `ConfirmEmailPage` — public; auto `auth.confirmEmail` on mount → RU success → lobby (no Confirm button).
- `ResetPasswordPage` — public; form → `auth.resetPassword` → RU success → login.
- `AccountPage` — cabinet: confirm send + change email via store; anonymous redirect to lobby on mount.
- `LobbyPage` — room list / create (maxSeats + `grilleDensity` + `catapultDensity` few/medium/many each, default medium) / join via `useGameStore()`; navigates to `game` with `roomId`; account link for non-anonymous.
- `GamePage` — board in scroll region; unfinished pieces + finish travel from last board cell + fade + return travel from nearest center; holes for `removedTaskKeys` (not landable; piece may stand); grille overlays from `holdingGrilleKeys` (`grille.png` drop/rise, **`GRILLE_ANIM_MS = 1000`**; **defer new drops** while catapult hop queue busy — `pendingGrilleDropKeys`, flush on idle — SC-BOARD-29/30); catapult sequential overlays from `revealingCatapultKeys` / `brokenCatapultKeys` (`catapult.png` / `catapult-broken.png`, **`CATAPULT_ANIM_MS = 1000`**, broken 300+300 hold, land→overlay→fling for every viewer, D13 atomic mirror, `isBoardBusy` incl. pending grille); trapped pieces visible (no move/peek); rescue affordance **top-center** + `sendRescue`; push icons **top-center** over targets of selected free pusher + `sendPush`; **top** presence row (seated opponents / spectator all occupied); sticky bottom `.game-hud` **only when seated** (own + strip — wide row N,E,W,S / HUD ≤~420 → 2×2; **no** chip/`q-menu`); finish flag on strip; return → **green `undo` icon** over finished tourist when `canReturn` → same red `.tile--target` ring + `sendReturnFromFinish` (dim finished only if `!canReturn`; slot body finished → noop; **no** confirm modal); chrome grille on strip when trapped; finish 2×2 click → nearest legal center (Chebyshev); all-jail warning modal (`allJailWarning`); dual rings + 72px avatar + place/ready top-left + say top-right + end-turn `skip_next` **right-center** (`sendEndTurn` / `canSendEndTurn`; no dock/dialog; own-slot gap so icon does not cover budgets); say: top markers bubbles **down**, own bottom **up**; budgets **beside** own avatar; peek eye **top-center** + Correct/Wrong modal; keep-focus after non-finishing move/push; push→center travel+fade like move finish; +N budget fall ≈ 2 s; solo peeks∞ + dual timer-vs-steps end modals; ready/countdown UX; syncs via `useGameStore()`; `rejoinGame(roomId)` on mount / soft-fail / browser reopen. **No** page-local leave/status/roomId chrome — that lives in `App.vue` on Game route.

Do not put a second app shell (global layout host) inside a page — `App.vue` already mounts `router-view`.

## App Shell (`App.vue`)

Every route renders inside:

```vue
<q-layout view="hHh lpR fFf">
  <q-header bordered>
    <q-toolbar>
      <!-- Game route only: icon-only leave (left) + centered match status -->
      <q-btn v-if="isGameRoute" flat round dense icon="logout" :aria-label="t('game.leave')" … />
      <q-space />
      <div v-if="isGameRoute" class="text-subtitle1 text-center">{{ statusLabel }}</div>
      <q-space />
      <q-btn flat round dense :icon="…" aria-label="Toggle theme" @click="onToggleTheme" />
    </q-toolbar>
  </q-header>
  <!-- Game leave confirm dialog lives here (not on GamePage) -->
  <q-page-container>
    <q-banner v-if="theme.error" …>{{ theme.error }}</q-banner>
    <router-view />
  </q-page-container>
</q-layout>
```

Shared theme toggle + `theme.error` banner live here (`useThemeStore`). Account nav + once-per-session email-verify reminder modal (registered, `needsEmailVerification`; mark `sessionStorage` seen **when shown**) also live in `App.vue` — CTA to `/account`; do not claim mail was already sent (`client-work-with-auth`). On **Game** (`route.name === 'game'`): icon-only leave left + centered status from `useGameStore` (same branches as former page `statusLabel` — SC-PRESENCE-23); Login/Forgot/Lobby omit leave/status (SC-LEAVE-08). Leave confirm when seated ∧ `playing` ∧ `finishPlace === 0` ∧ `!timeExpired`, else immediate `leaveGame` → lobby (`work-with-rooms`). Wire theme restore with a stable multi-source watch — `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `syncFromAuthUser` (registered → `GET /api/theme`, guest → `localStorage`) — not JWT `user.theme` alone, and not `watch(() => […])` (new array each run). Do **not** replace `auth.user` after GET (theme lives in the theme store; SC-THEME-10). Do not duplicate a layout wrapper, per-page theme control, or page-local leave/status header when adding pages.

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
| Auth | `LoginPage` / `ForgotPasswordPage` / `ConfirmEmailPage` / `ResetPasswordPage` / `AccountPage` | `stores/auth`; guest / public / requiresAuth; soft verify modal in App |
| Theme (chrome Dark) | `App.vue` header | `stores/theme` + `boot/theme` |
| Game leave + match status | `App.vue` header (Game route only) | `stores/game` status / `leaveGame`; confirm dialog in App |
| Lobby / rooms | `LobbyPage` | `stores/game.subscribeLobby`, create `{ maxSeats, grilleDensity, catapultDensity }` / join; `meta.requiresAuth` |
| Game session | `GamePage` | `stores/game` `rejoinGame`/`sendMove`/`sendRescue`/`sendPush`/`sendReturnFromFinish`/`sendPeek`/`sendPeekAnswer`/`sendEndTurn`/`sendSay`; top presence + seated sticky `.game-hud` (own + strip row/2×2; no chip/`q-menu`); unfinished pieces + holes + grille overlays (`GRILLE_ANIM_MS=1000`; defer drop during catapult hops) + catapult land→overlay→fling (`CATAPULT_ANIM_MS=1000`, spectator parity, D13, board-busy lock) + trap/rescue/push + return strip icon (no modal) + all-jail modal + budgets beside avatar / end-turn icon on avatar + peek + finish nearest-center / timeout UX + dual rings + say top↓ / own↑; route param `roomId` (reconnect only — **not** shown in chrome); `meta.requiresAuth` |

Room name `tourist` + live lobby align with `../happy-tourist-server`; Game mirrors seats/`finishPlace`/`timeExpired`/piece `finished`/`trapped`/`currentTurnSessionId`/`turnUntil`/`turnBudgetSeconds`/`removedTaskKeys`/`holdingGrilleKeys`/`revealingCatapultKeys`/`brokenCatapultKeys`, listens private `budgets`/`peekOpen`/`allJailWarning`, and renders unfinished pieces (after materialize) + holes + grille/catapult overlays + rescue/push/return-icon chrome + own counters (beside avatar) / end-turn icon on avatar + peek chrome + finish/timeout + top + seated-bottom presence rings + strip + local move chrome (eligible seats; move/push do not end turn; trapped locked) + ephemeral say bubbles (top↓ / own↑).

## Verification and Final Response

For page-only tasks, verify by manually reviewing the changed page and `src/router/routes.ts` (and `src/router/index.ts` if guards changed). In the final response, state which files changed and explicitly mention that tests, scripts, package installs, and IDE diagnostics were not run or used when this skill forbids them.
