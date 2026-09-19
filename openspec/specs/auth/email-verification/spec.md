# auth/email-verification Specification

## Purpose

Soft-подтверждение email по явному действию в личном кабинете: кнопка рядом с почтой, смена адреса, напоминание-модалка без автоотправки при регистрации и без блокировки игры. Ссылка из письма открывает **клиентскую SPA**-страницу на `CLIENT_APP_URL` (не HTML на API); подтверждение идёт через **JSON**; после успеха — лобби. Письма и SPA-тексты — на русском; после отправки — напоминание про «Спам».

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-EMAIL-01 | covered (server: register does **not** send confirm mail) |
| SC-EMAIL-02 | covered (client SPA + JSON confirm; not API HTML) |
| SC-EMAIL-03 | covered-by-reuse (existing JWT lobby/game gate unchanged) |
| SC-EMAIL-04 | covered (server: Google path sets verified) |
| SC-EMAIL-05 | covered (server: existing rows default/migration verified) |
| SC-EMAIL-06 | covered (client+server: cabinet button sends confirm mail + success dialog incl. spam) |
| SC-EMAIL-07 | covered (server: send-confirm cooldown 60s) |
| SC-EMAIL-08 | covered (client: session modal → cabinet) |
| SC-EMAIL-09 | covered-by-reuse (anonymous has no email verify UX) |
| SC-EMAIL-10 | covered (client+server: change email resets verified) |
| SC-EMAIL-11 | covered (SPA success → client `#/lobby`) |
| SC-EMAIL-12 | covered (expired token → SPA RU error; no verify) |
| SC-EMAIL-13 | covered (server: confirm email subject+body in Russian; link → client SPA hash) |
| SC-EMAIL-14 | covered (client: post-send UI mentions spam folder) |
| SC-EMAIL-15 | covered (confirm link host = client origin hash route) |

## Requirements

### Requirement: Registration does not auto-send confirmation email

Email/password registration MUST create the account and issue a normal authenticated session without sending a confirmation email. Confirmation mail MUST be sent only when the user explicitly requests it from the personal account area.

#### Scenario [SC-EMAIL-01]: Register does not send confirmation email

- **GIVEN** an email address is not yet registered
- **WHEN** the user completes email/password registration
- **THEN** the system creates the account and issues an auth session
- **AND** no confirmation email is sent as part of that registration

### Requirement: Confirming the email marks the account verified via client SPA

Following the confirmation link from the email MUST open a **client application** guest route (hash path on the configured client origin), not an auth-API HTML page. The client MUST immediately submit the token to a JSON confirmation endpoint (no extra user click to “confirm”). A valid token MUST mark the account email-verified. After success the client MUST navigate to the lobby entry (`#/lobby`). Failure outcomes (expired, invalid token) MUST be shown in Russian on the SPA page. The system MUST NOT rely on legacy `api.*/auth/confirm-email` HTML links.

#### Scenario [SC-EMAIL-02]: Link confirms email on client SPA

- **GIVEN** a registered email/password account that is not yet email-verified
- **AND** the user previously requested a confirmation email from the account area
- **WHEN** the user opens a valid confirmation link from that email
- **THEN** the client SPA confirmation page loads with the token from the link
- **AND** the client calls the JSON confirmation API without requiring another confirm button
- **AND** the account is marked email-verified
- **AND** the user is shown a successful confirmation outcome in Russian on the SPA

#### Scenario [SC-EMAIL-11]: Success navigates to client lobby

- **GIVEN** the user has just successfully confirmed their email via the SPA + JSON flow
- **WHEN** the confirmation success flow completes
- **THEN** the client navigates to the lobby entry (hash lobby route)
- **AND** is not directed to the Login screen as the primary success destination

#### Scenario [SC-EMAIL-15]: Confirmation link uses client origin

- **GIVEN** the system composes a confirmation email
- **WHEN** the confirmation link is built
- **THEN** the link’s origin is the configured client application URL (`CLIENT_APP_URL`)
- **AND** the path is the client hash confirm route with the token
- **AND** the link does not use the API host as the user-facing confirmation page

#### Scenario [SC-EMAIL-12]: Confirmation link expires after 30 minutes

- **GIVEN** the user received a confirmation email whose link token is older than 30 minutes
- **WHEN** the user opens that confirmation link
- **THEN** the account is not marked email-verified via that link
- **AND** the client SPA shows a failed/expired confirmation outcome in Russian

### Requirement: Soft verification does not block play

Until a later product change, an unverified email/password user with a valid session MUST still be able to use lobby and game rooms the same way as a verified registered user. Verification status MUST NOT gate room join.

#### Scenario [SC-EMAIL-03]: Unverified user can still play

