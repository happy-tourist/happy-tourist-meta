## Context

See `proposal.md` — Why. Client-only change; anonymous JWT and guest create gates already exist on server.

Today: `src/router/index.ts` redirects any `requiresAuth` without session to `/login` (no silent guest, no `?redirect=`). Login `meta.guest` bounces **any** authenticated user (including anonymous) away from Login. Header (`App.vue`) shows logout for all authenticated users. Packs/maps filter chrome uses `:disable` for `identityOnly && isGuest`. Breadcrumbs sit in `q-page-container` **above** `q-page`; Quasar page min-height ≈ container → phantom scroll on packs/maps (lobby has no crumbs).

## Goals / Non-Goals

**Goals:**

- Silent anonymous on ordinary protected navigation; stay on target URL.
- Staff/admin without session → Login only.
- Anonymous can open Login; header login icon; hide «Войти как гость» when already guest; return after upgrade.
- Hide identity filters on packs + maps for guests.
- Eliminate phantom scroll when crumbs + short content.

**Non-Goals:**

- Server auth/API changes; new deps; share-query invite links; redesign Login chrome beyond guest-button visibility.

## Decisions

### D1 — Silent guest in router `beforeEach` (not Login soft-gate)

After `auth.whenReady()`, if `to.meta.requiresAuth && !auth.isAuthenticated` and **not** (`requiresStaff` | `requiresAdmin`): call `auth.loginAnonymously()`, then allow navigation (retry/continue to `to`). If anonymous sign-in fails → Login with error via existing auth error surface (SC-AUTH-11b).

Ordinary `requiresAuth` includes `/account` and `/support`: silent guest still runs; page-level rules unchanged (Account already bounces anonymous → lobby; Support already allows guest tickets). Do **not** special-case those routes in the guard.

**Alternatives:** redirect to Login with auto-click guest (flicker); per-page mount hooks (easy to miss routes). Rejected for consistency.

### D2 — Staff/admin unauthenticated → Login without anonymous

If `requiresStaff` or `requiresAdmin` and `!isAuthenticated` → Login with `query.redirect` set to the requested staff/admin path (so post-login return works via existing `goAfterLogin`). Do **not** call `loginAnonymously` first.

### D3 — Guest guard exception for anonymous (Login only)

On the **Login** route (`name: 'login'` / `meta.guest` for that screen): bounce to lobby only when authenticated **and** `!auth.user?.anonymous` (registered session). Anonymous MAY stay on Login to upgrade.

Do **not** broaden this to every `meta.guest` route. `forgot-password` (also `meta.guest`) MUST keep today’s bounce for **any** authenticated session including anonymous — guest has no password-reset path, and opening forgot-password while already anonymous is not a product goal.

### D4 — Header: login replaces logout for anonymous

In `App.vue`: `showSessionLogout` only for registered (non-anonymous) off-Game and outside auth screens. Reuse today’s `isLoginRoute` / `AUTH_ROUTE_NAMES` set (`login` | `forgot-password` | `confirm-email` | `reset-password`) so neither logout nor the guest login affordance appear on those screens (same omission surface as current logout). For anonymous off-Game **and not on that auth-route set** show icon `login` (same flat/dense pattern as current logout); add i18n aria key (e.g. `auth.login` — «Войти») parallel to `auth.logout`. Navigate to `{ name: 'login', query: { redirect: current fullPath } }` without logout. Narrow burger: same swap. Keep today’s omission of the account control for anonymous (cabinet remains registered-only); guest wide/burger cluster = theme + login.

### D5 — Post-auth return

Reuse existing `LoginPage` `goAfterLogin` + `route.query.redirect`. Header login must set `redirect`. Silent guest path does not need redirect.

### D6 — Hide identity filters

`ContentCatalogPage.vue` / `MapsListPage.vue`: render identity filter buttons only when `!isGuest` (`v-if`), not `:disable`. Keep the **all** filter control visible for guests (SC-PACK-286 / SC-MAP-69). Keep filter logic empty-guard as safety.

### D7 — Phantom scroll

Account for breadcrumb strip height against `q-page` min-height (preferred: CSS on layout when `.app-breadcrumbs` present — reduce page min-height / flex column so page fills remaining space without overflowing). Avoid moving crumbs into every page. Lobby unchanged (no crumbs).

**Files (client):**

| Area | Touch |
|------|--------|
| Guard / silent guest | `src/router/index.ts`, `src/stores/auth.ts` (`loginAnonymously`) |
| Login upgrade UX | `src/pages/LoginPage.vue` |
| Header | `src/App.vue` (+ i18n aria if needed) |
| Filters | `src/pages/ContentCatalogPage.vue`, `src/pages/MapsListPage.vue` |
| Scroll | `src/App.vue` / `src/css/app.scss` |
| Tests | `AppHeaderChrome.test.ts`, Login/router tests, ContentCatalog / MapsList filter tests |

Server: none required.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Every cold visit to `/`→lobby creates anonymous accounts | Accepted product choice; same as manual guest |
| Anonymous on Login then browser Back | Session stays guest — OK |
| Silent guest fails (network) | Fall back to Login; surface auth error |
| Scroll CSS fights Quasar layout | Prefer min-height calc / flex over `overflow:hidden` on body |
| Staff deep-link loses target if no redirect | Required: `?redirect=` (or equivalent) to staff path on Login (D2 / SC-AUTH-12) |
| Broad `meta.guest` exception lets anonymous open forgot-password | D3: Login-only exception; other guest routes keep full authenticated bounce |

## Migration Plan

Client-only deploy. No DB migration. Rollback = revert client. Existing sessions unchanged.

## Open Questions

None — explore decisions locked (auto-guest scope, login icon, hide filters packs+maps, staff→Login, hide guest button, scroll fix).

## Technical prerequisites

None blocking. Stack already has `signInAnonymously`, guest create gates, filter `identityOnly` flag.

## Server contract (unchanged — aligns with sibling)

Client-only change; **no** new room messages, HTTP routes, schema fields, or DB migrations. Silent guest and catalog UX reuse existing server surfaces:

| Surface | Role for this change |
|---------|----------------------|
| `@colyseus/auth` anonymous register (`onRegisterAnonymously` + JWT via `signInAnonymously`) | Silent `loginAnonymously` / guest session; MUST NOT invent a new auth endpoint |
| `@colyseus/auth` email/password + Google (existing upgrade paths) | Anonymous → registered after Login upgrade (SC-AUTH-15); no new server API |
| Room `tourist` (`MyRoom.onAuth` → `JWT.verify`, any valid JWT incl. anonymous) | Game join after auto-guest |
| Room `lobby` (built-in `LobbyRoom` listing) | Listing only — **no** custom `onAuth` / no reconnect hold; SPA still needs guest JWT for protected routes + later `tourist` join |
| `GET /api/content/packs` (`listCatalog`) + `GET /api/content/maps` (`listMaps`) with `auth.middleware()` | JWT required (incl. anonymous); guest sees in-catalog subset; identity filters remain **client chrome only** (server does not add filter query params) |
| Pack/map create & editor gates (`requireVerifiedEditor` → `registered_user_required` when `anonymous === true`) | Guest create rights stay denied — do not widen |

Out of scope remains: server-side filter query params, auth API changes, OAuth providers, DB migrations.
