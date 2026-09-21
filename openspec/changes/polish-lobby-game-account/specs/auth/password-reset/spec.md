## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-RESET-09 | pending |

## ADDED Requirements

### Requirement: Password policy on SPA reset

Setting a new password via the SPA password-reset flow MUST enforce the same product password policy as email registration (`auth/login`): minimum length **8**; Latin lowercase; Latin uppercase; digit; symbol (any non Latin-letter non-digit character). The SPA MUST show the same advisory colored strength indication as register. Submit MUST be gated by the policy, not by a minimum strength score alone. After a successful reset, navigation to Login and authentication with the new password remain as in existing reset scenarios.

#### Scenario [SC-RESET-09]: Reset rejects weak new password

- **GIVEN** the user has a valid reset token and is on the SPA reset form
- **WHEN** the user submits a new password that fails the shared password policy
- **THEN** the password is not updated
- **AND** the user sees clear Russian feedback that the password does not meet the policy
