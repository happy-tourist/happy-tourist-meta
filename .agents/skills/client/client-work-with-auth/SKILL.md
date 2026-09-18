---
name: client-work-with-auth
description: >-
  Use when adding, changing, reviewing, or debugging client authentication:
  LoginPage register/login/anonymous guest/Google one-click, forgot-password,
  AccountPage (confirm button + change email), session verify modal → cabinet,
  Pinia auth store, client.auth from @colyseus/sdk, colyseus-auth-token,
  whenReady, router requiresAuth / guest guards, or logout in this Quasar Vue 3
  tourist SPA. No SPA confirm/reset pages; no auto mail on register.
---

# Work With Auth

Use this skill for **authentication** in the happy-tourist client (`happy-tourist.github.io`).

Auth is **token-based Colyseus Auth** via `client.auth` from `@colyseus/sdk`. The token is persisted under `colyseus-auth-token`. There is **no** cookie session, XSRF, captcha, or Qrator layer.

**Important:** `@colyseus/auth` is a **server-side** package. The client never imports it; all client auth goes through `client.auth` on the SDK `Client` from `src/boot/colyseus.ts`. Confirm/reset **HTML** lives on the API — do **not** add SPA confirm/reset pages.

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Boot | `src/boot/colyseus.ts` | `Client` singleton (`VITE_COLYSEUS_URL`); `$colyseus` on app |
| Store | `src/stores/auth.ts` | Pinia setup store: register / login / loginAnonymously / loginWithGoogle / logout / forgotPassword / sendEmailConfirmation / changeEmail / whenReady |
| Login | `src/pages/LoginPage.vue` | Email/password register↔login + anonymous guest + Google; link to forgot; **no** post-register «письмо уже ушло» |
| Forgot | `src/pages/ForgotPasswordPage.vue` | Request reset email; `guest` route; banner feedback; back to Login |
| Cabinet | `src/pages/AccountPage.vue` | Current email; confirm button next to email if `!emailVerified`; change-email UI; success dialog after send |
| App shell | `src/App.vue` | Session reminder modal → cabinet (once per `sessionStorage`); skip anonymous |
| Router | `src/router/index.ts` | `beforeEach` awaits `whenReady()`; `requiresAuth` / `guest` |
| Routes | `src/router/routes.ts` | `/login`, `/forgot-password` `guest`; `/lobby`, `/account`, `/game/:roomId` `requiresAuth` |
| Token | SDK storage key `colyseus-auth-token` | Persisted by `@colyseus/sdk` Auth; synced via `onChange` |

Prefer Colyseus I/O in the Pinia auth store, not scattered `client.auth.*` / `client.http.*` calls in components.

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
        ├─ register → client.auth.registerWithEmailAndPassword  (NO auto confirm mail)
        ├─ login    → client.auth.signInWithEmailAndPassword
        ├─ guest    → client.auth.signInAnonymously
        ├─ Google   → client.auth.signInWithProvider('google')
        └─ forgot link → /forgot-password
                │
                ▼
          onChange updates token/user → isAuthenticated
                │
                ▼
          replace(redirect || /lobby)
        │
        ▼
  Soft verify (registered, !emailVerified, not anonymous):
        App.vue session modal once → CTA to /account
        │
        ▼
  [/account] AccountPage
        ├─ sendEmailConfirmation → POST /api/auth/send-email-confirmation → dialog
        └─ changeEmail → POST /api/auth/email (resets verified; no auto mail)
        │
        ▼
  Confirm/reset links open **server HTML** on API host → redirect lobby
        │
        └─ logout → client.auth.signOut() → onChange clears session → /login
