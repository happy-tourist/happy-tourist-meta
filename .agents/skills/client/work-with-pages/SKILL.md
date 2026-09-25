---
name: work-with-pages
description: >-
  Use when creating or changing Vue page views under src/pages/*Page.vue, routes in
  src/router/routes.ts, router guards in src/router/index.ts, App.vue shell wiring
  (section Packs/Maps/Support + staff Модерация; narrow burger rightmost + Acc/Theme/Logout
  in menu SC-BRAND-19; crumbs inside q-page-container SC-BRAND-17…18; session logout;
  Game status + header-game-leave), hash-mode deep links, meta.guest / meta.requiresAuth /
  requiresStaff / requiresAdmin, or navigation between login, lobby (room list only —
  sections/logout in App header → content-catalog / content-maps), support/content/admin
  pages, unified packs list (filters/statuses/favorites; no published badge; collection
  redirect) + author re-edit + staff take + MapsList/MapEditor (view meta vs paint;
  crumbs replace «К картам»); pack/map editor/staff details in work-with-stores topic
  files; App brand logo (always-button ≥60px; Game leave / auth+other → lobby / lobby noop;
  no page «В лобби» / list «К наборам» / moderation catalogNav when crumbs cover path),
  productName/favicon, and
  game (top opponents presence, seated strip HUD row/2×2, budgets/end-turn icon on avatar,
  return strip icon no modal, push icons, nearest-center finish, grille trap/rescue +
  catapult land→overlay→fling + deferred grille drops + board-busy lock) in this Quasar
  Vue 3 client.
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
4. Keep the global shell in `App.vue` — `q-layout` → shared elevated `q-header` → `q-page-container` (first: **page-zone breadcrumbs** for packs/maps/staff|author moderation — SC-BRAND-17…18 / D13; then theme `q-banner` + `<router-view />`). Brand logo (≥60px height, `@/assets/brand/logo.png`) is always left as the **same** interactive `<button>` on every route (no bare `img` / decorative swap — SC-BRAND-09). Authenticated non-Game / non-auth **wide**: section links (Packs/Maps/Support + staff Модерация) + account + theme + **session logout** (rightmost). **Narrow** with section nav: burger **rightmost**; Packs/Maps/Support/Модерация **and** Acc/Theme/Logout **inside** that menu (SC-BRAND-14/19 / D21) — do **not** also show Acc/Theme/Logout as toolbar icons. On **Game**: centered match status; leave = logo click **and** right-side `header-game-leave` (same confirm / `leaveGame`) — **no** session logout, **no** section burger fold. Auth/login: no section burger; Theme stays in toolbar. Logo click modes: auth → lobby; lobby → noop; Game → leave; other authenticated → lobby. Do **not** add page-level «В лобби» / list «К наборам» / «К картам» when breadcrumbs cover the path (incl. pack/staff/my-moderation chrome — SC-PACK-194/195 / D20). Do not duplicate the theme toggle or section/logout toolbar per page.
5. Prefer: `pages` → `stores` / `boot` / `components`. Keep Colyseus I/O in Pinia (`auth`, `theme`, `game`, `support`, `content`), not scattered across new pages.
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
| `/lobby` | `lobby` | `LobbyPage` | `meta.requiresAuth`; room list only — Packs/Maps/Support/Moderation/account/logout in App header (SC-BRAND-11…15) |
| `/account` | `account` | `AccountPage` | `meta.requiresAuth`; cabinet (registered only — anonymous → lobby) |
| `/support` | `support` | `SupportPage` | `meta.requiresAuth`; create + own list (topic `change_pack` + catalog pack select title/description) |
| `/support/staff` | `support-staff` | `SupportStaffPage` | `meta.requiresAuth` + `requiresStaff` |
| `/support/:id` | `support-ticket` | `SupportTicketPage` | `meta.requiresAuth`; thread (`packId` when change_pack) |
| `/admin/users` | `admin-users` | `AdminUsersPage` | `meta.requiresAuth` + `requiresAdmin` |
| `/content/packs` | `content-catalog` | `ContentCatalogPage` | `meta.requiresAuth`; **unified packs list** (filters/statuses/star; staff soft-unpublish; **no** list-chrome staff «Модерация» — App header SC-PACK-184; published/`in_catalog` rows **no** status badge SC-PACK-185) |
| `/content/collection` | `content-collection` | — | **redirect** → `content-catalog` (SC-PACK-164; do not revive collection page) |
| `/content/my-moderation` | `content-my-moderation` | `ContentMyModerationPage` | `meta.requiresAuth`; deep-link/API retained; **no** non-staff header nav (use catalog filters); **no** `catalogNav` (SC-PACK-194) |
| `/content/maps` | `content-maps` | `MapsListPage` | `meta.requiresAuth`; list filters/statuses + Create; no collection; staff Модерация in App header (SC-MAP-43); published rows **no** badge (SC-MAP-45) |
| `/content/maps/:id/edit` | `content-map-edit` | `MapEditorPage` | `meta.requiresAuth`; view = author+seats (no paint); edit = palette; breadcrumbs replace «К картам» (SC-MAP-44/46/47); staff `?staff=1` (blocked if author request open) |
| `/content/packs/new` | `content-pack-new` | `ContentPackCreatePage` | `meta.requiresAuth`; create → working-copy editor; verify modal if ineligible |
| `/content/packs/:id` | `content-pack` | `ContentPackPage` | `meta.requiresAuth`; live + star + author/staff Edit gates + add-task-set beside «Задания» (verified+in-catalog; **no** collect) |
| `/content/packs/:id/edit` | `content-pack-edit` | `ContentPackEditorPage` | `meta.requiresAuth`; creator **or** `editorKind` author re-edit (`submitPack`) or staff (`staffSavePack` + lock session) |
| `/content/packs/:id/add-task-set` | `content-pack-add-task-set` | `ContentPackAddTaskSetPage` | `meta.requiresAuth`; post-publish add-only new task set (**no** collection gate) |
| `/content/packs/:id/tasks/:taskSetId` | `content-pack-tasks` | `ContentPackTasksPage` | `meta.requiresAuth`; nested task-set (unpublished / author / staff session) |
| `/content/packs/:id/moderation` | `content-pack-moderation` | `ContentPackModerationPage` | `meta.requiresAuth`; author thread (also embedded on edit); **no** `catalogNav` (SC-PACK-194) |
| `/content/staff` | `content-staff` | `ContentStaffPage` | `meta.requiresAuth` + `requiresStaff`; queue + **Take** (pack\|map); **no** `catalogNav` (SC-PACK-195) |
| `/content/staff/requests/:id` | `content-staff-request` | `ContentStaffRequestPage` | `meta.requiresAuth` + `requiresStaff`; take before Approve / needs-revision / cancel; **no** `catalogNav` (SC-PACK-195) |
| `/content/staff/requests/:id/tasks` | `content-staff-request-tasks` | `ContentStaffTasksPage` | `meta.requiresAuth` + `requiresStaff`; redirect → request hub (legacy bookmark); **no** `catalogNav` |
| `/game/:roomId` | `game` | `GamePage` | `meta.requiresAuth`; param `roomId` |
| `/:catchAll(.*)*` | — | — | redirect → `/lobby`; keep last |

Deep links on GitHub Pages use the hash form: `/#/lobby`, `/#/account`, `/#/support`, `/#/content/packs`, `/#/content/maps`, `/#/content/staff`, `/#/admin/users`, `/#/forgot-password`, `/#/confirm-email`, `/#/reset-password`, `/#/game/<roomId>`, `/#/login`.

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
|-- SupportPage.vue
|-- SupportTicketPage.vue
|-- SupportStaffPage.vue
|-- AdminUsersPage.vue
|-- MapsListPage.vue
|-- MapEditorPage.vue
`-- GamePage.vue
```

- Use `<script setup lang="ts">`.
- Path alias: `@/` → `src/`.
- Root element: `q-page` (optionally with Quasar utility classes like nearby pages).
- Shared map grid chrome: `components/MapGridPreview.vue` (list mini-preview, editor paint surface, staff request preview).

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

- `LoginPage` — form UI; calls `useAuthStore()` (`register` / `login` / `loginAnonymously` / Google); register: required name + password policy + meter; login: no complexity; link to forgot-password; then `router.replace`.
- `ForgotPasswordPage` — email → `auth.forgotPassword`; success / `email_not_found` banners; back to login.
- `ConfirmEmailPage` — public; auto `auth.confirmEmail` on mount → RU success → lobby (no Confirm button).
- `ResetPasswordPage` — public; policy + meter → `auth.resetPassword` → RU success → login.
- `AccountPage` — cabinet: displayName form; change-password if `canChangePassword`; confirm send + change email; logout; soft-verify (no hard-gate); anonymous redirect to lobby on mount; **no** page «В лобби» — use App brand logo.
- `LobbyPage` — greeting + room list / create (maxSeats + `grilleDensity` + `catapultDensity` few/medium/many each, default medium) / join via `useGameStore()`; **join busy-lock** (early-return + disable rows while `joining` — SC-LOBBY-19); navigates to `game` with `roomId`. **No** page-local Packs/Maps/Support/account/logout — those live in App header (SC-BRAND-15).
- `SupportPage` — create ticket + own list via `useSupportStore()`; guest banner (no email notify + session-loss risk); create form: `lazy-rules` + clear → `await nextTick()` → `resetValidation` (see `work-with-forms`); **no** page «В лобби» — use App brand logo.
- `SupportTicketPage` — public thread (gap messages + gap thread→reply); **one** Close via `canClose` (author **or** staff — no duplicate when staff is author); staff take / set-awaiting (**only when** `status === 'in_progress'`); reply: `lazy-rules` + clear → `nextTick` → `resetValidation`; closed read-only + CTA new ticket.
- `SupportStaffPage` — staff queue (`requiresStaff`); topic filter (default all) + status `open`\|`closed`\|`all` (default `open`); refresh list.
- `AdminUsersPage` — list users + change role (`requiresAdmin`); server excludes anonymous; badge/chip when `emailVerified === false` (survives role change — store merge); moderator has no role UI.
- `GamePage` — board in scroll region; unfinished pieces + finish travel from last board cell + fade + return travel from nearest center; holes for `removedTaskKeys` (not landable; piece may stand); grille overlays from `holdingGrilleKeys` (`grille.png` drop/rise, **`GRILLE_ANIM_MS = 1000`**; **defer new drops** while catapult hop queue busy — `pendingGrilleDropKeys`, flush on idle — SC-BOARD-29/30); catapult sequential overlays from `revealingCatapultKeys` / `brokenCatapultKeys` (`catapult.png` / `catapult-broken.png`, **`CATAPULT_ANIM_MS = 1000`**, broken 300+300 hold, land→overlay→fling for every viewer, D13 atomic mirror, continuous `isBoardBusy` = intent lock before send + visual presentation + pipeline settle into grille hold — SC-BOARD-27/32); trapped pieces visible (no move/peek); rescue affordance **top-center** + `sendRescue`; push icons **top-center** over targets of selected free pusher + `sendPush`; **top** presence row (seated opponents / spectator all occupied; **seated always reserves row height** even with 0 opponents — SC-PRESENCE-26); sticky bottom `.game-hud` **only when seated** (own + strip — wide row N,E,W,S / HUD ≤~420 → 2×2; **no** chip/`q-menu`); finish flag on strip; return → **green `undo` icon** over finished tourist when `canReturn` → same red `.tile--target` ring + `sendReturnFromFinish` (dim finished only if `!canReturn`; slot body finished → noop; **no** confirm modal); chrome grille on strip when trapped; finish 2×2 click → nearest legal center (Chebyshev); all-jail warning modal (`allJailWarning`); dual rings + 72px avatar + place/ready top-left + say top-right + end-turn `skip_next` **right-center** (`sendEndTurn` / `canSendEndTurn`; no dock/dialog; own-slot gap so icon does not cover budgets); say: top markers bubbles **down**, own bottom **up** (say stays available under board-busy); budgets **beside** own avatar; peek eye **top-center** + shared Q&A modal (`sendPeek` / `sendPeekPlace` / `sendPeekSubmit`); focus nearest-actionable on own avatar; keep-focus after non-finishing move/push; push→center travel+fade like move finish; +N budget fall ≈ 2 s; solo peeks∞ + dual timer-vs-steps end modals; ready/countdown UX; syncs via `useGameStore()`; `rejoinGame(roomId)` on mount / soft-fail / browser reopen. **No** page-local leave/status/roomId chrome — leave is App brand-logo **and** right-side `header-game-leave` + status in `App.vue` on Game.

Do not put a second app shell (global layout host) inside a page — `App.vue` already mounts `router-view`.

## App Shell (`App.vue`)

Every route renders inside:

```vue
<q-layout view="hHh lpR fFf">
  <q-header bordered>
    <q-toolbar>
      <!-- Brand logo left (≥60px); always same button (auth/other → lobby; lobby noop; Game leave) -->
      <button type="button" class="brand-logo-control"
        :aria-label="brandLogoAria" @click="onBrandLogoClick">
        <img :src="brandLogoUrl" alt="" class="brand-logo" />
      </button>
      <!-- Wide: Packs / Maps / Support (+ staff Модерация); Acc/Theme/Logout rightmost cluster -->
      <!-- Narrow + section nav: burger rightmost; sections + Acc/Theme/Logout inside menu (SC-BRAND-19) -->
      <q-space />
      <div v-if="isGameRoute" class="text-subtitle1 text-center">{{ statusLabel }}</div>
      <q-space />
      <!-- wide: account + theme; Game → header-game-leave; else session logout; narrow fold → burger -->
    </q-toolbar>
  </q-header>
  <!-- Game leave confirm dialog lives here (not on GamePage) -->
  <q-page-container>
    <!-- SC-BRAND-17 / SC-PACK-193 / SC-MAP-52: crumbs inside page-container offset zone (D13), not sibling above it -->
    <div v-if="breadcrumbItems.length" class="app-breadcrumbs" data-test-id="app-breadcrumbs">…</div>
    <q-banner v-if="theme.error" …>{{ theme.error }}</q-banner>
    <router-view />
  </q-page-container>
</q-layout>
```



Brand logo (`@/assets/brand/logo.png`, height ≥60px, `width: auto; object-fit: contain`) is always left as one interactive control. Click modes (`brandLogoMode`): **Game** → `leave` (`onExitClick` / confirm / `leaveGame`); **auth** routes (`login` / `forgot-password` / `confirm-email` / `reset-password`) → `toLobby` (`router.push({ name: 'lobby' })`; guest may bounce via `requiresAuth`); **lobby** → `noop` (same button, click ignored); other authenticated → `toLobby`. Aria: Game → `game.leave`; non-Game → `auth.backToLobby` (key kept for aria; no page «В лобби» buttons on Account/Support).

**Shared header chrome (SC-BRAND-11…19 / SC-LEAVE-08…12):** to the right of the logo on authenticated non-Game / non-auth **wide** screens — Packs / Maps / Support (+ staff «Модерация») + account + theme + **session logout** (logout rightmost). **Narrow** with section nav: burger **rightmost**; menu = sections then Acc / Theme / Logout (no separate toolbar Acc/Theme/Logout — SC-BRAND-14/19). On **Game** — centered match status, **right-side leave-from-room** (`header-game-leave`, same confirm as logo), **no** session logout, **no** section burger fold. Auth/login: Theme in toolbar; no section burger. Breadcrumbs sit **inside** `q-page-container` (header offset zone; `app-breadcrumbs` — SC-BRAND-17…18 / SC-PACK-193 / SC-MAP-52 / D13), not inside the elevated header and not as a sibling between `</q-header>` and `<q-page-container>`; staff/author moderation crumbs Lobby / Модерация (SC-BRAND-18); live pack/tasks and moderation chrome omit redundant «К наборам» / «Вернуться» / `content.catalogNav` when crumbs cover the path (SC-PACK-194/195). Shared theme toggle + `theme.error` banner live here (`useThemeStore`). Account nav + once-per-session email-verify reminder modal (registered, `needsEmailVerification`; mark `sessionStorage` seen **when shown**) also live in `App.vue` — CTA to `/account`; do not claim mail was already sent (`client-work-with-auth`). On **Game** (`route.name === 'game'`): centered status from `useGameStore` (same branches as former page `statusLabel` — SC-PRESENCE-23). Leave confirm when seated ∧ `playing` ∧ `finishPlace === 0` ∧ `!timeExpired`, else immediate `leaveGame` → lobby (`work-with-rooms`). Document title: `package.json` `productName` = `Happy Tourist` (`index.html` `<%= productName %>`); favicon: only `favicon.ico` in `index.html` / `public/` (no scaffold PNG icon set). Wire theme restore with a stable multi-source watch — `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `syncFromAuthUser` (registered → `GET /api/theme`, guest → `localStorage`) — not JWT `user.theme` alone, and not `watch(() => […])` (new array each run). Do **not** replace `auth.user` after GET (theme lives in the theme store; SC-THEME-10). Do not duplicate a layout wrapper, per-page theme control, page «В лобби», Lobby section toolbar, or page-local leave/status header when adding pages.

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

  if (to.meta.requiresAdmin && !auth.isAdmin) {
    return '/lobby';
  }

  if (to.meta.requiresStaff && !auth.isStaff) {
    return '/lobby';
  }
});
```

Do not remove `auth.whenReady()` or the `guest` / `requiresAuth` redirects unless the user explicitly asks. Staff/admin meta gates nav only — server still enforces on HTTP.

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
| Brand + section nav + breadcrumbs + Game leave + match status | `App.vue` elevated header (logo ≥60px; Packs/Maps/Support/Модерация; narrow burger rightmost + Acc/Theme/Logout in menu SC-BRAND-19) + crumbs **inside** `q-page-container` (SC-BRAND-17…18); Game status + `header-game-leave` | logo `leave`/`toLobby`/noop; session logout off-Game (wide toolbar / narrow menu); `stores/game` / `leaveGame`; `auth.backToLobby` aria-only |
| Title / favicon | `package.json` + `index.html` + `public/favicon.ico` | `productName` = Happy Tourist; single `favicon.ico` link |
| Lobby / rooms | `LobbyPage` | `stores/game.subscribeLobby`, create `{ maxSeats, grilleDensity, catapultDensity }` / join; **no** page section/logout chrome (App header → `content-catalog` / `content-maps` / support); `meta.requiresAuth` |
| Support | `SupportPage` / `SupportTicketPage` / `SupportStaffPage` | `stores/support` HTTP; `change_pack` + catalog pack select; `requiresAuth`; staff uses `requiresStaff` + `auth.isStaff` |
| Content packs | `ContentCatalogPage` (filters/star; **no** published badge; **no** list staff Модерация; row open: never-published / pack-level draft\|pending\|needs_revision → Edit; add-task-set-only → live first via `openRequestType`; SC-PACK-186/191/192) / `ContentMyModerationPage` (deep-link only; **no** `catalogNav` SC-PACK-194) / `ContentPackPage` (breadcrumbs replace «К наборам»; set-row `moderationStatus` for set author+staff; `neverLive` ghost → add-task-set Edit SC-PACK-188…190; star; Edit gates; add-task-set) / `ContentPackCreatePage` / `ContentPackEditorPage` / `ContentPackAddTaskSetPage` / `ContentPackTasksPage` (live: no «Вернуться» — crumbs; editor keeps back-to-answers) / `ContentPackModerationPage` (**no** `catalogNav`) / `ContentStaffPage` / `ContentStaffRequestPage` / `ContentStaffTasksPage` (**no** `catalogNav` SC-PACK-195) | `stores/content`; cancel→draft; `TaskSet.moderationStatus` / `neverLive`; `openRequestType`; SC-PACK-148…195; staff `requiresStaff` |
| Content maps | `MapsListPage` (author draft/pending/needs_revision → Edit SC-MAP-50; clean published → View) / `MapEditorPage` (title-row Edit SC-MAP-51; + `MapGridPreview`) | `stores/maps`; list no published badge; view-only meta vs paint; crumbs inside `q-page-container`; cancel→draft; SC-MAP-41…52 |
| Roles / admin | `AdminUsersPage` | `stores/support` admin HTTP; `requiresAdmin` + `auth.isAdmin` (server enforces) |
| Game session | `GamePage` | `stores/game` `rejoinGame`/`sendMove`/`sendRescue`/`sendPush`/`sendReturnFromFinish`/`sendPeek`/`sendPeekPlace`/`sendPeekSubmit`/`sendEndTurn`/`sendSay`; top presence + seated sticky `.game-hud` (own + strip row/2×2; no chip/`q-menu`); unfinished pieces + holes + grille overlays (`GRILLE_ANIM_MS=1000`; defer drop during catapult hops) + catapult land→overlay→fling (`CATAPULT_ANIM_MS=1000`, spectator parity, D13, board-busy lock) + trap/rescue/push + return strip icon (no modal) + all-jail modal + budgets beside avatar / end-turn icon on avatar + peek + finish nearest-center / timeout UX + dual rings + say top↓ / own↑; route param `roomId` (reconnect only — **not** shown in chrome); `meta.requiresAuth` |

Room name `tourist` + live lobby align with `../happy-tourist-server`; Game mirrors seats/`finishPlace`/`timeExpired`/piece `finished`/`trapped`/`currentTurnSessionId`/`turnUntil`/`turnBudgetSeconds`/`removedTaskKeys`/`holdingGrilleKeys`/`revealingCatapultKeys`/`brokenCatapultKeys`, listens private `budgets`/`peekOpen`/`allJailWarning`, and renders unfinished pieces (after materialize) + holes + grille/catapult overlays + rescue/push/return-icon chrome + own counters (beside avatar) / end-turn icon on avatar + peek chrome + finish/timeout + top + seated-bottom presence rings + strip + local move chrome (eligible seats; move/push do not end turn; trapped locked) + ephemeral say bubbles (top↓ / own↑).

## Verification and Final Response

For page-only tasks, verify by manually reviewing the changed page and `src/router/routes.ts` (and `src/router/index.ts` if guards changed). In the final response, state which files changed and explicitly mention that tests, scripts, package installs, and IDE diagnostics were not run or used when this skill forbids them.
