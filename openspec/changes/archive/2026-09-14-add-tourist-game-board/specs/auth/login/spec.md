## MODIFIED Requirements

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