```

### Step-by-step (happy path)

1. Quasar boots; `colyseus` boot creates shared `Client`.
2. Auth store subscribes to `client.auth.onChange`. SDK restores token from `colyseus-auth-token` and fetches userdata → first `onChange` sets `ready`.
3. Any navigation awaits `whenReady()` before applying guards.
4. Unauthenticated user hitting `/lobby`, `/account`, or `/game/...` is sent to `/login`.
5. On LoginPage: register, email/password login, anonymous guest, or Google one-click via store actions. Register does **not** imply mail was sent — no post-register mail hint.
6. Success → `onChange` fills `token`/`user` → page `replace`s to `?redirect` or `/lobby`.
7. Authenticated user opening `/login` or `/forgot-password` (`meta.guest`) is redirected to `/lobby`.
8. Unverified registered users may see a one-shot session modal pointing to the cabinet; confirm mail is sent only from the cabinet button.

## Pinia Auth Store

File: `src/stores/auth.ts` (setup store).

| State / computed | Meaning |
|------------------|---------|
| `user` | SDK userdata (`id`, `email`, `name`, `anonymous`, `emailVerified?`, optional `theme` `light`\|`dark`\|null, …) |
| `token` | Auth token string or `null` |
| `loading` / `error` | In-flight flag + last error message |
| `ready` | First `onChange` completed (session restore settled) |
| `isAuthenticated` | `Boolean(token && user)` |
| `needsEmailVerification` | Registered (has email, not anonymous) and `emailVerified !== true` |
| `displayName` | `name` → `email` → `'Гость'` if anonymous → `'Игрок'` |

Actions (all set `loading`/`error`; rethrow after storing message):

| Action | Call |
|--------|------|
| `register(email, password, options?)` | `client.auth.registerWithEmailAndPassword` — **no** confirm mail |
| `login(email, password)` | `client.auth.signInWithEmailAndPassword` |
| `loginAnonymously(options?)` | `client.auth.signInAnonymously` |
| `loginWithGoogle()` | `client.auth.signInWithProvider('google')` |
| `logout()` | `client.auth.signOut()` |
| `forgotPassword(email)` | `client.auth.sendPasswordResetEmail(email)` |
| `sendEmailConfirmation()` | `client.http.post('/api/auth/send-email-confirmation')` |
| `changeEmail(email)` | `client.http.post('/api/auth/email', { body: { email } })` then apply returned token/user + `refreshUserData` |
| `refreshUserData()` | `client.auth.getUserData` into store |
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
Chrome Dark preference is owned by `stores/theme` (`GET /api/theme` restore for
registered users — not JWT `user.theme` alone; guest `localStorage`); auth store
must not call `Dark` or theme HTTP.

## Login Page Patterns

File: `src/pages/LoginPage.vue`.

- Local `isRegister` toggles register ↔ login UI (not Vuex/Pinia).
- Register shows optional display name → passed as `{ name }` options.
- Email/password validation: required email; password min 6 chars.
- Errors: `q-banner` bound to `auth.error`; clear on mode toggle.
- Submit → `auth.register` or `auth.login` → `goAfterLogin()`.
- Guest → `auth.loginAnonymously(options?)` → same redirect.
- Google → one button (`$t('login.google')`) → `auth.loginWithGoogle()` → same redirect; cancel/failure → store `error` + existing `q-banner`. Email/password and guest stay (do not remove).
- Forgot link → route `forgot-password` (`$t('auth.forgotLink')`).
- **Do not** show a post-register hint that mail was already sent (confirm is button-only in cabinet).
- Redirect: `route.query.redirect` if string, else `/lobby`.

Catch blocks intentionally empty — store already holds `error`.

## Forgot Password Page

File: `src/pages/ForgotPasswordPage.vue`.

- `guest` route `/forgot-password`.
- Form: email → `auth.forgotPassword` → success banner; link back to Login.
- Reset link itself is **server HTML** on the API — no client reset page.

## Account (Cabinet) Page

File: `src/pages/AccountPage.vue` (`requiresAuth`, `/account`).

- Cabinet is for **registered non-anonymous** users only: on mount, if `auth.user?.anonymous` → `router.replace({ name: 'lobby' })` (design D7). Nav links in App/Lobby already hide for anonymous.
- Show current email; if `emailVerified` — caption confirmed.
- If registered and `!emailVerified` — confirm button **next to** the email → `sendEmailConfirmation` → dialog «письмо отправлено, проверьте почту».
- Change-email form → `changeEmail` (server resets verified; **no** auto-send).
- Navigation from App / lobby into cabinet.

## Session Verify Reminder

File: `src/App.vue`.

- Once per browser session (`sessionStorage` key); skip anonymous and guest routes.
- Copy: confirm from the **personal cabinet** (+ CTA to `/account`).
- Do **not** claim that mail was already sent.

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
| `/login`, `/forgot-password` | `guest: true` |
| `/lobby`, `/account`, `/game/:roomId` | `requiresAuth: true` |

Router mode is **hash** (`/#/login`, `/#/lobby`, `/#/account`). Always await `whenReady()` before deciding auth redirects — otherwise a restored token looks logged-out on first paint.

