## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-SAY-01 | covered (server mocha — whitelist still includes greeting/luck) |
| SC-SAY-13 | pending (server mocha) |
| SC-SAY-14 | pending (client i18n / bubble) |

## MODIFIED Requirements

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
