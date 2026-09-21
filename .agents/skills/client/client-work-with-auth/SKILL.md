---
name: client-work-with-auth
description: >-
  Use when adding, changing, reviewing, or debugging client authentication:
  LoginPage register/login/anonymous guest/Google one-click, forgot-password,
  ConfirmEmailPage / ResetPasswordPage (SPA + JSON, password policy + meter),
  AccountPage (displayName, change-password, confirm button + change email),
  session verify modal → cabinet, Pinia auth store, client.auth from
  @colyseus/sdk, colyseus-auth-token, whenReady, router requiresAuth / guest /
  requiresStaff / requiresAdmin guards, userdata `role` / `isStaff` / `isAdmin`
  (nav only — server enforces), or logout in this Quasar Vue 3 tourist SPA.
  No auto mail on register; confirm/reset UX is client SPA not API HTML.
---

# Work With Auth

Use this skill for **authentication** in the happy-tourist client (`happy-tourist.github.io`).

Auth is **token-based Colyseus Auth** via `client.auth` from `@colyseus/sdk`. The token is persisted under `colyseus-auth-token`. There is **no** cookie session, XSRF, captcha, or Qrator layer.

**Important:** `@colyseus/auth` is a **server-side** package. The client never imports it; all client auth goes through `client.auth` on the SDK `Client` from `src/boot/colyseus.ts`. Confirm/reset **product UX** is **client SPA** + JSON (`POST /api/auth/confirm-email`, `POST /api/auth/reset-password`) — do **not** rely on Colyseus API HTML pages.

## Map Of Pieces

| Layer | Path | Role |
|-------|------|------|
| Boot | `src/boot/colyseus.ts` | `Client` singleton (`VITE_COLYSEUS_URL`); `$colyseus` on app |
| Store | `src/stores/auth.ts` | Pinia setup store: register / login / loginAnonymously / loginWithGoogle / logout / forgotPassword / confirmEmail / resetPassword / sendEmailConfirmation / changeEmail / updateDisplayName / changePassword / whenReady; exposes `role` / `isStaff` / `isAdmin` / `canChangePassword` from userdata |
| Password policy | `src/lib/passwordPolicy.ts` + `PasswordStrengthMeter.vue` | Client mirror of server rules (≥8 + lower/upper/digit/symbol); zxcvbn meter advisory only; gate Submit on policy (register / reset / change) — not login |
| Login | `src/pages/LoginPage.vue` | Email/password register↔login + anonymous guest + Google; register: name + policy meter; login: no complexity; link to forgot; **no** post-register «письмо уже ушло» |
| Forgot | `src/pages/ForgotPasswordPage.vue` | Request reset email; `guest` route; success without «если существует»; `email_not_found` → RU not-found; back to Login |
| Confirm | `src/pages/ConfirmEmailPage.vue` | Public hash route; **auto** `confirmEmail(token)` on mount (no Confirm button); success → lobby; RU errors |
| Reset | `src/pages/ResetPasswordPage.vue` | Public hash route; same policy + meter → `resetPassword`; success → login; RU errors |
| Cabinet | `src/pages/AccountPage.vue` | displayName form; change-password if `canChangePassword` (hide Google/anonymous); email confirm + change-email; soft-verify (no hard-gate); logout |
| App shell | `src/App.vue` | Session reminder modal → cabinet (once per `sessionStorage`; mark seen **when shown**); skip anonymous + auth-email public routes |
| Router | `src/router/index.ts` | `beforeEach` awaits `whenReady()`; `requiresAuth` / `guest` / `requiresStaff` / `requiresAdmin` (staff/admin → `/support` if denied; **nav only**) |
| Routes | `src/router/routes.ts` | `/login`, `/forgot-password` `guest`; `/confirm-email`, `/reset-password` **public** (no `guest` redirect — logged-in confirm must run); `/lobby`, `/account`, `/support` (+ staff/ticket), `/admin/users`, `/game/:roomId` `requiresAuth` |
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
        ├─ meta.requiresAdmin && !isAdmin → /support
        ├─ meta.requiresStaff && !isStaff → /support
        └─ else → proceed  (confirm-email / reset-password have neither meta)
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
        ├─ updateDisplayName → POST /api/auth/display-name
        ├─ changePassword → POST /api/auth/change-password → logout → /login
        ├─ sendEmailConfirmation → POST /api/auth/send-email-confirmation → dialog
        └─ changeEmail → POST /api/auth/email (resets verified; no auto mail)
        │
        ▼
  Mail links → CLIENT_APP_URL SPA:
        /#/confirm-email?token=… → auto POST /api/auth/confirm-email → /lobby
        /#/reset-password?token=… → form POST /api/auth/reset-password → /login
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
9. Opening a confirm mail link loads `ConfirmEmailPage`, which immediately calls JSON confirm (no second button) and navigates to lobby on success.