**No** client routes for confirm-email or reset-password — those are API HTML pages.

## Client Singleton

File: `src/boot/colyseus.ts`.

```ts
export const client = new Client(import.meta.env.VITE_COLYSEUS_URL);
```

In `<script setup>`, import `client` from `@/boot/colyseus` (or use the auth store). Prefer store actions over calling `client.auth` / `client.http` from pages.

Env: `VITE_COLYSEUS_URL` (also `VITE_API_URL` for HTTP). Local defaults → `localhost:2567`; production → `api.happy-tourist.ru`.

## SDK Surface (client)

| Method | Purpose |
|--------|----------|
| `registerWithEmailAndPassword(email, password, options?)` | Register; `options` (e.g. `{ name }`) forwarded to server; **does not** send confirm mail |
| `signInWithEmailAndPassword(email, password)` | Email/password sign-in |
| `signInAnonymously(options?)` | Guest session |
| `signInWithProvider('google')` | Google OAuth one-click (store: `loginWithGoogle`) |
| `sendPasswordResetEmail(email)` | Forgot-password request (store: `forgotPassword`) |
| `signOut()` | Clear session / token |
| `getUserData()` | Refresh userdata (e.g. after confirm elsewhere) |
| `token` / `onChange` | Current token + userdata sync |
| `client.http.post(...)` | Thin authenticated HTTP (send-confirm, change-email) |

Coordinate register options and userdata shape (`email`, `emailVerified`) with [`../happy-tourist-server`](../../../../happy-tourist-server) (Colyseus Auth backend). Do not add axios Authorization headers for auth — the SDK owns the token.

## Do / Don't

| Do | Don't |
|----|--------|
| Call auth only via `stores/auth` actions | Scatter `client.auth.*` / `client.http.*` across many components |
| Await `whenReady()` before guard decisions | Redirect on first tick before `onChange` |
| Use `meta.requiresAuth` / `meta.guest` | Re-implement cookie/XSRF or Bearer axios auth |
| Keep token key as SDK default `colyseus-auth-token` | Import `@colyseus/auth` in the client |
| Surface errors in store `error` + page banner | Swallow failures without setting `error` |
| Confirm mail only from cabinet button + dialog | Auto-mail UX on register or SPA confirm/reset pages |
| Session modal → cabinet for unverified registered users | Show verify modal to anonymous guests |
| Sync contract changes with `happy-tourist-server` | Change auth endpoints only on the client |

## Change Checklist

When touching auth:

1. Which piece? store / LoginPage / ForgotPasswordPage / AccountPage / App modal / router guard / colyseus boot / server Auth hooks.
2. Keep all client auth through `client.auth` / store `http` helpers (`@colyseus/sdk`), not `@colyseus/auth`.
3. Preserve `onChange` → `token`/`user`/`ready` and `whenReady()` for the router.
4. Preserve `isAuthenticated === Boolean(token && user)`.
5. Keep `requiresAuth` → `/login` and `guest` → `/lobby` behavior (incl. forgot + account routes).
6. Update Login / forgot / cabinet + store together; keep anonymous guest and Google one-click paths.
7. Never add SPA pages for confirm-email / reset-password; never imply auto mail on register.
8. If register/userdata options or `emailVerified` change, coordinate with `happy-tourist-server`.
9. Prefer store `error` + existing `q-banner` / dialogs over a new toast layer.
