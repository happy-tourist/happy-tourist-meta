# content/packs — delta: auto-grant default packs

Базовый канон: `openspec/specs/content/packs/spec.md` (коллекция, catalog, soft-unpublish). Этот change только добавляет одноразовую выдачу default-паков.

## Traceability

| ID | Coverage |
|----|----------|
| SC-PACK-142 | done (`test/zz-defaultContentPacks.test.ts`) |
| SC-PACK-143 | done (`test/zz-defaultContentPacks.test.ts`) |
| SC-PACK-144 | done (`test/zz-defaultContentPacks.test.ts`) |
| SC-PACK-145 | done (`test/zz-defaultContentPacks.test.ts`) |
| SC-PACK-146 | done (`test/zz-defaultContentPacks.test.ts`) |
| SC-PACK-147 | done (`test/zz-defaultContentPacks.test.ts`) |

## ADDED Requirements

### Requirement: Default published packs are granted once to every user

The system MUST maintain a configured list of content pack ids that are treated as **default packs**. On server startup the system MUST, for each configured pack id that currently has live content and is **in catalog** (and not blocked), ensure that pack is present in the collection of **every** existing user (including anonymous), idempotently. When a **new** user account is created by any supported auth mode (email/password register, Google OAuth, anonymous guest), the system MUST once grant each eligible default pack into that user’s collection. A pack id that is missing, not live, soft-unpublished (not in catalog), or blocked MUST be skipped without failing startup or user creation. An empty or unset configuration MUST grant nothing. After a user **removes** a default pack from their collection, the system MUST NOT automatically re-add that pack on later login, token refresh, or collection list requests. Manual add from the catalog remains allowed.

#### Scenario [SC-PACK-142]: Startup backfill grants in-catalog defaults to existing users

- **GIVEN** configuration lists pack id P and P is live, in catalog, and not blocked
- **AND GIVEN** existing users U1 (registered) and U2 (anonymous) who do not have P in collection
- **WHEN** the server starts (or runs the default-pack backfill)
- **THEN** P is in U1’s and U2’s collections
- **AND** repeating the backfill does not error or duplicate membership

#### Scenario [SC-PACK-143]: New user receives defaults on account creation

- **GIVEN** configuration lists in-catalog pack P
- **WHEN** a new user is created via email/password register, Google OAuth, or anonymous sign-in
- **THEN** P is in that new user’s collection after creation
- **AND** the user did not need to call the collection-add API

#### Scenario [SC-PACK-144]: Non-eligible pack ids are skipped

- **GIVEN** configuration lists pack id Q that is missing, or has no live revision, or is soft-unpublished (not in catalog), or is blocked
- **WHEN** backfill or new-user grant runs
- **THEN** Q is not added to any user’s collection
- **AND** the operation completes without treating that skip as a fatal error

#### Scenario [SC-PACK-145]: Remove is sticky — no automatic re-grant

- **GIVEN** user U had default pack P granted and then removed P from their collection
- **WHEN** U later signs in again or loads their collection list
- **THEN** P remains absent from U’s collection
- **AND** U MAY still add P manually from the catalog

#### Scenario [SC-PACK-146]: Already collected default is left unchanged

- **GIVEN** user U already has default pack P in collection
- **WHEN** backfill or create-time grant runs for P
- **THEN** U still has exactly one membership for P
- **AND** the operation succeeds

#### Scenario [SC-PACK-147]: Empty configuration grants nothing

- **GIVEN** the default-pack configuration is empty or unset
- **WHEN** the server starts and a new user is created
- **THEN** no packs are auto-added to collections solely by this feature
