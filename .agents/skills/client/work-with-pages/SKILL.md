---
name: work-with-pages
description: >-
  Use when creating or changing Vue page views under src/pages/*Page.vue, routes in
  src/router/routes.ts, router guards, or App.vue shell wiring in the happy-tourist
  Quasar Vue 3 client. Core in SKILL.md; topic details in shell.md (header/brand/
  crumbs/leave), content-pages.md (packs/maps UI), game-page.md (GamePage wiring).
  Pack/map store contracts → work-with-stores topics; board chrome → work-with-game-board.
---

# Work With Pages

Use this skill when adding or changing pages in the happy-tourist client (`happy-tourist.github.io`).

Stack: **Vue 3** Composition API (`<script setup>`), **TypeScript**, Vue Router **5** (**hash** mode), Quasar 2, Pinia.

Skills for this client live under `.agents/skills/client/`.

## Specialized Topics

Read the matching file in this folder when the change involves that area
(do not load every file at once):

| Topic | File |
|-------|------|
| App shell — brand logo, section nav, breadcrumbs, Game leave/status, theme banner | [shell.md](shell.md) |
| Content packs + maps pages / routes / list-card UI | [content-pages.md](content-pages.md) |
| GamePage composition + route (board details → `work-with-game-board`) | [game-page.md](game-page.md) |

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
4. Keep the global shell in `App.vue` — details in [shell.md](shell.md). Do not duplicate theme toggle, section/logout toolbar, or page «В лобби» when crumbs cover the path.
5. Prefer: `pages` → `stores` / `boot` / `components`. Keep Colyseus I/O in Pinia, not scattered across new pages.
6. Wrap page content in Quasar `q-page` (match nearby pages).

If the route path or page name is unclear, ask the user before editing.

## Existing Routes

From `src/router/routes.ts` (hash mode via `createWebHashHistory` when `vueRouterMode: 'hash'`):

| Path | Name | Page | Notes |
|------|------|------|-------|
| `/` | — | — | redirect → `/lobby` |
| `/login` | `login` | `LoginPage` | `meta.guest` |
| `/forgot-password` | `forgot-password` | `ForgotPasswordPage` | `meta.guest` |
| `/confirm-email` | `confirm-email` | `ConfirmEmailPage` | **public** (no `guest`) |
| `/reset-password` | `reset-password` | `ResetPasswordPage` | **public** |
| `/lobby` | `lobby` | `LobbyPage` | `requiresAuth`; room list only — sections/logout in App header |
| `/account` | `account` | `AccountPage` | `requiresAuth`; anonymous → lobby |
| `/support` | `support` | `SupportPage` | `requiresAuth` |
| `/support/staff` | `support-staff` | `SupportStaffPage` | `requiresStaff` |
| `/support/:id` | `support-ticket` | `SupportTicketPage` | `requiresAuth` |
| `/admin/users` | `admin-users` | `AdminUsersPage` | `requiresAdmin` |
| `/content/*` | — | — | See [content-pages.md](content-pages.md) |
| `/game/:roomId` | `game` | `GamePage` | See [game-page.md](game-page.md) |
| `/:catchAll(.*)*` | — | — | redirect → `/lobby`; keep last |

Deep links: `/#/lobby`, `/#/content/packs`, `/#/content/maps`, `/#/game/<roomId>`, …

## Page Component

Flat file with the `*Page` suffix:

```text
src/pages/
|-- LoginPage.vue
|-- LobbyPage.vue
|-- Support*.vue / AdminUsersPage.vue
|-- MapsListPage.vue / MapEditorPage.vue
|-- Content*.vue
`-- GamePage.vue
```

- Use `<script setup lang="ts">`. Path alias: `@/` → `src/`.
- Root element: `q-page`. Shared map chrome: `components/MapGridPreview.vue`.

## Page Composition

**Page owns the route view; stores own realtime/auth I/O.**

- `pages` → `stores` / `boot` / `components`
- Prefer importing `client` from `@/boot/colyseus` only inside stores

Examples (auth/lobby/support stay short here; packs/maps/game → topics):

- `LoginPage` / email-flow pages / `AccountPage` — `useAuthStore()`; soft verify modal in App ([shell.md](shell.md)).
- `LobbyPage` — room list / create (map + maxSeats + pack/task sets + densities) / join; **join busy-lock** (SC-LOBBY-19); selected-map mini `MapGridPreview` (SC-LOBBY-31); **no** page-local Packs/Maps/Support/logout.
- `Support*` / `AdminUsersPage` — `useSupportStore()`; `change_pack` + catalog pack select; staff/admin meta.
- Content packs/maps — [content-pages.md](content-pages.md).
- `GamePage` — [game-page.md](game-page.md).

Do not put a second app shell inside a page — `App.vue` already mounts `router-view`.

## Router Patterns

- `src/router/routes.ts` — route table; `src/router/index.ts` — `defineRouter` + `beforeEach`.
- Lazy imports; lowercase `name`; `meta.requiresAuth` / `guest` / `requiresStaff` / `requiresAdmin`.
- Keep catch-all last. Do not switch away from `hash` unless asked.

### Auth `beforeEach`

```ts
Router.beforeEach(async (to) => {
  const auth = useAuthStore();
  await auth.whenReady();
  if (to.meta.requiresAuth && !auth.isAuthenticated) return '/login';
  if (to.meta.guest && auth.isAuthenticated) return '/lobby';
  if (to.meta.requiresAdmin && !auth.isAdmin) return '/lobby';
  if (to.meta.requiresStaff && !auth.isStaff) return '/lobby';
});
```

Staff/admin meta gates nav only — server still enforces on HTTP.

## Navigation

Prefer named routes: `router.push({ name: 'game', params: { roomId } })`. Browser URL under hash: `/#/lobby`. Keep redirects for old paths in `routes.ts`.

## Domain Map

| Domain | Where |
|--------|--------|
| App shell / brand / crumbs / Game leave | [shell.md](shell.md) |
| Auth / Account | `Login*` / `AccountPage` + `stores/auth` |
| Lobby | `LobbyPage` + `stores/game` (`work-with-lobby`) |
| Support / admin | `Support*` / `AdminUsersPage` + `stores/support` |
| Content packs / maps | [content-pages.md](content-pages.md) |
| Game session | [game-page.md](game-page.md) |

## Verification and Final Response

For page-only tasks, verify by manually reviewing the changed page and `src/router/routes.ts` (and `src/router/index.ts` if guards changed). In the final response, state which files changed and explicitly mention that tests, scripts, package installs, and IDE diagnostics were not run or used when this skill forbids them.
