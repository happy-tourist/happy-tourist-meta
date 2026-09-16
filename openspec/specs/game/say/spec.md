# game/say Specification

## Purpose

Пресетные реплики у presence на Game: короткие фразы из фиксированного whitelist рассылаются всем в room `tourist` и показываются комикс-облаками у маркера отправителя без свободного текста и без истории чата.

Связанные capability: раскладка presence и углы affordance — `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-01 | covered (server mocha) |
| SC-SAY-02 | covered (server mocha) |
| SC-SAY-03 | covered (server mocha) |
| SC-SAY-04 | covered (server mocha) |
| SC-SAY-05 | covered (server mocha) |
| SC-SAY-06 | covered (server mocha) |
| SC-SAY-07 | covered (client say affordance top-right on own marker) |
| SC-SAY-08 | covered (client UX GamePage) |
| SC-SAY-09 | covered (client UX GamePage) |
| SC-SAY-10 | covered (server mocha) |
| SC-SAY-11 | covered (client bubbles toward board — replaces side-slot stack) |
| SC-SAY-12 | covered (client bubbles toward board; newer closer to avatar) |
| SC-SAY-13 | covered (server mocha) |
| SC-SAY-14 | covered (client i18n / bubble) |
| SC-SAY-15 | covered (client say affordance/picker activatable — no overflow clip) |

Related: presence row layout and affordance corners — `game/presence`.

## Requirements

### Requirement: Whitelist-only preset say

The system SHALL accept only a fixed set of preset identifiers for say intents in the tourist room. The product presets are: the greeting preset (client display text «Всем привет»), the luck preset (client display text «Удачи»), and the readiness preset (client display text «Готов начать!»). Free-form or arbitrary text MUST NEVER be accepted as a say payload. Unknown, missing, or non-whitelist identifiers MUST be rejected without broadcasting. A successful ready intent in `game/start` MUST result in a broadcast of the readiness preset through the same say channel (same bubble UX as other presets).

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

#### Scenario [SC-SAY-13]: Readiness preset is on the whitelist

- **GIVEN** a seated connected player in a tourist room
- **WHEN** a say event with the readiness preset identifier is broadcast (including as a result of a successful ready intent)
- **THEN** every client may show the speech bubble for that preset
- **AND** the client display text for that preset is «Готов начать!» (localized per client locale files)

#### Scenario [SC-SAY-14]: Readiness bubble uses say UX

- **GIVEN** a successful ready intent that broadcasts the readiness preset
- **WHEN** other clients in the room receive that say event
- **THEN** they show it as a comic bubble at the sender’s presence marker like other say presets
- **AND** no separate chat history is created

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

While the user is seated and connected on the Game screen, the system SHALL show a speech-bubble affordance only at the **top-right** of that user’s own presence marker. Activating the affordance MUST reveal exactly two preset choices with the display texts «Всем привет» and «Удачи». Choosing a preset MUST close the picker immediately and submit that preset’s identifier. Spectators MUST NOT see a send affordance. Other players’ markers MUST NOT show a send affordance for the local user. The affordance and its open picker MUST remain pointer- and touch-activatable: presence row overflow MUST NOT clip them out of hit-testing or hide the open picker, and the affordance hit target MUST be large enough for touch (at least about 32 CSS pixels on each side).

#### Scenario [SC-SAY-07]: Seated player opens picker on own marker

- **GIVEN** the user is seated and connected on the Game screen
- **WHEN** the user activates the speech-bubble affordance at the top-right of their own presence marker
- **THEN** two preset choices «Всем привет» and «Удачи» are shown
- **AND** choosing one closes the picker immediately

#### Scenario [SC-SAY-08]: Spectator has no send affordance

- **GIVEN** the user is a spectator on the Game screen
- **WHEN** the Game presence layout is shown
- **THEN** no say send affordance is available to that user

#### Scenario [SC-SAY-15]: Affordance and picker stay activatable

- **GIVEN** the user is seated and connected with their own presence marker visible (top or bottom row)
- **WHEN** the user taps or clicks the speech-bubble affordance
- **THEN** the preset picker becomes visible without being clipped away by the presence row
- **AND** the affordance itself receives the activation (is not blocked by rings, avatar, or row overflow)
- **AND** the user can choose a preset from the picker

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

Speech bubbles for a seat MUST stack against that seat’s presence marker on the side toward the board: for markers in the top presence row, bubbles appear below the avatar; for the seated viewer’s own marker in the bottom row, bubbles appear above the avatar. Newer bubbles MUST appear closer to the avatar than older ones. Horizontal spacing between neighboring presence markers MUST keep concurrent bubbles of adjacent seats from overlapping. Bubbles MUST NOT use viewport toast notifications as their primary presentation. Left/right side-slot stack rules MUST NOT apply (those slots are removed by `game/presence`).

#### Scenario [SC-SAY-11]: Side slots stack newer below

- **GIVEN** a sender whose presence marker is in the top row for the viewing client and that sender already has one live bubble
- **WHEN** a second say from that sender is accepted
- **THEN** both bubbles appear below that avatar toward the board
- **AND** the newer bubble is closer to the avatar than the older bubble
- **AND** no left/right side-column presence slot is used for bubble orientation

#### Scenario [SC-SAY-12]: Home and opposite slots keep newer closer to avatar

- **GIVEN** a seated viewer whose own presence marker is in the bottom row and multiple live bubbles exist for that seat
- **WHEN** any client views that marker
- **THEN** the bubbles appear above the avatar toward the board
- **AND** the newest bubble is visually closer to the avatar than older bubbles
- **AND** bubbles are attached beside that presence marker rather than as screen-edge toasts
