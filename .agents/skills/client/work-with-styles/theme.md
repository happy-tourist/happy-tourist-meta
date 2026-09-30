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

`AuthUser` may include optional `theme?: string | null` from userdata (login /
display); restore after reload must use **GET** `/api/theme`, not JWT claims
alone. Anonymous sessions must not GET/POST `/api/theme` (server rejects).
Colyseus HTTP for theme belongs in the theme store only (`colyseus-client` /
stores pattern).

Muted secondary chrome text uses global `.text-muted` (with `.body--dark`
override) instead of hardcoding `text-grey-7` on Login / Lobby / Game chrome.
