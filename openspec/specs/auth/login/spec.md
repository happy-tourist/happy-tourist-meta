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
| SC-AUTH-07 | pending (needs Google JWT → join `checkers` smoke outside tasks) |

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
- **WHEN** the client joins or creates a `checkers` room
- **THEN** the server accepts the connection through the existing JWT room auth gate
- **AND** does not require a separate Google-specific room credential
