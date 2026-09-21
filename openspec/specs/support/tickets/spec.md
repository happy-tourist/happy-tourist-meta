# support/tickets Specification

## Purpose

Обращения в поддержку по HTTP: создание, список и тред, статусы, лимиты, автозакрытие, письма автору при создании и смене статуса (не гостю), staff-очередь с фильтрами, UX формы/треда/guest-warn. Ссылка из лобби. Без Colyseus realtime.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SUP-01 | server: support.test.ts |
| SC-SUP-02 | server: support.test.ts |
| SC-SUP-03 | server: support.test.ts |
| SC-SUP-04 | client: guest warning (mail + session) |
| SC-SUP-05 | server: support.test.ts |
| SC-SUP-06 | server: support.test.ts |
| SC-SUP-07 | server: support.test.ts |
| SC-SUP-08 | server: support.test.ts |
| SC-SUP-09 | server: support.test.ts |
| SC-SUP-10 | server: support.test.ts |
| SC-SUP-11 | server: support.test.ts |
| SC-SUP-12 | server: support.test.ts |
| SC-SUP-13 | server: support.test.ts |
| SC-SUP-14 | server: support.test.ts |
| SC-SUP-15 | server: support.test.ts |
| SC-SUP-16 | server: support.test.ts |
| SC-SUP-17 | server: support.test.ts |
| SC-SUP-18 | client: lobby support link |
| SC-SUP-19 | server: support.test.ts |
| SC-SUP-20 | server: support.test.ts |
| SC-SUP-21 | server: support.test.ts |
| SC-SUP-22 | server: support.test.ts |
| SC-SUP-23 | client: SupportStaffPage filters |
| SC-SUP-24 | client: SupportPage / SupportTicketPage clear + no red empty (lazy-rules / nextTick reset) |
| SC-SUP-25 | client: SupportTicketPage thread message spacing |
| SC-SUP-26 | client: SupportTicketPage gap between last message and reply composer |

## Requirements

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

### Requirement: Guest is warned about email and session loss

When an anonymous user opens the support create flow, the client MUST show a clear Russian warning that: (1) the ticket will be reviewed but email status notifications are unavailable for guests; (2) if they lose the current session (new guest sign-in), they may not see replies or ticket history.

#### Scenario [SC-SUP-04]: Guest sees no-notify and session warning

- **GIVEN** the user is authenticated as anonymous
- **WHEN** the user opens the support create UI
- **THEN** the UI shows a Russian warning covering unavailable email notifications and the risk of losing access to the ticket if the guest session is lost

### Requirement: Author has a ticket list and a public thread page

The author MUST see a list of their own tickets and MUST open a ticket detail that shows the public message thread (author and staff messages only; no private staff notes). On the ticket detail UI, consecutive messages MUST be visually separated so the thread is readable (not edge-to-edge glued text blocks).

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

#### Scenario [SC-SUP-25]: Thread messages are visually spaced

- **GIVEN** a ticket detail with two or more messages
- **WHEN** the author or staff views the thread
- **THEN** messages are shown with visible separation between entries

#### Scenario [SC-SUP-26]: Visible gap between thread and reply composer

- **GIVEN** an open ticket with at least one message and a visible reply form
- **WHEN** the author or staff views the ticket detail
- **THEN** there is a visible vertical gap between the last message block and the reply input
- **AND** the message block border and the reply field border are not flush against each other

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

### Requirement: Emails to the author on create and on status change

When a ticket is successfully created, if the author is a registered (non-anonymous) user with an email address, the system MUST send a Russian acknowledgment email stating the request was received (with a link to the ticket). When a ticket status changes, if the author has an email, the system MUST send a Russian email describing the new status (including manual close and auto-close). Anonymous authors MUST NOT receive email. Staff MUST NOT receive ticket emails in this change. Create-ack copy MUST be distinct from auto-close copy.

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

#### Scenario [SC-SUP-21]: Registered author receives create acknowledgment email

- **GIVEN** a registered user with an email
- **WHEN** the user successfully creates a support ticket
- **THEN** an email is sent acknowledging receipt of the request in Russian
- **AND** the email includes a link to the ticket detail

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

### Requirement: Staff can list tickets with topic and status filters

Moderator and admin MUST be able to list tickets for moderation. The staff list MUST support filtering by topic (all topics or one of the allowed topics) and by status group: `open` (any status except `closed`), `closed`, or `all`. Default when unspecified: topic = all, status group = `open`. Listing with status group `all` MUST include open and closed tickets (history).

#### Scenario [SC-SUP-17]: Staff lists tickets across statuses when requested

- **GIVEN** tickets from multiple authors in various statuses
- **AND** the actor is moderator or admin
- **WHEN** the staff requests the staff ticket list with status group `all`
- **THEN** the response includes those tickets across statuses

#### Scenario [SC-SUP-22]: Staff list filters by topic and open status by default

- **GIVEN** open and closed tickets with different topics
- **AND** the actor is moderator or admin
- **WHEN** the staff requests the staff ticket list without filters (defaults)
- **THEN** the response includes only non-closed tickets
- **AND** tickets of every topic may appear

#### Scenario [SC-SUP-23]: Staff UI exposes topic and status filters

- **GIVEN** a staff user on the staff ticket queue UI
- **WHEN** the queue is shown
- **THEN** controls allow choosing topic (default all) and status group open|closed|all (default open)
- **AND** changing filters refreshes the listed tickets accordingly

### Requirement: Lobby header exposes Support for authenticated users

While the user is on the lobby experience with a valid session, the lobby header MUST provide a Support entry that opens the support section (create/list). Guests with JWT MUST also see it.

#### Scenario [SC-SUP-18]: Lobby header has Support link

- **GIVEN** an authenticated user on the lobby
- **WHEN** the lobby header is shown
- **THEN** a Support navigation control is available
- **AND** activating it opens the support section

### Requirement: Support forms clear validation after successful submit

After a successful create or reply submit, the client MUST clear the message field and MUST NOT leave the empty field in an error/validation-failed visual state (empty after success is expected). Clearing the model MUST NOT by itself re-trigger a required-field error for that empty state after success.

#### Scenario [SC-SUP-24]: Create or reply does not show empty-field error after success

- **GIVEN** an authenticated user on the support create or ticket reply form
- **WHEN** the user successfully submits a non-empty message
- **THEN** the message field is cleared
- **AND** the form does not show a validation error for the empty field solely because of that clear
- **AND** the empty field is not left in a red/error visual state without a real validation failure