- **GIVEN** the user registered with email/password and has a valid session but email is not verified
- **WHEN** the user opens the lobby or joins/creates a game room
- **THEN** access succeeds under the same JWT rules as for other authenticated players
- **AND** the system does not reject the action solely because email is unverified

### Requirement: Google and pre-existing accounts are verified

Accounts authenticated or created via Google MUST be treated as email-verified. Existing email/password accounts already present before this capability was deployed MUST be treated as email-verified after deployment (migration or equivalent default), so they are not prompted to confirm.

#### Scenario [SC-EMAIL-04]: Google accounts are verified

- **GIVEN** the user signs in or registers through Google
- **WHEN** the auth session is established
- **THEN** the account is marked email-verified
- **AND** the client does not treat the user as needing email confirmation

#### Scenario [SC-EMAIL-05]: Legacy email accounts are verified

- **GIVEN** an email/password account existed before this capability was deployed
- **WHEN** the deployment migration (or equivalent default) has been applied
- **THEN** that account is marked email-verified
- **AND** the user is not shown the unverified-email reminder solely due to legacy data

### Requirement: Account area sends confirmation on button next to email

A registered (non-anonymous) user MUST see their current email in a personal account area. If the email is not verified, the UI MUST show a confirmation control next to the email. Activating that control MUST send a confirmation email through an authenticated HTTP endpoint, subject to a cooldown of 60 seconds between successful sends for the same user. After a successful send the client MUST show a dialog (or equivalent modal) in Russian stating that the email was sent and that the user should check their mailbox **and the spam folder**. The confirmation link in the email MUST expire after 30 minutes.

#### Scenario [SC-EMAIL-06]: Cabinet button sends confirmation email and shows dialog

- **GIVEN** a registered non-anonymous user with an unverified email and a valid session
- **WHEN** the user opens the personal account area
- **THEN** the UI shows the current email and that it is not verified
- **AND** a confirmation control is shown next to the email
- **WHEN** the user activates that control and the send succeeds
- **THEN** a confirmation email is sent to the current address with a working confirmation link
- **AND** the client shows a dialog telling the user the email was sent and to check their mail and spam folder

#### Scenario [SC-EMAIL-07]: Send confirmation cooldown

- **GIVEN** the user successfully requested a confirmation email less than 60 seconds ago
- **WHEN** the user requests another confirmation send before the cooldown ends
- **THEN** the system rejects or ignores the new send
- **AND** does not send another confirmation email for that request

#### Scenario [SC-EMAIL-14]: Post-send UI mentions spam folder

- **GIVEN** a registered non-anonymous user successfully requested a confirmation email from the account area
- **WHEN** the client shows the send-success dialog
- **THEN** the dialog text is in Russian
- **AND** the text tells the user to check the spam folder as well as the inbox

### Requirement: Confirmation email content is in Russian

The confirmation email subject and body MUST be in Russian. The body MUST carry the same intent as the previous English template (welcome / ask to confirm / button to confirm), translated — not a new product flow. The button/link MUST target the client SPA confirm route.

#### Scenario [SC-EMAIL-13]: Confirm mail is Russian

- **GIVEN** the system sends a confirmation email from the account-area send endpoint
- **WHEN** the message is composed
- **THEN** the subject and HTML body are in Russian
- **AND** the body includes a working confirmation link to the client SPA confirm route

### Requirement: Change email from account area

A registered (non-anonymous) user MUST be able to change their account email from the personal account area. After a successful change the new address MUST be marked not verified, and confirmation MUST again require the explicit cabinet button (no auto-send on change unless the user presses the button).

#### Scenario [SC-EMAIL-10]: Change email resets verification

- **GIVEN** a registered non-anonymous user with a valid session
- **WHEN** the user successfully changes their account email to a new unused address
- **THEN** the account email is updated to the new address
- **AND** the account is marked not email-verified
- **AND** no confirmation email is sent solely because of the change
- **AND** the confirmation control next to the email is available again

### Requirement: Once-per-session reminder points to account area

For a registered non-anonymous session whose email is not verified, the client MUST show a reminder modal once per browser session (dismissible for the rest of that session). The modal MUST tell the user they can confirm their email from the personal account area (not that a mail was already sent). Anonymous guests MUST NOT see this reminder.

#### Scenario [SC-EMAIL-08]: Modal once per session points to cabinet

- **GIVEN** a registered user with unverified email opens a protected screen in a new browser session
- **WHEN** the client finishes auth readiness
- **THEN** a reminder modal is shown once stating that email can be confirmed from the personal account area
- **AND** after dismiss, reopening protected screens in the same session does not show the modal again

#### Scenario [SC-EMAIL-09]: Anonymous has no reminder

- **GIVEN** the user is signed in anonymously
- **WHEN** the user opens protected screens
- **THEN** the email confirmation reminder modal is not shown
