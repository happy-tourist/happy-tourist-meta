## Purpose

Lobby create UX for people vs AI opponent mode, and live-listing omission for **effective AI** rooms. Server create coerce / bots — `game/ai`; seating — `game/pieces`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-50 | `src/pages/__tests__/LobbyCreateWire.test.ts` (client) — "SC-LOBBY-50: create modal shows people vs AI radio…" |
| SC-LOBBY-51 | `src/pages/__tests__/LobbyCreateWire.test.ts` (client) — "SC-LOBBY-51: confirm create sends opponentMode ai…" |
| SC-LOBBY-52 | `src/pages/__tests__/LobbyCreateWire.test.ts` (client) — "SC-LOBBY-52: maxSeats 1 + AI still sends opponentMode ai…" |
| SC-LOBBY-53 | `test/MyRoom.test.ts` — "SC-LOBBY-53: AI room never appears in lobby list" |

## ADDED Requirements

### Requirement: Create game offers human vs AI opponent mode

When an authenticated user opens create-game on the lobby screen and has selected an in-catalog map (so maxSeats is shown), the system MUST present a two-option opponent-mode control **immediately after** the maxSeats control (product sense: play with **people** vs play with **AI**). Options MUST use distinct Quasar Material icons already in the client bundle: `person` (people) and `smart_toy` (AI), with localized labels (product sense «Люди» / «AI»). The default selection when the create modal opens or resets MUST be **people**. Confirming create MUST send create option `opponentMode: 'people' | 'ai'` with the existing map / pack / task sets / maxSeats payload (client MUST NOT coerce away `ai` when maxSeats is 1). When the chosen maxSeats equals 1, selecting AI MUST still create a normal single-seat solo room (server coerces to effective people/solo — no bot seats, no AI unlist — see `game/ai`).

#### Scenario [SC-LOBBY-50]: Create modal shows people vs AI radio

- **GIVEN** the user is on the lobby screen, opens create-game, and has selected an in-catalog map
- **WHEN** the create modal shows maxSeats
- **THEN** a people-vs-AI opponent-mode control is visible immediately after maxSeats with `person` and `smart_toy` icons
- **AND** the default selection is people
- **AND** the user can select either mode before confirm

#### Scenario [SC-LOBBY-51]: Confirm create sends AI mode with seats

- **GIVEN** the create modal has a valid map, pack, at least one task set, maxSeats 3, and AI opponent mode selected
- **WHEN** the user confirms create
- **THEN** the client creates a tourist room with create option opponentMode `ai` and maxSeats 3
- **AND** the user navigates to that game session

#### Scenario [SC-LOBBY-52]: MaxSeats one keeps solo regardless of AI selection

- **GIVEN** the create modal has maxSeats 1 and AI opponent mode selected
- **WHEN** the user confirms create
- **THEN** the created room has a single human seat and no bot seats
- **AND** start follows the existing single-seat full-table path (`game/start`)
- **AND** the room is not treated as effective AI for listing/unlist gates

### Requirement: AI opponent rooms are omitted from live lobby listing

A `tourist` room in **effective AI mode** (`opponentMode: 'ai'` and `maxSeats` ≥ 2; see `game/ai`) MUST NOT appear in the live lobby listing for any subscriber (server: `setPrivate(true)` or Colyseus 0.18 equivalent). Human-vs-human rooms and coerced solo (`maxSeats` 1) continue to list as today. Join-by-listed-room MUST NOT be offered for effective-AI rooms because they are not listed.

#### Scenario [SC-LOBBY-53]: AI room never appears in lobby list

- **GIVEN** at least one client is subscribed to the lobby listing for `tourist`
- **WHEN** a new tourist room is created with opponentMode `ai` and maxSeats ≥ 2
- **THEN** subscribed lobby clients do not receive that room in the listing
- **AND** human-mode rooms created afterward still appear as usual
