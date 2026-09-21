## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-LOBBY-19 | covered (client LobbyPage join busy-lock) |
| SC-LOBBY-20 | covered (client game store clear rooms on leave/resubscribe) |

## ADDED Requirements

### Requirement: Join action is busy-locked

While a lobby join into a listed `tourist` room is in progress, the client MUST prevent a second join attempt from the same listing (including repeated activation of the same or another listed row). The busy state MUST be visible on the join control. Create-game MAY continue to use its existing modal busy pattern; join MUST NOT leave listing rows fully clickable without a re-entrancy guard.

#### Scenario [SC-LOBBY-19]: Second join click ignored while connecting

- **GIVEN** the user is on the lobby screen with at least one listed room
- **WHEN** the user starts join for a listed room and activates join again before the first attempt finishes
- **THEN** only one join attempt proceeds
- **AND** the join control shows a busy/loading state during the attempt

### Requirement: Lobby listing clears stale rooms on resubscribe

When the client leaves a tourist game and returns to the lobby listing (or otherwise resubscribes to the live lobby list), the client MUST NOT briefly show a stale prior room list as if it were the current live snapshot. The listing MUST either stay in a loading/empty-safe state until a fresh lobby rooms snapshot arrives, or clear the previous rooms collection before presenting rooms again after resubscribe.

#### Scenario [SC-LOBBY-20]: No ghost room flash after leaving last seat

- **GIVEN** the user was the last seated player in a tourist room and returns to the lobby screen
- **WHEN** the lobby listing resubscribes
- **THEN** the user MUST NOT see a flash of that disposed room as an available game between loading and the fresh empty (or updated) list
- **AND** after the fresh lobby snapshot, disposed rooms are absent from the list
