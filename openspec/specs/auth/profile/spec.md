# auth/profile Specification

## Purpose

Личный кабинет зарегистрированного (не anonymous) пользователя: просмотр и смена отображаемого имени, смена пароля для email/password-аккаунтов, существующий soft email-verify контур Wave 1, выход из сессии. Модерация имён — вне этой capability.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PROFILE-01 | covered (server mocha + client AccountPage) |
| SC-PROFILE-02 | covered (server mocha + client logout after change) |
| SC-PROFILE-03 | covered (client UX — hide change-password for Google) |
| SC-PROFILE-04 | covered (server mocha — wrong current rejected) |
| SC-PROFILE-05 | covered (server mocha + client empty name reject) |
| SC-PROFILE-06 | covered-by-reuse (soft-verify; no hard-gate added) |

## Requirements

### Requirement: Update display name from cabinet

A registered non-anonymous user MUST be able to update their display name from the personal cabinet. The new name MUST have trimmed length of at least **1** character. On success the session and UI MUST reflect the new display name. Content moderation / denylist of names is out of scope for this capability.

#### Scenario [SC-PROFILE-01]: Display name updates

- **GIVEN** a registered non-anonymous user is on the personal cabinet
- **WHEN** the user submits a new display name N with trimmed length ≥ 1
- **THEN** the account display name becomes N
- **AND** subsequent lobby/game chrome that shows the display name uses N

#### Scenario [SC-PROFILE-05]: Empty display name rejected

- **GIVEN** a registered non-anonymous user is on the personal cabinet
- **WHEN** the user submits a blank or whitespace-only display name
- **THEN** the display name is not changed
- **AND** the user sees clear feedback

### Requirement: Change password for email/password accounts

A registered email/password user (account that has a password credential) MUST be able to change password from the cabinet by providing the current password and a new password that satisfies the shared password policy (`auth/login`). On success the system MUST invalidate prior sessions (token version bump) and the client MUST sign the user out and require sign-in with the new password. Google OAuth users without a password credential MUST NOT see a change-password control. Anonymous guests MUST NOT use this cabinet password flow.

#### Scenario [SC-PROFILE-02]: Password change succeeds then re-login required

- **GIVEN** an email/password user is on the personal cabinet
- **WHEN** the user submits the correct current password and a new password that meets the shared policy
- **THEN** the password is updated
- **AND** the client ends the current session and navigates to Login
- **AND** sign-in with the new password succeeds afterward

#### Scenario [SC-PROFILE-03]: Google user has no change-password control

- **GIVEN** a registered user authenticated via Google without a local password credential is on the personal cabinet
- **WHEN** the cabinet is shown
- **THEN** there is no change-password form or control

#### Scenario [SC-PROFILE-04]: Wrong current password rejected

- **GIVEN** an email/password user is on the personal cabinet
- **WHEN** the user submits an incorrect current password with any new password
- **THEN** the password is not changed
- **AND** the session remains signed in
- **AND** the user sees clear feedback

### Requirement: Cabinet keeps soft email-verify and logout

The personal cabinet MUST continue to expose the Wave 1 soft email confirmation / change-email controls for eligible users. Soft-verify MUST NOT hard-gate lobby or game actions in this capability. The cabinet or its surrounding authenticated chrome MUST allow the user to sign out.

#### Scenario [SC-PROFILE-06]: Unverified user keeps play access

- **GIVEN** a registered email/password user with `emailVerified` false
- **WHEN** the user uses lobby create/join and game actions that are otherwise allowed
- **THEN** those actions are not blocked solely because the email is unverified
