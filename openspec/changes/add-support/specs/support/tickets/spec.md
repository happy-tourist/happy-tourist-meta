## Purpose

Обращения в поддержку по HTTP: создание (тема + текст), список и тред автора, статусы, ответы staff и автора, лимиты, автозакрытие без ответа, письма автору при смене статуса (не гостю). Ссылка из лобби. Без Colyseus realtime.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SUP-01 | pending (server: create ticket topic+body) |
| SC-SUP-02 | pending (server: reject missing topic or empty body) |
| SC-SUP-03 | pending (server: anonymous create allowed) |
| SC-SUP-04 | pending (client: guest warning no email notify) |
| SC-SUP-05 | pending (server: author list own tickets) |
| SC-SUP-06 | pending (server: ticket detail + public thread) |
| SC-SUP-07 | pending (server: author message when open) |
| SC-SUP-08 | pending (server: status under_review → in_progress) |
| SC-SUP-09 | pending (server: staff sets awaiting_response) |
| SC-SUP-10 | pending (server: mail on status change if email) |
| SC-SUP-11 | pending (server: no mail for anonymous) |
| SC-SUP-12 | pending (server: auto-close awaiting after 3d) |
| SC-SUP-13 | pending (server: auto-close mail distinct copy) |
| SC-SUP-14 | pending (server: author may close) |
| SC-SUP-15 | pending (server: closed thread rejects new messages) |
| SC-SUP-16 | pending (server: rate limits create/open/msg) |
| SC-SUP-17 | pending (server: staff list all tickets) |
| SC-SUP-18 | pending (client: lobby header support link) |
| SC-SUP-19 | pending (server: topics enum) |
| SC-SUP-20 | pending (server: unauthenticated rejected) |

## ADDED Requirements

### Requirement: Authenticated user creates a support ticket with topic and body

Any user with a valid JWT (registered or anonymous) MUST be able to create a support ticket by selecting a topic and providing a non-empty message body. Unauthenticated callers MUST be rejected. Topic MUST be one of: `problem`, `suggestion`, `feedback`, `question`, `other`. On success the ticket MUST start in status `under_review` and MUST have a dedicated detail resource the author can open.

#### Scenario [SC-SUP-01]: Create ticket with topic and body

- **GIVEN** an authenticated user (registered or anonymous)
- **WHEN** the user submits a support ticket with a valid topic and a non-empty body
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
- **WHEN** the user submits a topic outside `problem` | `suggestion` | `feedback` | `question` | `other`
- **THEN** the system rejects the create request

#### Scenario [SC-SUP-20]: Unauthenticated create is rejected

- **GIVEN** no auth session
- **WHEN** a create-ticket request is made
- **THEN** the system rejects the request

### Requirement: Guest is warned that status email will not be sent

When an anonymous user opens the support create flow, the client MUST show a clear Russian warning that the ticket will be reviewed but the team cannot notify them of status changes because they are a guest.

#### Scenario [SC-SUP-04]: Guest sees no-notify warning

- **GIVEN** the user is authenticated as anonymous
- **WHEN** the user opens the support create UI
- **THEN** the UI shows a Russian warning that the request will be considered but email status notifications are unavailable for guests

### Requirement: Author has a ticket list and a public thread page

The author MUST see a list of their own tickets and MUST open a ticket detail that shows the public message thread (author and staff messages only; no private staff notes).

#### Scenario [SC-SUP-05]: Author lists own tickets

- **GIVEN** the user has created one or more tickets
- **WHEN** the user requests their ticket list
- **THEN** the response includes those tickets
- **AND** does not include other users' tickets

#### Scenario [SC-SUP-06]: Ticket detail exposes public thread

- **GIVEN** a ticket with messages from the author and from staff
- **WHEN** the author opens the ticket detail
- **THEN** the public thread shows those messages in order
- **AND** there are no staff-only private notes in the product

### Requirement: Author and staff may post messages while the ticket is open

While the ticket is not `closed`, the author and staff (moderator or admin) MUST be able to append messages to the public thread. When the status is `awaiting_response` and the author posts a message, the status MUST return to `in_progress`.

#### Scenario [SC-SUP-07]: Author replies and awaiting returns to in progress

- **GIVEN** a ticket in status `awaiting_response` owned by the user
- **WHEN** the author posts a non-empty message
- **THEN** the message is appended to the thread
- **AND** the ticket status becomes `in_progress`

### Requirement: Staff advances ticket status through the defined lifecycle

