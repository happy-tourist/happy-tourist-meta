# auth/login — delta: silent guest entry, Login upgrade for anonymous

Базовый канон: `openspec/specs/auth/login/spec.md`. Change: без сессии на обычных защищённых экранах — automatic anonymous; staff/admin deep-link → Login; already-anonymous MAY open Login; hide «Войти как гость» when already guest; return to prior URL after upgrade auth.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-AUTH-11 | covered (client vitest — `authGuard.test.ts` silent guest) |
| SC-AUTH-11b | covered (client vitest — silent guest failure / no-session → Login + redirect) |
| SC-AUTH-12 | covered (client vitest — staff/admin → Login, no anonymous) |
| SC-AUTH-13 | covered (client vitest — anonymous stays on Login; bounced from forgot-password) |
| SC-AUTH-14 | covered (client vitest — `LoginPageSilentGuest.test.ts` guest button hidden) |
| SC-AUTH-15 | covered (client vitest — post-auth `query.redirect`) |

Related: `ui/branding` (header login affordance); game room `tourist` already accepts anonymous JWT via `MyRoom.onAuth` → `JWT.verify` (no new message/route in this change); lobby listing uses built-in `LobbyRoom` (no custom `onAuth` change); create pack/map guest gates unchanged in `content/packs` / `content/maps`. Silent guest MUST call the existing Colyseus Auth anonymous register path (`signInAnonymously` / `onRegisterAnonymously`) — MUST NOT invent a new server auth endpoint. Catalog list endpoints already require JWT via `auth.middleware()` and already return the in-catalog subset for anonymous.

## ADDED Requirements

### Requirement: Silent anonymous entry on ordinary protected screens

When an unauthenticated user navigates to a client screen that requires authentication but does **not** require staff or admin role, the client MUST establish an anonymous guest session automatically and MUST keep the user on the requested screen (including lobby, game by room id, packs list, maps list, and other ordinary authenticated surfaces). The client MUST NOT force the Login screen solely because no session existed yet for those screens. If anonymous sign-in fails (for example network or auth API error), the client MUST fall back to the Login screen and MUST surface the failure via the existing auth error path (store `error` / Login banner) — MUST NOT leave the user on a protected screen without a session.

#### Scenario [SC-AUTH-11]: Unauthenticated lobby visit becomes guest in place

- **GIVEN** the user has no auth session
- **WHEN** the user opens the lobby screen
- **THEN** the client creates an anonymous guest session
- **AND** the user remains on the lobby screen as an authenticated anonymous guest
- **AND** the Login screen is NOT shown as a mandatory gate for that navigation

#### Scenario [SC-AUTH-11b]: Silent guest failure falls back to Login

- **GIVEN** the user has no auth session and anonymous sign-in will fail
- **WHEN** the user opens an ordinary protected screen (for example lobby)
- **THEN** the client navigates to the Login screen
- **AND** the auth error is available on the existing error surface
- **AND** the user is not left on the protected screen without a session

### Requirement: Staff and admin deep links require Login without auto-guest

When an unauthenticated user navigates to a screen that requires staff or admin role, the client MUST send the user to the Login screen and MUST NOT create an anonymous guest session for that navigation.

#### Scenario [SC-AUTH-12]: Staff or admin deep link without session goes to Login

- **GIVEN** the user has no auth session
- **WHEN** the user opens a staff-only moderation screen or an admin-only screen
- **THEN** the client navigates to the Login screen
- **AND** the Login navigation includes a return target for that requested screen (for example `redirect` query to the requested path)
- **AND** the client does not establish an anonymous guest session as part of that navigation

### Requirement: Anonymous guest may open Login to upgrade

An authenticated anonymous guest MUST be able to open the Login screen (for email/password or Google sign-in/register) without being redirected away solely because a session already exists. Registered (non-anonymous) authenticated users MUST still be kept off the Login screen per existing guest-route behavior. This Login exception MUST NOT weaken bounce-away behavior on other `meta.guest` screens (for example forgot-password): any authenticated session, including anonymous, MUST still be redirected away from those screens as today.

#### Scenario [SC-AUTH-13]: Guest opens Login for upgrade

- **GIVEN** an authenticated anonymous guest on the lobby
- **WHEN** the guest opens the Login screen via the header login affordance
- **THEN** the Login screen is shown
- **AND** the guest is not bounced back to lobby solely for being authenticated

### Requirement: Hide anonymous control when already a guest

When the Login screen is shown to an already authenticated anonymous guest, the client MUST NOT show the «Войти как гость» (anonymous sign-in) control. Unauthenticated visitors on Login MUST still see that control.

#### Scenario [SC-AUTH-14]: Guest button hidden for already-anonymous

- **GIVEN** an authenticated anonymous guest on the Login screen
- **WHEN** the Login screen renders
- **THEN** the anonymous «Войти как гость» control is not shown
- **AND** email/password and Google controls remain available

### Requirement: Return to prior screen after upgrade auth

When an anonymous guest reaches Login from another application screen and then successfully signs in or registers (email/password or Google), the client MUST navigate back to that prior screen (or its equivalent authenticated route). When Login was opened without a prior protected target (for example cold Login, or after logout), the client MUST continue to default to the lobby after success.

#### Scenario [SC-AUTH-15]: Upgrade returns to packs list

- **GIVEN** an anonymous guest on the packs list who opens Login to upgrade
- **WHEN** the guest successfully signs in with email/password or Google
- **THEN** the client returns the user to the packs list
- **AND** the session is no longer anonymous
