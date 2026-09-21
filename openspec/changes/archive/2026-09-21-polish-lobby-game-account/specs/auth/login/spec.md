## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-AUTH-08 | covered (server mocha + client register policy/meter) |
| SC-AUTH-09 | covered (client UX — advisory meter; policy gates submit) |
| SC-AUTH-10 | covered (server mocha + client session displayName) |

## ADDED Requirements

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