Staff (moderator or admin) MUST be able to move a ticket from `under_review` to `in_progress` (take into work), set `awaiting_response` when asking the user for clarification, and set `closed`. Allowed statuses are exactly: `under_review`, `in_progress`, `awaiting_response`, `closed`.

#### Scenario [SC-SUP-08]: Staff takes ticket into work

- **GIVEN** a ticket in `under_review`
- **AND** the actor is moderator or admin
- **WHEN** the staff takes the ticket into work
- **THEN** the status becomes `in_progress`

#### Scenario [SC-SUP-09]: Staff sets awaiting response

- **GIVEN** a ticket in `in_progress`
- **AND** the actor is moderator or admin
- **WHEN** the staff marks the ticket as awaiting the user's response
- **THEN** the status becomes `awaiting_response`

### Requirement: Status change emails the author when an email exists

When a ticket status changes, if the author is a registered user with an email address, the system MUST send a Russian email describing the new status (including manual close and auto-close). Anonymous authors MUST NOT receive email. Staff MUST NOT receive ticket emails in this change.

#### Scenario [SC-SUP-10]: Registered author receives status email

- **GIVEN** a ticket authored by a registered user with an email
- **WHEN** staff changes the ticket status
- **THEN** an email is sent to that address with Russian subject and body reflecting the new status

#### Scenario [SC-SUP-11]: Anonymous author receives no status email

- **GIVEN** a ticket authored by an anonymous user
- **WHEN** the ticket status changes
- **THEN** no status notification email is sent

#### Scenario [SC-SUP-13]: Auto-close email explains missing reply

- **GIVEN** a registered author with email
- **AND** a ticket auto-closed after no reply in `awaiting_response`
- **WHEN** the auto-close notification is sent
- **THEN** the email states in Russian that the ticket was closed automatically because no reply was received within three days

### Requirement: Awaiting response auto-closes after three days without author reply

If a ticket remains in `awaiting_response` for three full days without a new author message, the system MUST set status to `closed`. Closing MAY be applied lazily on access and/or by a periodic server check.

#### Scenario [SC-SUP-12]: Auto-close after three days awaiting

- **GIVEN** a ticket entered `awaiting_response` more than three days ago
- **AND** the author has not posted a message since that status change
- **WHEN** the system evaluates auto-close (on access or periodic check)
- **THEN** the ticket status becomes `closed`

### Requirement: Author may close an open ticket; closed is read-only

The author MUST be able to close their own open ticket. Staff MUST also be able to close. Once `closed`, neither author nor staff MUST append messages; the author MUST create a new ticket for further contact.

#### Scenario [SC-SUP-14]: Author closes own ticket

- **GIVEN** an open ticket owned by the user
- **WHEN** the author closes the ticket
- **THEN** the status becomes `closed`
- **AND** if the author has an email, a status email is sent

#### Scenario [SC-SUP-15]: Closed ticket rejects new messages

- **GIVEN** a ticket in status `closed`
- **WHEN** the author or staff attempts to post a message
- **THEN** the system rejects the message
- **AND** the thread remains unchanged

### Requirement: Create and message rate limits apply per user

The system MUST enforce: at most **5** ticket creates per user per calendar day; at most **3** non-closed tickets per user at once; at most **30** messages per hour per ticket from a given user. Excess MUST be rejected with a clear error.

#### Scenario [SC-SUP-16]: Rate limits reject excess creates

- **GIVEN** an authenticated user who already created 5 tickets today or already has 3 open tickets
- **WHEN** the user attempts to create another ticket
- **THEN** the system rejects the create request

### Requirement: Staff can list all tickets including history

Moderator and admin MUST be able to list all tickets (open and closed) for moderation history, not only the open queue.

#### Scenario [SC-SUP-17]: Staff lists all tickets

- **GIVEN** tickets from multiple authors in various statuses
- **AND** the actor is moderator or admin
- **WHEN** the staff requests the staff ticket list
- **THEN** the response includes those tickets across statuses

### Requirement: Lobby header exposes Support for authenticated users

While the user is on the lobby experience with a valid session, the lobby header MUST provide a Support entry that opens the support section (create/list). Guests with JWT MUST also see it.

#### Scenario [SC-SUP-18]: Lobby header has Support link

- **GIVEN** an authenticated user on the lobby
- **WHEN** the lobby header is shown
- **THEN** a Support navigation control is available
- **AND** activating it opens the support section
