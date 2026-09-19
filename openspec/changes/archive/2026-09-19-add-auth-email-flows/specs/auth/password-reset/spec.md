## Purpose

Восстановление доступа к аккаунту email/password: запрос письма со ссылкой сброса и установка нового пароля без знания старого. Ссылка открывает **клиентскую SPA**-форму на `CLIENT_APP_URL` (не HTML на API); сброс через **JSON**; после успеха — Login. Неизвестный email при запросе — явная ошибка «не найден». Письмо и SPA — на русском; после успешной отправки письма — напоминание про «Спам».

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-RESET-01 | covered (server: forgot-password → RU mail; link → client SPA hash) |
| SC-RESET-02 | covered (client SPA reset form + JSON; not API HTML) |
| SC-RESET-03 | covered-by-reuse (login with new password via Colyseus auth) |
| SC-RESET-04 | covered (client: Login link to forgot flow) |
| SC-RESET-05 | covered (forgot success clear + spam; unknown email RU not-found) |
| SC-RESET-06 | covered (server: reset email subject+body in Russian) |
| SC-RESET-07 | covered (unknown email → explicit not-found UX) |
| SC-RESET-08 | covered (reset link = client hash route; success → login) |

## ADDED Requirements

### Requirement: Request password reset email

An unauthenticated user MUST be able to request a password-reset email for an account email address. When the address belongs to a known email/password account and mail delivery is configured, the system SHALL send an email containing a reset link to the **client SPA** reset route. The client MUST provide a dedicated forgot-password flow reachable from the Login screen. After a successful request the client feedback MUST be in Russian, MUST state that the mail was sent (not a soft “if the account exists” hedge), and MUST tell the user to check their mailbox **and the spam folder**. When no account exists for the submitted address, the client MUST show a clear Russian error that the account was not found (so the user can check the address), using the existing auth error feedback pattern.

#### Scenario [SC-RESET-01]: Known email receives reset mail

- **GIVEN** an email/password account exists for address E
- **WHEN** the user submits a forgot-password request for E
- **THEN** a password-reset email is sent to E with a working reset link to the client SPA reset route

#### Scenario [SC-RESET-04]: Login exposes forgot-password entry

- **GIVEN** the user is on the Login screen and is not authenticated
- **WHEN** the user chooses the forgot-password control
- **THEN** the client opens the forgot-password flow
- **AND** the user can submit an email address for reset

#### Scenario [SC-RESET-05]: Forgot request feedback

- **GIVEN** the user is on the forgot-password flow
- **WHEN** the request completes successfully for a known account
- **THEN** the UI shows a clear Russian outcome that the mail was sent and to check inbox and spam
- **AND** the user can return to Login
- **WHEN** the request fails for a reason other than unknown email
- **THEN** the UI shows a clear error using the existing auth error feedback pattern
- **AND** the user can return to Login

#### Scenario [SC-RESET-07]: Unknown email is explicit not-found

- **GIVEN** the user is on the forgot-password flow
- **AND** no email/password account exists for the submitted address
- **WHEN** the user submits the forgot-password request
- **THEN** the UI shows a clear Russian message that the account was not found
- **AND** the UI does not claim that a reset email was sent
- **AND** the user can return to Login

### Requirement: Reset email is in Russian; reset UX is client SPA

The password-reset email subject and body MUST be in Russian (same intent as the prior template). The reset link MUST open a **client** guest route with a password form (hash path on `CLIENT_APP_URL`), not an auth-API HTML form. Submitting a new password MUST call a JSON reset endpoint. Success and failure outcomes on the SPA MUST be in Russian (including expired or invalid token). After a successful reset the client MUST navigate to Login so the user can sign in with the new password. Legacy `api.*/auth/reset-password` HTML MUST NOT be the product path.

#### Scenario [SC-RESET-06]: Reset mail is Russian

- **GIVEN** the system sends a password-reset email
- **WHEN** the message is composed
- **THEN** the subject and HTML body are in Russian
- **AND** the body includes a working reset link to the client SPA reset route

#### Scenario [SC-RESET-02]: Valid link sets new password on SPA

- **GIVEN** the user received a valid password-reset link for their account
- **WHEN** the user opens the reset flow
- **THEN** the client SPA reset form is shown in Russian
- **WHEN** the user submits a new password through the SPA form
- **THEN** the account password is updated via the JSON reset API
- **AND** the SPA shows a successful reset outcome in Russian

#### Scenario [SC-RESET-08]: Reset success navigates to Login; link uses client origin

- **GIVEN** the user has just successfully reset their password via the SPA + JSON flow
- **WHEN** the success flow completes
- **THEN** the client navigates to the Login screen
- **AND** is not left on the reset form as the final screen
- **GIVEN** the system composes a password-reset email
- **WHEN** the reset link is built
- **THEN** the link’s origin is the configured client application URL
- **AND** the path is the client hash reset route with the token

### Requirement: Reset password via email link

Opening a valid password-reset link MUST allow the user to set a new password through the client SPA reset flow. After a successful reset, signing in with the email and the new password MUST succeed; the previous password MUST no longer authenticate.

#### Scenario [SC-RESET-03]: Login works with new password only

- **GIVEN** the user has successfully reset their password to P_new (previous password was P_old)
- **WHEN** the user signs in with email and P_new
- **THEN** authentication succeeds
- **AND** signing in with email and P_old fails
