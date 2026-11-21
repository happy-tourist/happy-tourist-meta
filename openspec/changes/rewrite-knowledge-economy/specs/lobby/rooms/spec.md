# Delta: lobby/rooms

Фиксированный сид ловушек на сервере; create-game без выбора density.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-13 | removed — grille density UI |
| SC-LOBBY-14 | removed — grille density create option |
| SC-LOBBY-15 | removed — default grille density |
| SC-LOBBY-16 | removed — catapult density UI |
| SC-LOBBY-17 | removed — catapult density create option |
| SC-LOBBY-18 | removed — default catapult density |
| SC-LOBBY-40 | pending — client UX create without density |
| SC-LOBBY-41 | pending — create omits density; server ignores if sent |

## ADDED Requirements

### Requirement: Create game has no trap density choice

When an authenticated user opens create-game from the lobby, the system MUST NOT present grille density or catapult density presets (few/medium/many). Confirming create MUST create a `tourist` room without client-selected trap densities (create payload omits `grilleDensity` / `catapultDensity`). If those options are still present, the server MUST **ignore** them. Trap seed ratios are fixed server-side per `game/board` / `game/move` (grille/catapult partition formula). Other create options (map, pack, task sets, maxSeats) remain as previously specified.

#### Scenario [SC-LOBBY-40]: Create modal has no density controls

- **GIVEN** the user is on the lobby screen and opens create-game
- **WHEN** the create modal appears
- **THEN** there are no selectable grille density options
- **AND** there are no selectable catapult density options
- **AND** map, pack, task-set, and maxSeats choices remain available

#### Scenario [SC-LOBBY-41]: Create omits density; server uses fixed seed

- **GIVEN** the create modal is open with valid map/pack/task-set/maxSeats selections
- **WHEN** the user confirms create
- **THEN** the create payload omits grilleDensity and catapultDensity
- **AND** a new `tourist` room is created and seeded with the fixed exclusive 15%/15% partition (density options ignored if present)
- **AND** the user enters that game session

## REMOVED Requirements

### Requirement: Create game chooses grille density

**Reason**: Trap counts are fixed (15% grilles / 15% catapults); player choice removed.
**Migration**: Server always seeds grilles at 15% of task cells; omit create UI and options.

### Requirement: Create game chooses catapult density

**Reason**: Trap counts are fixed (15% catapults); player choice removed.
**Migration**: Server always seeds catapults at 15% of task cells; omit create UI and options.
