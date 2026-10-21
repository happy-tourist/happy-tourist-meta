# Router guards (`authGuard.ts`)

Read with the [core auth skill](SKILL.md) when changing navigation auth,
silent guest entry, Login upgrade bounce, or staff/admin route meta.

## Files

| Path | Role |
|------|------|
| `src/router/authGuard.ts` | `authBeforeEach`; shared `silentGuestInFlight`; `resetSilentGuestInFlightForTests` (tests only) |
| `src/router/index.ts` | `Router.beforeEach(authBeforeEach)` |
| `src/router/routes.ts` | `meta.requiresAuth` / `guest` / `requiresStaff` / `requiresAdmin` |

## Pattern

```ts
// Shared in-flight silentGuestInFlight dedupes concurrent navigations.
export async function authBeforeEach(to: RouteLocationNormalized) {
  const auth = useAuthStore();
  await auth.whenReady();

  if (to.meta.requiresAuth && !auth.isAuthenticated) {
    if (to.meta.requiresStaff || to.meta.requiresAdmin) {
      return { path: '/login', query: { redirect: to.fullPath } };
    }
    try {
      await auth.loginAnonymously(); // via shared in-flight promise
      if (!auth.isAuthenticated) {
        return { path: '/login', query: { redirect: to.fullPath } };
      }
    } catch {
      // error already in store (SC-AUTH-11b)
      return { path: '/login', query: { redirect: to.fullPath } };
    }
  }

  if (to.meta.guest && auth.isAuthenticated) {
    const allowAnonymousOnLogin = to.name === 'login' && Boolean(auth.user?.anonymous);
    if (!allowAnonymousOnLogin) {
      return '/lobby';
    }
  }

  // Client nav gating only — server still enforces (SC-ROLE-08).
  if (to.meta.requiresAdmin && !auth.isAdmin) {
    return '/support';
  }

  if (to.meta.requiresStaff && !auth.isStaff) {
    return '/support';
  }
}
```

## Cases

| Case | Behavior |
|------|----------|
| Ordinary `requiresAuth`, no session | Silent `loginAnonymously` (shared in-flight); stay on `to` (SC-AUTH-11). Fail / no session after resolve → `/login?redirect=` + store error (SC-AUTH-11b). Includes `/account` / `/support` — page-level rules unchanged; do not special-case in the guard. |
| `requiresStaff` / `requiresAdmin`, no session | `/login?redirect=fullPath`; **no** anonymous (SC-AUTH-12) |
| Login + anonymous session | Stay (upgrade; SC-AUTH-13) |
| Login + registered session | Bounce → `/lobby` |
| Other `meta.guest` (e.g. forgot-password) + any authenticated | Bounce → `/lobby` (incl. anonymous) |
| Authenticated but not staff/admin on staff/admin route | `/support` |

## Route meta

| Path | Meta |
|------|------|
| `/login`, `/forgot-password` | `guest: true` |
| `/confirm-email`, `/reset-password` | public (neither `guest` nor `requiresAuth`) |
| `/lobby`, `/account`, `/support`, `/support/:id`, `/game/:roomId`, packs/maps | `requiresAuth: true` |
| `/support/staff` | `requiresAuth` + `requiresStaff` |
| `/admin/users` | `requiresAuth` + `requiresAdmin` |

Router mode is **hash**. Always await `whenReady()` before deciding auth redirects — otherwise a restored token looks logged-out on first paint. Staff/admin meta gates **nav only** — HTTP still enforces on the server. Do **not** document ordinary `requiresAuth` → `/login` as the only path.

## Tests

- `src/router/__tests__/authGuard.test.ts` — silent guest stay/fail, staff/admin → Login + `redirect`, Login anonymous stay / registered bounce, concurrent in-flight dedupe (`resetSilentGuestInFlightForTests`).
- `src/pages/__tests__/LoginPageSilentGuest.test.ts` — hide «Войти как гость» when already anonymous; upgrade submit keeps session path (SC-AUTH-14/15).
