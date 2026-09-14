# ui/theme Specification

## Purpose

Управление светлой и тёмной темой UI Happy Tourist: следование теме устройства до явного выбора, локальное сохранение для гостя и серверный preference для зарегистрированных пользователей с применением при логине и при reload/restore сессии из актуального профиля БД (в том числе на другом уже залогиненном устройстве после его обновления страницы). Live-push без reload не требуется. Restore preference HTTP MUST NOT storm (один логический restore → конечное малое число запросов).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-THEME-01 | implemented (client Dark boot/`auto`; lint+typecheck) |
| SC-THEME-02 | implemented (App.vue header toggle; lint+typecheck) |
| SC-THEME-03 | implemented (guest `localStorage` `ht-theme`; lint+typecheck) |
| SC-THEME-04 | server mocha (HTTP save theme + reject unauthenticated / anonymous) |
| SC-THEME-05 | server mocha (registered user theme persisted; login userdata includes theme) |
| SC-THEME-06 | implemented (theme from login/userdata; lint+typecheck) — restore after reload уточняется SC-THEME-08/09 |
| SC-THEME-07 | covered-by-reuse (board visuals unchanged — no board CSS change in scope) |
| SC-THEME-08 | implemented (server mocha GET profile after save with same JWT; client GET restore) |
| SC-THEME-09 | implemented (server mocha + client: other device session picks up theme after its reload) |
| SC-THEME-10 | implemented (stable App.vue watch + no auth.user replace after GET; lint+typecheck) |

## Requirements

### Requirement: Default theme follows the device until an explicit choice

The system SHALL present the application chrome in light or dark appearance according to the device (operating system) color scheme when the user has not yet stored an explicit theme preference. After the user explicitly selects light or dark, the system MUST use that selection instead of the device default until changed again.

#### Scenario [SC-THEME-01]: Unset preference follows device

- **GIVEN** the user has no stored theme preference (guest without local preference, or registered user with unset theme in profile)
- **WHEN** the client presents the application UI
- **THEN** the chrome appearance follows the device light/dark color scheme

#### Scenario [SC-THEME-02]: Explicit toggle switches between light and dark

- **GIVEN** the user can access the theme control in the shared application header on Login, Lobby, and Game screens
- **WHEN** the user activates the theme control
- **THEN** the chrome switches between light and dark
- **AND** the choice is treated as an explicit preference (no longer solely device-driven for that session’s stored preference)

### Requirement: Guest preference is device-local only

For an anonymous (guest) session the system SHALL persist the explicit theme preference only on the current browser/device. The system MUST NOT rely on the server profile to restore guest theme across devices.

#### Scenario [SC-THEME-03]: Guest theme survives reload on the same device

- **GIVEN** the user is authenticated as anonymous (guest) and has explicitly chosen light or dark
- **WHEN** the page is reloaded on the same browser
- **THEN** the previously chosen light or dark chrome is applied
- **AND** that preference is not required to exist in the server user profile for the guest

### Requirement: Registered user theme is stored in the user profile

The system SHALL store an explicit `light` or `dark` theme preference on the registered user profile (email/password or Google). Saving MUST require a valid JWT for a non-anonymous user. Unauthenticated or anonymous callers MUST be rejected. Until the user saves an explicit choice, the profile theme MAY be unset (device default on the client).

#### Scenario [SC-THEME-04]: Save theme requires registered JWT

- **GIVEN** a caller without a valid registered-user JWT, or with an anonymous session
- **WHEN** the caller attempts to save a theme preference via the preferences HTTP API
- **THEN** the server rejects the request (unauthorized / forbidden)
- **AND** no registered user’s theme is updated

#### Scenario [SC-THEME-05]: Registered user theme is persisted

- **GIVEN** a registered user with a valid JWT
- **WHEN** the client saves theme `light` or `dark` via the preferences HTTP API
- **THEN** the server stores that value on the user profile
- **AND** a subsequent successful login for that user includes the stored theme in userdata

### Requirement: Registered theme is applied on login

When a registered user completes login (email/password or Google), the client SHALL apply that light or dark preference to the chrome from login userdata (or an equivalent authenticated profile read). A second device obtains the same preference when that user logs in there. Live push to already-open sessions without reload is NOT required.

#### Scenario [SC-THEME-06]: Theme from userdata applied after login

- **GIVEN** a registered user whose profile has theme `dark` (or `light`)
- **WHEN** the user successfully logs in (or the client receives userdata containing that theme on session restore after login)
- **THEN** the client applies the corresponding dark or light chrome
- **AND** a second device obtains the same preference when that user logs in there

### Requirement: Registered theme on session restore comes from the user profile

When a registered (non-anonymous) user restores or reloads an authenticated session, the system SHALL apply the current `light` or `dark` theme stored on that user’s profile in the database. The system MUST NOT rely solely on theme claims frozen in the JWT from an earlier login when those claims differ from the profile. If the profile theme is unset, the client MAY follow the device color scheme. The system is NOT required to push theme updates to other open sessions without a page reload or equivalent session restore on that client.

#### Scenario [SC-THEME-08]: Reload applies profile theme after save

- **GIVEN** a registered user with a valid JWT whose token claims may still reflect an older or unset theme
- **AND** the user has saved theme `light` or `dark` on the profile via the preferences HTTP API
- **WHEN** the client restores or reloads the authenticated session
- **THEN** the chrome applies the theme currently stored on the user profile
- **AND** a stale theme value in JWT claims alone does not override the profile value

#### Scenario [SC-THEME-09]: Other device picks up theme after its reload

- **GIVEN** a registered user has an authenticated session on device B
- **AND** the same user has saved a different theme on the profile from device A
- **WHEN** device B reloads or restores the session (without requiring a new login)
- **THEN** device B applies the theme currently stored on the user profile
- **AND** the system is not required to update device B before that reload or restore

### Requirement: Theme restore does not storm preference HTTP

When the client restores theme for a registered session (after `auth.ready` / identity is established), the system SHALL issue a finite small number of preference read requests for that logical restore — typically one `GET` of the profile theme. Completing the restore (including applying chrome and any in-memory userdata or local preference updates) MUST NOT by itself trigger another preference read for the same ready identity. Ignoring stale HTTP responses alone is NOT sufficient if new reads keep being started. Guest restore MUST NOT call the preference read API.

#### Scenario [SC-THEME-10]: Registered restore issues a single preference read

- **GIVEN** a registered (non-anonymous) user whose auth session becomes ready with a stable identity
- **WHEN** the client performs theme restore from the user profile
- **THEN** the preference read HTTP API is invoked a finite small number of times for that restore (typically once)
- **AND** applying the returned theme (and any in-memory userdata or localStorage update) does not start another preference read for the same ready identity without a new identity change, sign-out, or explicit user theme action

### Requirement: Tourist board appearance is unchanged by theme

Changing the application chrome theme MUST NOT alter the tourist board tile colors (start, task, and center fills). Board holes MUST continue to show the Game screen page background under the active light or dark chrome.

#### Scenario [SC-THEME-07]: Board looks the same in light and dark chrome

- **GIVEN** the user is on the Game screen with a visible tourist board
- **WHEN** the chrome theme is switched between light and dark
- **THEN** start, task, and center tile colors keep their gameplay fills
- **AND** board holes remain visually aligned with the current page background
