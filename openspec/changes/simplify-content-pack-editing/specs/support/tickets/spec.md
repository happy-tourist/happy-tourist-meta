# support/tickets — delta (simplify-content-pack-editing)

Дополняет `openspec/specs/support/tickets/spec.md`: тема запроса на изменение набора карточек + выбор пака из каталога.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SUP-27 | pending |
| SC-SUP-28 | pending |
| SC-SUP-29 | pending |
| SC-SUP-01 | pending |
| SC-SUP-02 | pending |
| SC-SUP-03 | pending |
| SC-SUP-19 | pending |
| SC-SUP-20 | pending |

## ADDED Requirements

### Requirement: Change-pack topic includes catalog pack selection and link

Authenticated users MUST be able to create a support ticket with topic **change_pack** (изменить набор карточек). When that topic is selected, the client MUST show a pack selector listing packs from the **public catalog** (title and description). On submit the ticket MUST include the selected pack identity and a link/URL to that pack’s live page so staff can open the pack and use Edit themselves. Create without a selected pack for this topic MUST be rejected. Other existing topics keep topic+body-only create.

#### Scenario [SC-SUP-27]: Change-pack topic requires pack from catalog

- **GIVEN** an authenticated user and at least one published catalog pack P
- **WHEN** the user creates a ticket with the change-pack topic, selects P, and a non-empty body
- **THEN** the ticket is created
- **AND** the ticket payload/thread context includes P’s identity and a link to P’s live pack page

#### Scenario [SC-SUP-28]: Change-pack without pack selection is rejected

- **GIVEN** an authenticated user
- **WHEN** the user submits the change-pack topic without selecting a catalog pack
- **THEN** the system rejects the create request

#### Scenario [SC-SUP-29]: Pack selector shows catalog title and description

- **GIVEN** the user opened support create and chose the change-pack topic
- **WHEN** the pack selector is shown
- **THEN** options are taken from the public catalog
- **AND** each option presents the pack title and description

## MODIFIED Requirements

### Requirement: Authenticated user creates a support ticket with topic and body

Any user with a valid JWT (registered or anonymous) MUST be able to create a support ticket by selecting a topic and providing a non-empty message body. Unauthenticated callers MUST be rejected. Topic MUST be one of: `problem`, `suggestion`, `feedback`, `question`, `other`, `change_pack`. On success the ticket MUST start in status `under_review` and MUST have a dedicated detail resource the author can open. When the topic is `change_pack`, create MUST also require a catalog `packId` and pack link as specified in the change-pack topic requirement.

#### Scenario [SC-SUP-01]: Create ticket with topic and body

- **GIVEN** an authenticated user (registered or anonymous)
- **WHEN** the user submits a support ticket with a valid non-change-pack topic and a non-empty body
- **THEN** the system creates the ticket in status `under_review`
- **AND** the author can retrieve that ticket by its id

#### Scenario [SC-SUP-02]: Missing topic or empty body is rejected

- **GIVEN** an authenticated user
- **WHEN** the user submits a ticket without a topic or with an empty/whitespace-only body
- **THEN** the system rejects the create request
- **AND** no ticket is created

#### Scenario [SC-SUP-03]: Anonymous guest may create a ticket

- **GIVEN** an authenticated anonymous user
- **WHEN** the user creates a valid support ticket
- **THEN** the ticket is created and listed for that anonymous identity

#### Scenario [SC-SUP-19]: Only allowed topics are accepted

- **GIVEN** an authenticated user
- **WHEN** the user submits a topic outside `problem` | `suggestion` | `feedback` | `question` | `other` | `change_pack`
- **THEN** the system rejects the create request

#### Scenario [SC-SUP-20]: Unauthenticated create is rejected

- **GIVEN** no auth session
- **WHEN** a create-ticket request is made
- **THEN** the system rejects the request
