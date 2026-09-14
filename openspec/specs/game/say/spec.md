# game/say Specification

## Purpose

Пресетные реплики у presence на Game: короткие фразы из фиксированного whitelist рассылаются всем в room `tourist` и показываются комикс-облаками у маркера отправителя без свободного текста и без истории чата.

Связанные capability: раскладка presence — `game/presence` (без изменения требований в этом change).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-01 | covered (server mocha) |
| SC-SAY-02 | covered (server mocha) |
| SC-SAY-03 | covered (server mocha) |
| SC-SAY-04 | covered (server mocha) |
| SC-SAY-05 | covered (server mocha) |
| SC-SAY-06 | covered (server mocha) |
| SC-SAY-07 | covered (client UX GamePage) |
| SC-SAY-08 | covered (client UX GamePage) |
| SC-SAY-09 | covered (client UX GamePage) |
| SC-SAY-10 | covered (server mocha) |
| SC-SAY-11 | covered (client UX GamePage) |
| SC-SAY-12 | covered (client UX GamePage) |

## Requirements

### Requirement: Whitelist-only preset say

The system SHALL accept only a fixed set of preset identifiers for say intents in the tourist room. The two product presets in this change are the greeting preset (client display text «Всем привет») and the luck preset (client display text «Удачи»). Free-form or arbitrary text MUST NEVER be accepted as a say payload. Unknown, missing, or non-whitelist identifiers MUST be rejected without broadcasting.

#### Scenario [SC-SAY-01]: Known preset is accepted

- **GIVEN** a seated connected player in a tourist room
- **WHEN** that player submits a say intent with a whitelist preset identifier
- **THEN** the server accepts the intent
- **AND** every client in the room observes a say event for that seat with that preset identifier

#### Scenario [SC-SAY-02]: Non-whitelist payload is rejected

- **GIVEN** a seated connected player in a tourist room
- **WHEN** that player submits a say intent with free-form text or an unknown preset identifier
- **THEN** the server rejects the intent without broadcasting a say event
- **AND** no client shows a new speech bubble from that intent

### Requirement: Who may send a say

The server SHALL accept a say intent only from a client that currently holds a seat and is connected (not offline within reconnect grace). Spectators MUST NOT successfully send say intents. Sending MUST NOT require that it is the sender’s current turn. An offline seat within reconnect grace MUST NOT successfully send say intents.

#### Scenario [SC-SAY-03]: Seated connected player may say off-turn

- **GIVEN** a tourist room with at least two seated players and the current turn belonging to another seat
- **WHEN** a seated connected player who does not hold the turn submits a whitelist say intent
- **THEN** the server accepts and broadcasts the say event

#### Scenario [SC-SAY-04]: Spectator cannot say

- **GIVEN** a spectator client in a tourist room that has at least one seated player
- **WHEN** that spectator submits a say intent with a whitelist preset identifier
- **THEN** the server rejects the intent without broadcasting

#### Scenario [SC-SAY-05]: Offline grace seat cannot say

- **GIVEN** a seated player whose seat is offline within reconnect grace
- **WHEN** a say intent is submitted for that session while the seat remains offline
- **THEN** the server rejects the intent without broadcasting

### Requirement: Broadcast to all room clients

On an accepted say, the server SHALL notify every client currently in the tourist room, including seated players and spectators. Each client MUST show the speech bubble against the sender’s presence marker using that client’s local presence slot layout for the sender’s seat (`game/presence`).

#### Scenario [SC-SAY-06]: Spectators see seated player say

- **GIVEN** a tourist room with a seated connected sender and at least one spectator
- **WHEN** the sender successfully says a whitelist preset
- **THEN** the spectator observes the say event for that sender’s seat
- **AND** the seated clients also observe the same say event

### Requirement: Own-marker affordance and preset picker

While the user is seated and connected on the Game screen, the system SHALL show a speech-bubble affordance only next to that user’s own presence marker. Activating the affordance MUST reveal exactly two preset choices with the display texts «Всем привет» and «Удачи». Choosing a preset MUST close the picker immediately and submit that preset’s identifier. Spectators MUST NOT see a send affordance. Other players’ markers MUST NOT show a send affordance for the local user.

#### Scenario [SC-SAY-07]: Seated player opens picker on own marker

- **GIVEN** the user is seated and connected on the Game screen
- **WHEN** the user activates the speech-bubble affordance on their own presence marker
- **THEN** two preset choices «Всем привет» and «Удачи» are shown
- **AND** choosing one closes the picker immediately

#### Scenario [SC-SAY-08]: Spectator has no send affordance

- **GIVEN** the user is a spectator on the Game screen
- **WHEN** the Game presence layout is shown
- **THEN** no say send affordance is available to that user

### Requirement: Speech bubble lifetime and concurrency

Each accepted say MUST appear as a comic-style speech bubble near the sender’s presence marker and MUST disappear after **10 seconds**. A sender MUST have at most **three** live bubbles at once. While three live bubbles exist for that sender, further say intents from that sender MUST be rejected until at least one bubble’s lifetime has ended. After one bubble expires, the sender MAY send again subject to the same limit. The server MUST enforce the same concurrent limit; the client MUST also respect it for local UX.

#### Scenario [SC-SAY-09]: Bubble disappears after ten seconds

- **GIVEN** a successful say for a seated sender
- **WHEN** ten seconds elapse since that say was accepted
- **THEN** that speech bubble is no longer shown on any client

#### Scenario [SC-SAY-10]: Fourth concurrent say is rejected

- **GIVEN** a seated connected sender already has three live say bubbles
- **WHEN** that sender submits another whitelist say intent before any of those three expire
- **THEN** the server rejects the intent without broadcasting a fourth bubble
- **AND** after one of the three expires, a subsequent whitelist say from that sender MAY be accepted

### Requirement: Bubble stack orientation by presence slot

Speech bubbles for a seat MUST stack next to that seat’s presence marker according to the marker’s local slot. Newer bubbles MUST appear closer to the avatar than older ones. For markers on the left or right sides, newer bubbles MUST appear lower and older bubbles higher. Bubbles MUST NOT use viewport toast notifications as their primary presentation.

#### Scenario [SC-SAY-11]: Side slots stack newer below

- **GIVEN** a sender whose presence marker is on the left or right for the viewing client and that sender already has one live bubble
- **WHEN** a second say from that sender is accepted
- **THEN** the newer bubble is shown lower (closer toward the bottom of the stack)
- **AND** the older bubble is higher

#### Scenario [SC-SAY-12]: Home and opposite slots keep newer closer to avatar

- **GIVEN** a sender whose presence marker is at the bottom or top for the viewing client
- **WHEN** multiple live bubbles exist for that sender
- **THEN** the newest bubble is visually closer to the avatar than older bubbles
- **AND** bubbles are attached beside that presence marker rather than as screen-edge toasts
