## ADDED Requirements

### Requirement: Soft-unpublish pack cancels open moderation requests

When staff soft-unpublishes a pack that is currently in the public catalog, the system MUST set every **open** moderation request for that pack (`pending` or `needs_revision`, any request type including `pack`, `task_set`, and legacy `answers`/`tasks`) to **cancelled**. After that cascade, those requests MUST NOT appear in the author’s «На модерации» list or in the staff moderation queue. Soft-unpublish of an already soft-unpublished pack MUST NOT re-cancel or re-notify. **Republish** MUST NOT restore cancelled requests; authors MUST submit again if they still want review. Soft-unpublish of an individual **task set** MUST NOT by itself cancel pack-level open requests. The staff confirm dialog for pack unpublish MUST warn that open moderation requests will be cancelled. The system MUST NOT append a moderation thread message solely for this cascade cancel.

#### Scenario [SC-PACK-137]: Unpublish cancels open add-task-set request

- **GIVEN** in-catalog pack P with an open `task_set` moderation request authored by A
- **WHEN** staff soft-unpublishes P
- **THEN** that request status is `cancelled`
- **AND** the request MUST NOT appear on A’s «На модерации» list
- **AND** the request MUST NOT appear in the staff moderation queue

#### Scenario [SC-PACK-138]: Republish does not restore cancelled request

- **GIVEN** pack P was soft-unpublished while A had an open request that was cascade-cancelled
- **WHEN** staff republishes P
- **THEN** A’s cancelled request remains cancelled
- **AND** A MUST NOT see that request on «На модерации» solely because of republish

#### Scenario [SC-PACK-139]: Confirm warns about cancelling open requests

- **GIVEN** staff S about to soft-unpublish in-catalog pack P
- **WHEN** the client shows the unpublish confirmation
- **THEN** the confirm copy MUST state that open moderation requests for P will be cancelled
- **AND** cancelling the dialog MUST leave P in catalog and leave open requests unchanged

### Requirement: Email on pack unpublish cascade cancel

When staff soft-unpublish of a pack cascade-cancels one or more open moderation requests, the system MUST send Russian email (same mail channel as other content moderation mail) to each distinct **change author** of those cancelled requests who is a non-anonymous user with an email. Each such author MUST receive **exactly one** email for that unpublish event, even if several of their requests on that pack were cancelled. The email MUST state that the pack was soft-unpublished and that their pending moderation submission(s) were cancelled. The email MUST **NOT** include any URL or deep-link. Anonymous authors MUST NOT receive email. Manual author/staff Cancel of a request (outside this cascade) MUST NOT gain a new email requirement from this change. Soft-unpublish when the pack was already soft-unpublished MUST NOT send cascade-cancel email again.

#### Scenario [SC-PACK-140]: Cascade cancel notifies each author once without links

- **GIVEN** in-catalog pack P with open moderation request(s) by registered author A with email
- **WHEN** staff soft-unpublishes P (cascade-cancelling A’s open request(s) on P)
- **THEN** A receives exactly one Russian email about the unpublish and cancelled submission(s)
- **AND** that email contains no URL / deep-link
- **AND** anonymous change authors receive no email

#### Scenario [SC-PACK-141]: Multiple requests for one author → one email

- **GIVEN** in-catalog pack P with two open moderation requests both authored by A
- **WHEN** staff soft-unpublishes P
- **THEN** both requests are cancelled
- **AND** A receives exactly one cascade-cancel email for that event
