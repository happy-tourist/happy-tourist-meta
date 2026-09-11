---
name: client-work-with-auth
description: >-
  Use when adding, changing, reviewing, or debugging client authentication:
  LoginPage register/login/anonymous guest/Google one-click, Pinia auth store,
  client.auth from @colyseus/sdk, colyseus-auth-token, whenReady, router
  requiresAuth / guest guards, or logout in this Quasar Vue 3 checkers SPA.
---

# Work With Auth

Use this skill for **authentication** in the happy-tourist client (`happy-tourist.github.io`).

Auth is **token-based Colyseus Auth** via `client.auth` from `@colyseus/sdk`. The token is persisted under `colyseus-auth-token`. There is **no** cookie session, XSRF, captcha, or Qrator layer.

**Important:** `@colyseus/auth` is a **server-side** package. The client never imports it; all client auth goes through `client.auth` on the SDK `Client` from `src/boot/colyseus.ts`.

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Boot | `src/boot/colyseus.ts` | `Client` singleton (`VITE_COLYSEUS_URL`); `$colyseus` on app |
| Store | `src/stores/auth.ts` | Pinia setup store: register / login / loginAnonymously / loginWithGoogle / logout / whenReady |
| Page | `src/pages/LoginPage.vue` | Email/password register↔login + anonymous guest + one Google button |
| Router | `src/router/index.ts` | `beforeEach` awaits `whenReady()`; `requiresAuth` / `guest` |
| Routes | `src/router/routes.ts` | `/login` `guest`; `/lobby`, `/game/:roomId` `requiresAuth` |
| Token | SDK storage key `colyseus-auth-token` | Persisted by `@colyseus/sdk` Auth; synced via `onChange` |

Prefer Colyseus I/O in the Pinia auth store, not scattered `client.auth.*` calls in components.

## End-To-End Auth Flow

```text
App boot → Client(VITE_COLYSEUS_URL)
        │
        ▼
  auth store: client.auth.onChange → token + user; ready = true
        │
        ▼
  Router beforeEach → await whenReady()
        │
        ├─ meta.requiresAuth && !isAuthenticated → /login
        ├─ meta.guest && isAuthenticated → /lobby
        └─ else → proceed
        │
        ▼
  [/login] LoginPage
        │
        ├─ register → client.auth.registerWithEmailAndPassword
        ├─ login    → client.auth.signInWithEmailAndPassword
        ├─ guest    → client.auth.signInAnonymously
        └─ Google   → client.auth.signInWithProvider('google')  (store: loginWithGoogle)
                │
                ▼
          onChange updates token/user → isAuthenticated
                │
                ▼
          replace(redirect || /lobby)
        │
        ▼
  [/lobby] [/game/:roomId] — protected; SDK attaches token to room/HTTP
        │
        └─ logout → client.auth.signOut() → onChange clears session → /login
```

### Step-by-step (happy path)

1. Quasar boots; `colyseus` boot creates shared `Client`.
2. Auth store subscribes to `client.auth.onChange`. SDK restores token from `colyseus-auth-token` and fetches userdata → first `onChange` sets `ready`.
3. Any navigation awaits `whenReady()` before applying guards.
4. Unauthenticated user hitting `/lobby` or `/game/...` is sent to `/login`.
5. On LoginPage: register, email/password login, anonymous guest, or Google one-click via store actions.
6. Success → `onChange` fills `token`/`user` → page `replace`s to `?redirect` or `/lobby`.
7. Authenticated user opening `/login` (`meta.guest`) is redirected to `/lobby`.

## Pinia Auth Store

File: `src/stores/auth.ts` (setup store).

| State / computed | Meaning |
|------------------|---------|
| `user` | SDK userdata (`id`, `email`, `name`, `anonymous`, optional `theme` `light`\|`dark`\|null from userdata, …) |
| `token` | Auth token string or `null` |
| `loading` / `error` | In-flight flag + last error message |
| `ready` | First `onChange` completed (session restore settled) |
| `isAuthenticated` | `Boolean(token && user)` |
| `displayName` | `name` → `email` → `'Гость'` if anonymous → `'Игрок'` |

Actions (all set `loading`/`error`; rethrow after storing message):

| Action | SDK call |
|--------|----------|
| `register(email, password, options?)` | `client.auth.registerWithEmailAndPassword` |
| `login(email, password)` | `client.auth.signInWithEmailAndPassword` |
| `loginAnonymously(options?)` | `client.auth.signInAnonymously` |
| `loginWithGoogle()` | `client.auth.signInWithProvider('google')` |
| `logout()` | `client.auth.signOut()` |
| `whenReady()` | Resolves when `ready` (or immediately if already ready) |