## Pinia Auth Store

File: `src/stores/auth.ts` (setup store).

| State / computed | Meaning |
|------------------|---------|
| `user` | SDK userdata (`id`, `email`, `name`, `anonymous`, `emailVerified?`, optional `theme` `light`\|`dark`\|null, optional `role` `user`\|`moderator`\|`admin`, …) |
| `token` | Auth token string or `null` |
| `loading` / `error` | In-flight flag + last error message (may be stable code like `email_not_found` / `token_expired`) |
| `ready` | First `onChange` completed (session restore settled) |
| `isAuthenticated` | `Boolean(token && user)` |
| `needsEmailVerification` | Registered (has email, not anonymous) and `emailVerified !== true` |
| `displayName` | persisted `user.displayName` → `name` → `email` → `'Гость'` if anonymous → `'Игрок'` |
| `canChangePassword` | `user.hasPassword === true` (and registered) — show change-password in cabinet |
| `role` | Product role from userdata (`user` \| `moderator` \| `admin`; default `user`) |
| `isStaff` | `role === 'moderator' \|\| role === 'admin'` — staff support queue nav |
| `isAdmin` | `role === 'admin'` — admin users page nav (SC-ROLE-08) |

Actions (all set `loading`/`error`; rethrow after storing message):

| Action | Call |
|--------|------|
| `register(email, password, options?)` | `client.auth.registerWithEmailAndPassword` — **no** confirm mail |
| `login(email, password)` | `client.auth.signInWithEmailAndPassword` |
| `loginAnonymously(options?)` | `client.auth.signInAnonymously` |
| `loginWithGoogle()` | `client.auth.signInWithProvider('google')` |
| `logout()` | `client.auth.signOut()` |
| `forgotPassword(email)` | `client.auth.sendPasswordResetEmail`; maps `email_not_found` |
| `confirmEmail(token)` | `client.http.post('/api/auth/confirm-email', { body: { token } })` then optional `refreshUserData` |
| `resetPassword(token, password)` | `client.http.post('/api/auth/reset-password', { body: { token, password } })` |
| `sendEmailConfirmation()` | `client.http.post('/api/auth/send-email-confirmation')` |
| `changeEmail(email)` | `client.http.post('/api/auth/email', { body: { email } })` then apply returned token/user + `refreshUserData` |
| `updateDisplayName(displayName)` | `client.http.post('/api/auth/display-name', { body: { displayName } })` then refresh userdata |
| `changePassword(current, next)` | `client.http.post('/api/auth/change-password', …)` then **`logout()`** (tokenVersion bump) |
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
- Register requires display name (trim ≥ 1) → `{ name }` options; password uses shared policy + `PasswordStrengthMeter` (Submit only when policy passes — not zxcvbn score).
- Login: required email; password required only (no complexity / meter).
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
- Form: email → `auth.forgotPassword` → success banner (`auth.forgotSuccess` — RU; mail sent + check spam; **no** «если аккаунт существует» hedge).
- Unknown email: store code `email_not_found` → page local banner `$t('auth.forgotNotFound')` (do not mutate `auth.error` string) (SC-RESET-07).
- Reset link opens **SPA** `/#/reset-password?token=…` on `CLIENT_APP_URL` — not API HTML.

## Confirm Email Page

File: `src/pages/ConfirmEmailPage.vue` (public `/confirm-email`).

- On mount: read `token` from query → `auth.confirmEmail(token)` immediately — **no** Confirm button.
- Success RU banner (`auth.confirmSuccess`), then `router.replace({ name: 'lobby' })` (SC-EMAIL-02/11).
- Expired/invalid → RU banner (`auth.confirmExpired` / `auth.confirmInvalid`).
- Do **not** set `meta.guest` — authenticated users opening the mail link must still confirm.

## Reset Password Page

File: `src/pages/ResetPasswordPage.vue` (public `/reset-password`).

- Form: new password with same policy + meter as register → `auth.resetPassword(token, password)`.
- Success RU banner (`auth.resetSuccess`), then `router.replace({ name: 'login' })` (SC-RESET-02/08).
- Expired / invalid / already-used → RU banner.

## Account (Cabinet) Page

File: `src/pages/AccountPage.vue` (`requiresAuth`, `/account`).

