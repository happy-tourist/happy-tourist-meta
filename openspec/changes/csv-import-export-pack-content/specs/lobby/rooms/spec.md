## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-31 | covered (client vitest LobbyCreateWire selected mini) |

Related: create-game map picker — main `lobby/rooms` (map option capacity captions). This change requires the **selected** map state to show a mini preview, not only the open option list.

## ADDED Requirements

### Requirement: Create map select shows mini preview when selected

When an authenticated user opens create-game and selects an in-catalog map, the closed/selected presentation of the map picker MUST show a **mini preview** of that map’s grid together with the map’s identifying label (not label-only text). Option rows in the open list MUST continue to show mini preview and players×tourists per existing create-map picker rules.

#### Scenario [SC-LOBBY-31]: Selected map shows mini preview in create select

- **GIVEN** the create-game modal with an in-catalog map M selected
- **WHEN** the user views the closed map select
- **THEN** the selected presentation includes a mini preview of M’s grid
- **AND** the map’s label remains visible