```ts
client.auth.onChange((data) => {
  token.value = data.token ?? null;
  user.value = (data.user as AuthUser | null) ?? null;
  ready.value = true;
  resolveReady?.();
});
```

Do not invent a parallel session flag. Trust `onChange` + `isAuthenticated`.
Chrome Dark preference is owned by `stores/theme` (applies `user.theme` for
registered users); auth store must not call `Dark` or `POST /api/theme`.

## Login Page Patterns

File: `src/pages/LoginPage.vue`.

- Local `isRegister` toggles register ↔ login UI (not Vuex/Pinia).
- Register shows optional display name → passed as `{ name }` options.
- Email/password validation: required email; password min 6 chars.
- Errors: `q-banner` bound to `auth.error`; clear on mode toggle.
- Submit → `auth.register` or `auth.login` → `goAfterLogin()`.
- Guest → `auth.loginAnonymously(options?)` → same redirect.
- Google → one button (`$t('login.google')`) → `auth.loginWithGoogle()` → same redirect; cancel/failure → store `error` + existing `q-banner`. Email/password and guest stay (do not remove).
- Redirect: `route.query.redirect` if string, else `/lobby`.

Catch blocks intentionally empty — store already holds `error`.

## Router Guards

File: `src/router/index.ts`.

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

Route meta (`src/router/routes.ts`):

| Path | Meta |
|------|------|
| `/login` | `guest: true` |
| `/lobby`, `/game/:roomId` | `requiresAuth: true` |

Router mode is **hash** (`/#/login`, `/#/lobby`). Always await `whenReady()` before deciding auth redirects — otherwise a restored token looks logged-out on first paint.

## Client Singleton

File: `src/boot/colyseus.ts`.

```ts
export const client = new Client(import.meta.env.VITE_COLYSEUS_URL);
```

In `<script setup>`, import `client` from `@/boot/colyseus` (or use the auth store). Prefer store actions over calling `client.auth` from pages.

Env: `VITE_COLYSEUS_URL` (also `VITE_API_URL` for HTTP). Local defaults → `localhost:2567`; production → `happy-tourist.duckdns.org`.

## SDK Surface (client)

| Method | Purpose |
|--------|---------|
| `registerWithEmailAndPassword(email, password, options?)` | Register; `options` (e.g. `{ name }`) forwarded to server `onRegisterWithEmailAndPassword` |
| `signInWithEmailAndPassword(email, password)` | Email/password sign-in |
| `signInAnonymously(options?)` | Guest session |
| `signInWithProvider('google')` | Google OAuth one-click (store: `loginWithGoogle`) |
| `signOut()` | Clear session / token |
| `token` / `onChange` | Current token + userdata sync |

Coordinate register options and userdata shape with [`../happy-tourist-server`](../../../../happy-tourist-server) (Colyseus Auth backend). Do not add axios Authorization headers for auth — the SDK owns the token.

## Do / Don't

| Do | Don't |
|----|--------|
| Call auth only via `stores/auth` actions | Scatter `client.auth.*` across many components |
| Await `whenReady()` before guard decisions | Redirect on first tick before `onChange` |
| Use `meta.requiresAuth` / `meta.guest` | Re-implement cookie/XSRF or Bearer axios auth |
| Keep token key as SDK default `colyseus-auth-token` | Import `@colyseus/auth` in the client |
| Surface errors in store `error` + page banner | Swallow failures without setting `error` |
| Pass `{ name }` options for register/anonymous when UI has a name | Invent captcha / challenge / SMS verify steps |
| Sync contract changes with `happy-tourist-server` | Change auth endpoints only on the client |

## Change Checklist

When touching auth:

1. Which piece? store / LoginPage / router guard / colyseus boot / server Auth hooks.
2. Keep all client auth through `client.auth` (`@colyseus/sdk`), not `@colyseus/auth`.
3. Preserve `onChange` → `token`/`user`/`ready` and `whenReady()` for the router.
4. Preserve `isAuthenticated === Boolean(token && user)`.
5. Keep `requiresAuth` → `/login` and `guest` → `/lobby` behavior.
6. Update LoginPage + store together; keep anonymous guest and Google one-click paths.
7. If register/userdata options change, coordinate with `happy-tourist-server`.
8. Prefer store `error` + existing `q-banner` over a new toast layer.
