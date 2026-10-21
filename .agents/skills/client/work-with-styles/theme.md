# Quasar Dark / theme chrome

Read with the [core styles skill](SKILL.md).

## Runtime Light / Dark (Quasar Dark)

Preference values: `light` | `dark` | unset (`null`). Unset → `Dark.set('auto')`
(follows OS). After an explicit choice, toggle is light ↔ dark only (no return
to auto in v1).

| Concern | Behavior | Code |
|---------|----------|------|
| Early apply | Boot reads `localStorage` key `ht-theme` and calls `Dark.set` | `src/boot/theme.ts` |
| Guest / signed out | Persist explicit choice in `localStorage` only | `writeStoredTheme` / `readStoredTheme` |
| Registered restore | On `auth.ready` / identity change: async `client.http.get('/api/theme')` and apply; **not** JWT `user.theme` alone after reload (claims may be stale); align device copy (`writeStoredTheme` or `clearStoredTheme` when profile unset → auto); ignore stale GET via generation counter; **do not** replace `auth.user` after GET (theme lives in theme store — SC-THEME-10); GET fail → keep boot/`localStorage`, do not fall back to JWT-only | `stores/theme.ts` `syncFromAuthUser` |
| Registered save | Toggle → `client.http.post('/api/theme', { body: { theme } })`; also write `localStorage` (flash / device copy); optional in-memory `auth.user.theme` patch after POST (watch must not depend on `user.theme`); failures → store `error` + `q-banner` in `App.vue` | `stores/theme.ts` `toggle` |
| Header control | Shared `q-header` + `q-btn` icons `dark_mode` / `light_mode` | `App.vue` |
| Auth wiring | `App.vue`: `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` — stable multi-source (not `() => […]` which allocates a new array each run); not on `user.theme` → `theme.syncFromAuthUser` | do not call HTTP from page templates |
| Board | Unchanged — tourist tile fills are **fixed colors**, not app Dark mode | `GamePage.vue` scoped CSS (`.tourist-board` / `.tile-*`) |
| Shell page bg (SC-BRAND-21) | Fixed cover on `App.vue` `q-layout.app-page-bg`; asset = theme (`body.body--dark`) × viewport (`max-width: 599px` = mobile); four JPGs under `src/assets/packs/`; transparent Quasar page canvas under the art | `src/css/app.scss` `.app-page-bg` |
| Catalog chrome tokens (SC-BRAND-25) | Soft blue/navy header + muted crumb ink + catalog list title/create band — **Quasar Dark only** (no second theme engine). Logo mark ~44px (40–48); **no** wordmark (D4) | `src/css/app.scss` (`.app-page-bg > .q-header`, `.app-breadcrumbs`, `.catalog-list-*`); logo height in `App.vue` scoped |
| Crumbs flex (SC-BRAND-23 / D7) | When breadcrumbs show, `q-page-container--with-crumbs`: flex column + `min-height: 100vh`; child `.q-page` `min-height: 0 !important` so Quasar viewport−header min-height does not stack under crumbs (phantom scroll on short packs/maps). Lobby (no crumbs) keeps Quasar defaults. Prefer flex/min-height — **not** `overflow: hidden` on `body` | `App.vue` class bind + `src/css/app.scss` |
| Board holes (SC-THEME-07) | Removed cells stay transparent — atmospheric shell art is the page background after SC-BRAND-21 | `GamePage.vue` `.tile--removed` |

### Catalog list chrome (SC-BRAND-25/26)

Shared classes in `app.scss` (packs + maps list pages):

| Class | Role |
|-------|------|
| `.catalog-list-page` | `q-page` host; `.pack-card-grid` keeps SC-PACK-285 translucent underlay (SC-BRAND-28 / D10 — **no** solid cream/navy override) |
| `.catalog-list-title-band` | Title + subtitle + create CTA row (gap ~12; mb ~20 guest / filters tighten registered) |
| `.catalog-list-title` / `.catalog-list-subtitle` | ~34px / ~14px; light navy / muted; dark white / `#a8c0d0` |
| create CTA | Lobby Quasar primary (`color="primary" icon="add"`) — **no** `unelevated` / custom `.catalog-list-create` sizing (SC-BRAND-27 / D9) |
| `.catalog-list-filters` | Registered filter row spacing (hidden entirely for guest — SC-PACK-286 / SC-MAP-69) |

Header under `.app-page-bg`: light `#d6eafb` / ink `#1c2e44`; dark `#172236` / `#a8c0d0`; toolbar `min-height` ~56. Crumb strip: muted blue ink (`#5a8fb8` / dark `#8aa4bc`), pad ~8/4, ~13px — size/color in `app.scss`, not only `App.vue` scoped.

`AuthUser` may include optional `theme?: string | null` from userdata (login /
display); restore after reload must use **GET** `/api/theme`, not JWT claims
alone. Anonymous sessions must not GET/POST `/api/theme` (server rejects).
Colyseus HTTP for theme belongs in the theme store only (`colyseus-client` /
stores pattern).

Muted secondary chrome text uses global `.text-muted` (with `.body--dark`
override) instead of hardcoding `text-grey-7` on Login / Lobby / Game chrome.
