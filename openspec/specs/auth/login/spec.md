# auth/login Specification

## Purpose

Авторизация игрока: вход и регистрация через Google одной кнопкой на экране Login, с тем же JWT-сеансом, что и для email/password и anonymous, включая сохранение после обновления страницы.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-AUTH-01 | pending (needs Google OAuth e2e / ops smoke outside tasks) |
| SC-AUTH-02 | pending (needs Google OAuth e2e / ops smoke outside tasks) |
| SC-AUTH-03 | pending (needs Google OAuth e2e / ops smoke outside tasks) |
| SC-AUTH-04 | covered-by-reuse (existing token restore via SDK `colyseus-auth-token` + `whenReady`) |
| SC-AUTH-05 | pending (needs cancel/fail path check outside tasks; error UI covered by code) |
| SC-AUTH-06 | covered-by-reuse (existing email/password + anonymous paths unchanged) |
| SC-AUTH-07 | pending (needs Google JWT → join `tourist` smoke outside tasks) |
| SC-AUTH-08 | covered (server mocha + client register policy/meter) |
| SC-AUTH-09 | covered (client UX — advisory meter; policy gates submit) |
| SC-AUTH-10 | covered (server mocha + client session displayName) |
| SC-AUTH-11 | covered (client vitest — `authGuard.test.ts` silent guest) |
| SC-AUTH-11b | covered (client vitest — silent guest failure / no-session → Login + redirect) |
| SC-AUTH-12 | covered (client vitest — staff/admin → Login, no anonymous) |
| SC-AUTH-13 | covered (client vitest — anonymous stays on Login; bounced from forgot-password) |
| SC-AUTH-14 | covered (client vitest — `LoginPageSilentGuest.test.ts` guest button hidden) |
| SC-AUTH-15 | covered (client vitest — post-auth `query.redirect`) |

## Requirements

### Requirement: One-click Google sign-in and registration

The system SHALL allow an unauthenticated user on the Login screen to complete both registration and sign-in through Google with a single control. Successful Google authentication MUST issue a JWT session equivalent to email/password or anonymous auth for subsequent protected navigation and room join.

#### Scenario [SC-AUTH-01]: First-time Google user becomes authenticated

- **GIVEN** the user is not authenticated and is on the Login screen
- **WHEN** the user completes Google OAuth via the Google one-click control
- **THEN** the system creates a user account from the Google profile when no matching account exists
- **AND** the client holds a valid auth session (JWT)
- **AND** the user is treated as authenticated for protected screens (for example lobby)

#### Scenario [SC-AUTH-02]: Returning Google user signs in

- **GIVEN** a user account already exists for the Google account’s email from a prior Google sign-in
- **WHEN** the user completes Google OAuth again via the one-click control
- **THEN** the system authenticates that existing account
- **AND** does not create a duplicate account for the same email

### Requirement: Email identity linking for Google

When Google OAuth returns an email that already belongs to an existing non-anonymous account, the system SHALL authenticate that existing account rather than creating a second user for the same email.

#### Scenario [SC-AUTH-03]: Google matches existing email/password account

- **GIVEN** an account already exists with email/password for email E
- **WHEN** the user completes Google OAuth that provides the same email E
- **THEN** the system authenticates the existing account for E
- **AND** does not create a second account for E

### Requirement: Session persistence after Google auth

After a successful Google sign-in, the auth session MUST persist across a full page reload in the same browser until the user signs out or the token becomes invalid, using the same client token persistence mechanism as other Colyseus Auth modes.

#### Scenario [SC-AUTH-04]: Reload keeps Google session

- **GIVEN** the user has successfully signed in with Google and has a valid session
- **WHEN** the user reloads the application
- **THEN** the client restores the auth session without requiring Google OAuth again
- **AND** protected navigation remains available while the session is valid

### Requirement: Google auth failure feedback

If Google OAuth fails, is cancelled, or cannot complete, the system SHALL keep the user unauthenticated on the Login screen and SHALL show an error indication suitable for the existing Login error UI pattern.

#### Scenario [SC-AUTH-05]: Google OAuth cancelled or failed

- **GIVEN** the user is on the Login screen and starts Google one-click auth
- **WHEN** the OAuth flow fails or the user cancels it
- **THEN** the user remains unauthenticated
- **AND** the Login screen shows an error indication
- **AND** email/password and anonymous controls remain usable

### Requirement: Existing auth modes remain available

The Login screen SHALL continue to support email/password register and login and anonymous guest sign-in alongside Google one-click auth. Room authentication MUST continue to accept a valid JWT from any of these modes without a Google-specific room gate.

#### Scenario [SC-AUTH-06]: Email and anonymous still work

- **GIVEN** Google one-click auth is available on the Login screen
- **WHEN** the user signs in with email/password or as an anonymous guest instead
- **THEN** authentication succeeds as before
- **AND** the user can reach protected screens with a valid session

#### Scenario [SC-AUTH-07]: Google JWT joins game room

- **GIVEN** the user is authenticated via Google with a valid JWT
- **WHEN** the client joins or creates a `tourist` room
- **THEN** the server accepts the connection through the existing JWT room auth gate
- **AND** does not require a separate Google-specific room credential

### Requirement: Password policy on email registration

When registering with email and password, the client and server MUST enforce a shared password policy: minimum length **8**; at least one Latin lowercase letter `a-z`; at least one Latin uppercase letter `A-Z`; at least one digit `0-9`; at least one symbol defined as any character that is not a Latin letter and not a digit. Colyseus Auth API floor `min 6` MAY remain, but product accept MUST reject passwords that fail this policy. The **login** (sign-in) form MUST NOT enforce this complexity policy beyond whatever minimum the API already requires. Password strength estimation (meter) is advisory only and MUST NOT be an additional submit gate beyond this policy.

#### Scenario [SC-AUTH-08]: Weak register password rejected

- **GIVEN** the user is on the Login screen in register mode
- **WHEN** the user submits a password that is shorter than 8 or missing a required character class
- **THEN** registration does not succeed
- **AND** the user sees clear feedback that the password does not meet the policy

#### Scenario [SC-AUTH-09]: Strength meter is advisory on register

- **GIVEN** the user is typing a password on the register form
- **WHEN** the password changes
- **THEN** the client shows a colored strength indication
- **AND** submit remains governed by the password policy of SC-AUTH-08, not by a minimum strength score alone

### Requirement: Register display name persists

When a user registers with email/password and provides a display name of at least one character (after trim), the system MUST persist that value as the account display name available to the client session and cabinet. Empty display name after trim MUST NOT be accepted on register when the register UI collects a name.

#### Scenario [SC-AUTH-10]: Register name is stored

- **GIVEN** the user registers with email/password and display name N (trimmed length ≥ 1)
- **WHEN** registration completes successfully
- **THEN** the account display name is N
- **AND** the client session reflects N as the display name (not only email)

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
