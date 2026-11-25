# auth/login — delta: Login только по прямой ссылке; читаемые ошибки Google

Базовый канон: `openspec/specs/auth/login/spec.md`. Guard больше не направляет на Login; silent-guest fail и staff/admin без сессии → Lobby без петли; Google OAuth error на Login — читаемый текст.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-AUTH-05 | covered (client — Login banner text from structured OAuth error) |
| SC-AUTH-11 | covered-by-reuse (silent guest stay — unchanged happy path) |
| SC-AUTH-11b | covered (client vitest — fail → Lobby once, no Login, no lobby loop) |
| SC-AUTH-12 | covered (client vitest — staff/admin → Lobby, no anonymous on that hop) |
| SC-AUTH-13 | covered (client vitest — anonymous stays on Login via direct URL) |
| SC-AUTH-15 | covered-by-reuse (post-auth `query.redirect` when present) |
| SC-AUTH-16 | covered (client — OAuth object error → human-readable banner text) |

## MODIFIED Requirements

### Requirement: Google auth failure feedback

If Google OAuth fails, is cancelled, or cannot complete, the system SHALL keep the user unauthenticated on the Login screen and SHALL show an error indication suitable for the existing Login error UI pattern. When the failure payload is a structured object (for example provider `error` / `error_description`), the Login error surface MUST show a human-readable string derived from that payload (prefer `error_description`, else `error`, else a JSON serialization) and MUST NOT display the literal text `[object Object]`.

#### Scenario [SC-AUTH-05]: Google OAuth cancelled or failed

- **GIVEN** the user is on the Login screen and starts Google one-click auth
- **WHEN** the OAuth flow fails or the user cancels it
- **THEN** the user remains unauthenticated
- **AND** the Login screen shows an error indication
- **AND** email/password and anonymous controls remain usable

#### Scenario [SC-AUTH-16]: Structured Google OAuth error is readable on Login

- **GIVEN** the user is on the Login screen and starts Google one-click auth
- **WHEN** OAuth completes with a structured error object (for example `invalid_client` with `error_description`)
- **THEN** the Login error banner shows a human-readable message from that payload
- **AND** the banner MUST NOT show the literal string `[object Object]`

### Requirement: Silent anonymous entry on ordinary protected screens

When an unauthenticated user navigates to a client screen that requires authentication but does **not** require staff or admin role, the client MUST establish an anonymous guest session automatically and MUST keep the user on the requested screen (including lobby, game by room id, packs list, maps list, and other ordinary authenticated surfaces). The client MUST NOT force the Login screen solely because no session existed yet for those screens. If anonymous sign-in fails (for example network or auth API error), the client MUST navigate to the Lobby screen **once** (when the requested screen is not already Lobby) or MUST **remain** on Lobby (when the requested screen is Lobby), MUST surface the failure via the existing auth store `error` path, MUST NOT navigate to the Login screen as part of that failure handling, and MUST NOT perform a further redirect whose only purpose is to send the user to Lobby again.

#### Scenario [SC-AUTH-11]: Unauthenticated lobby visit becomes guest in place

- **GIVEN** the user has no auth session
- **WHEN** the user opens the lobby screen
- **THEN** the client creates an anonymous guest session
- **AND** the user remains on the lobby screen as an authenticated anonymous guest
- **AND** the Login screen is NOT shown as a mandatory gate for that navigation

#### Scenario [SC-AUTH-11b]: Silent guest failure falls back to Lobby

- **GIVEN** the user has no auth session and anonymous sign-in will fail
- **WHEN** the user opens an ordinary protected screen (for example packs list or lobby)
- **THEN** the client ends on the Lobby screen without navigating to Login
- **AND** if the requested screen was already Lobby, the client does not issue another Lobby-only redirect
- **AND** the auth error is available on the existing store error path
- **AND** the client does not leave the user on a non-Lobby protected screen without a session

### Requirement: Staff and admin deep links require Login without auto-guest

When an unauthenticated user navigates to a screen that requires staff or admin role, the client MUST send the user to the Lobby screen (not Login) and MUST NOT create an anonymous guest session as part of that staff/admin navigation hop. The client MUST NOT navigate to the Login screen for that case. (A subsequent Lobby entry MAY still establish anonymous guest per ordinary protected-screen rules.)

#### Scenario [SC-AUTH-12]: Staff or admin deep link without session goes to Lobby

- **GIVEN** the user has no auth session
- **WHEN** the user opens a staff-only moderation screen or an admin-only screen
- **THEN** the client navigates to the Lobby screen
- **AND** the client does not navigate to the Login screen for that hop
- **AND** the client does not establish an anonymous guest session as part of that staff/admin navigation hop

### Requirement: Anonymous guest may open Login to upgrade

An authenticated anonymous guest MUST be able to open the Login screen via a **direct URL** (hash route) for email/password or Google sign-in/register without being redirected away solely because a session already exists. The product MUST NOT require a header login affordance for that upgrade path. Registered (non-anonymous) authenticated users MUST still be kept off the Login screen per existing guest-route behavior. This Login exception MUST NOT weaken bounce-away behavior on other `meta.guest` screens (for example forgot-password): any authenticated session, including anonymous, MUST still be redirected away from those screens as today.

#### Scenario [SC-AUTH-13]: Guest opens Login for upgrade

- **GIVEN** an authenticated anonymous guest on the lobby
- **WHEN** the guest opens the Login screen via a direct Login URL
- **THEN** the Login screen is shown
- **AND** the guest is not bounced back to lobby solely for being authenticated

### Requirement: Return to prior screen after upgrade auth

When an anonymous guest reaches Login with a prior protected return target in the navigation (for example a `redirect` query) and then successfully signs in or registers (email/password or Google), the client MUST navigate back to that prior screen (or its equivalent authenticated route). When Login was opened without a prior protected target (for example cold direct Login, or after logout), the client MUST continue to default to the lobby after success.

#### Scenario [SC-AUTH-15]: Upgrade returns to packs list

- **GIVEN** an anonymous guest who opens Login via a direct URL that includes a return target for the packs list
- **WHEN** the guest successfully signs in with email/password or Google
- **THEN** the client returns the user to the packs list
- **AND** the session is no longer anonymous