- Cabinet is for **registered non-anonymous** users only: on mount, if `auth.user?.anonymous` → `router.replace({ name: 'lobby' })` (design D7). Nav links in App/Lobby already hide for anonymous.
- Display-name form → `updateDisplayName` (trim, min 1).
- Change-password form (policy + meter) only when `canChangePassword`; hide for Google / anonymous; success → `changePassword` → logout → login.
- Show current email; if `emailVerified` — caption confirmed.
- If registered and `!emailVerified` — confirm button **next to** the email → `sendEmailConfirmation` → dialog `auth.confirmSentDialog` («письмо отправлено, проверьте почту **и папку Спам**»). Soft-verify only — no hard-gate on lobby/game.
- Change-email form → `changeEmail` (server resets verified; **no** auto-send).
- Logout action; navigation from App / lobby into cabinet.

**Language canon:** human-facing auth-email strings (`auth.confirmSentDialog`, `auth.forgotSuccess`, confirm/reset outcomes, cabinet/forgot copy) are **Russian** in `src/i18n/en-US/` (technical locale name). New copy in this contour stays RU; do not require EN.

## Session Verify Reminder

File: `src/App.vue`.

- Once per browser session (`sessionStorage` key); skip anonymous and auth-email public routes (login / forgot / confirm / reset).
- Mark the session key **when the modal is shown** (not only on dismiss/OK), so another tab in the same browser session does not reopen it (SC-EMAIL-08).
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

  // Client nav gating only — server still enforces (SC-ROLE-08).
  if (to.meta.requiresAdmin && !auth.isAdmin) {
    return '/support';
  }

  if (to.meta.requiresStaff && !auth.isStaff) {
    return '/support';
  }
});
```

Route meta (`src/router/routes.ts`):

| Path | Meta |
|------|------|
| `/login`, `/forgot-password` | `guest: true` |
| `/confirm-email`, `/reset-password` | public (neither `guest` nor `requiresAuth`) |
| `/lobby`, `/account`, `/support`, `/support/:id`, `/game/:roomId` | `requiresAuth: true` |
| `/support/staff` | `requiresAuth` + `requiresStaff` |
| `/admin/users` | `requiresAuth` + `requiresAdmin` |

Router mode is **hash** (`/#/login`, `/#/confirm-email`, `/#/reset-password`, `/#/lobby`, `/#/support`). Always await `whenReady()` before deciding auth redirects — otherwise a restored token looks logged-out on first paint. Staff/admin meta gates **nav only** — HTTP still enforces on the server.

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
| `getUserData()` | Refresh userdata (e.g. after confirm) |
| `token` / `onChange` | Current token + userdata sync |
| `client.http.post(...)` | Thin HTTP (send-confirm, change-email, confirm-email, reset-password) |

Coordinate register options and userdata shape (`email`, `emailVerified`) with [`../happy-tourist-server`](../../../../happy-tourist-server) (Colyseus Auth backend). Do not add axios Authorization headers for auth — the SDK owns the token.

## Do / Don't

| Do | Don't |
|----|--------|
| Call auth only via `stores/auth` actions | Scatter `client.auth.*` / `client.http.*` across many components |
| Await `whenReady()` before guard decisions | Redirect on first tick before `onChange` |
| Use `meta.requiresAuth` / `meta.guest` | Re-implement cookie/XSRF or Bearer axios auth |
| Keep token key as SDK default `colyseus-auth-token` | Import `@colyseus/auth` in the client |
| Surface errors in store `error` + page banner | Swallow failures without setting `error` |
| Confirm mail only from cabinet button + dialog | Auto-mail UX on register |
| SPA confirm/reset via JSON store actions | Parse Colyseus API HTML or treat `/auth/confirm-email` HTML as product UX |
| Session modal → cabinet for unverified registered users | Show verify modal to anonymous guests |
| Sync contract changes with `happy-tourist-server` | Change auth endpoints only on the client |

## Change Checklist

When touching auth:

1. Which piece? store / LoginPage / ForgotPasswordPage / ConfirmEmailPage / ResetPasswordPage / AccountPage / App modal / router guard / colyseus boot / server Auth hooks.
2. Keep all client auth through `client.auth` / store `http` helpers (`@colyseus/sdk`), not `@colyseus/auth`.
3. Preserve `onChange` → `token`/`user`/`ready` and `whenReady()` for the router.
4. Preserve `isAuthenticated === Boolean(token && user)`.
5. Keep `requiresAuth` → `/login` and `guest` → `/lobby` behavior; keep confirm/reset **public**.
6. Update Login / forgot / confirm / reset / cabinet + store together; keep anonymous guest and Google one-click paths.
7. Never imply auto mail on register; never document API HTML as the product confirm/reset path.
8. If register/userdata options or `emailVerified` change, coordinate with `happy-tourist-server`.
9. Prefer store `error` + existing `q-banner` / dialogs over a new toast layer.
