## Purpose

Восстановление доступа к аккаунту email/password: запрос письма со ссылкой сброса и установка нового пароля без знания старого.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-RESET-01 | pending (server: forgot-password → mail) |
| SC-RESET-02 | pending (server: reset-password form/token) |
| SC-RESET-03 | pending (server: login with new password) |
| SC-RESET-04 | pending (client: Login link to forgot flow) |
| SC-RESET-05 | pending (client: forgot request UX + error banner) |

## ADDED Requirements

### Requirement: Request password reset email

An unauthenticated user MUST be able to request a password-reset email for an account email address. When the address belongs to a known email/password account and mail delivery is configured, the system SHALL send an email containing a reset link served by the auth API. The client MUST provide a dedicated forgot-password flow reachable from the Login screen.

#### Scenario [SC-RESET-01]: Known email receives reset mail

- **GIVEN** an email/password account exists for address E
- **WHEN** the user submits a forgot-password request for E
- **THEN** a password-reset email is sent to E with a working reset link

#### Scenario [SC-RESET-04]: Login exposes forgot-password entry

- **GIVEN** the user is on the Login screen and is not authenticated
- **WHEN** the user chooses the forgot-password control
- **THEN** the client opens the forgot-password flow
- **AND** the user can submit an email address for reset

#### Scenario [SC-RESET-05]: Forgot request feedback

- **GIVEN** the user is on the forgot-password flow
- **WHEN** the request completes (success or failure)
- **THEN** the UI shows a clear outcome using the existing auth error/success feedback pattern
- **AND** the user can return to Login

### Requirement: Reset password via email link

Opening a valid password-reset link MUST allow the user to set a new password through the auth API reset flow (built-in HTML form is acceptable). After a successful reset, signing in with the email and the new password MUST succeed; the previous password MUST no longer authenticate.

#### Scenario [SC-RESET-02]: Valid link sets new password

- **GIVEN** the user received a valid password-reset link for their account
- **WHEN** the user submits a new password through the reset flow
- **THEN** the account password is updated
- **AND** the auth API shows a successful reset outcome

#### Scenario [SC-RESET-03]: Login works with new password only

- **GIVEN** the user has successfully reset their password to P_new (previous password was P_old)
- **WHEN** the user signs in with email and P_new
- **THEN** authentication succeeds
- **AND** signing in with email and P_old fails
