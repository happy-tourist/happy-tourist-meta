## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-26 | pending |

## ADDED Requirements

### Requirement: Seated top presence row reserves height

For a **seated** viewer, the layout region above the board that holds opponent presence markers MUST reserve a stable vertical space even when no opponent markers are currently shown (for example the viewer is alone in the room). When an opponent later appears, the board MUST NOT jump downward solely because that reserved region appears for the first time. Empty seat **avatar placeholders** MUST NOT be introduced (SC-PRESENCE-05 remains). Spectators keep their existing top-marker layout without this alone-in-room reservation rule.

#### Scenario [SC-PRESENCE-26]: Alone seated viewer keeps top band reserved

- **GIVEN** the local user is seated alone in a tourist room on the Game screen
- **WHEN** a second player becomes seated and an opponent marker appears above the board
- **THEN** the board’s vertical position does not jump by approximately one presence-row height solely due to the top row appearing
- **AND** no empty seat avatar placeholders are shown
