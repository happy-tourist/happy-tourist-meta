## Purpose

Восстановление доступа к аккаунту email/password: запрос письма со ссылкой сброса и установка нового пароля без знания старого. Письмо и HTML-форма сброса — на русском; клиентский feedback после запроса письма напоминает проверить папку «Спам».

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-RESET-01 | covered (server: forgot-password → RU mail) |
| SC-RESET-02 | covered (server: reset-password form/token RU) |
| SC-RESET-03 | covered-by-reuse (login with new password via Colyseus auth) |
| SC-RESET-04 | covered (client: Login link to forgot flow) |
| SC-RESET-05 | covered (client: forgot request UX + spam hint + error banner) |
| SC-RESET-06 | covered (server: reset email subject+body in Russian) |

## ADDED Requirements

### Requirement: Request password reset email

An unauthenticated user MUST be able to request a password-reset email for an account email address. When the address belongs to a known email/password account and mail delivery is configured, the system SHALL send an email containing a reset link served by the auth API. The client MUST provide a dedicated forgot-password flow reachable from the Login screen. After a successful request the client feedback MUST be in Russian and MUST tell the user to check their mailbox **and the spam folder**.

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
- **WHEN** the request completes successfully
- **THEN** the UI shows a clear Russian outcome that the mail was sent and to check inbox and spam
- **AND** the user can return to Login
- **WHEN** the request fails
- **THEN** the UI shows a clear error using the existing auth error feedback pattern
- **AND** the user can return to Login

### Requirement: Reset email and form are in Russian

The password-reset email subject and body MUST be in Russian (same intent as the prior English template). The auth API reset-password HTML form and its success/error outcomes shown to the user MUST be in Russian (including expired or invalid token where applicable).

#### Scenario [SC-RESET-06]: Reset mail is Russian

- **GIVEN** the system sends a password-reset email
- **WHEN** the message is composed
- **THEN** the subject and HTML body are in Russian
- **AND** the body includes a working reset link

#### Scenario [SC-RESET-02]: Valid link sets new password

- **GIVEN** the user received a valid password-reset link for their account
- **WHEN** the user opens the reset flow
- **THEN** the auth API form copy is in Russian
- **WHEN** the user submits a new password through the reset flow
- **THEN** the account password is updated
- **AND** the auth API shows a successful reset outcome in Russian

### Requirement: Reset password via email link

Opening a valid password-reset link MUST allow the user to set a new password through the auth API reset flow (built-in HTML form is acceptable). After a successful reset, signing in with the email and the new password MUST succeed; the previous password MUST no longer authenticate.

#### Scenario [SC-RESET-03]: Login works with new password only

- **GIVEN** the user has successfully reset their password to P_new (previous password was P_old)
- **WHEN** the user signs in with email and P_new
- **THEN** authentication succeeds
- **AND** signing in with email and P_old fails
