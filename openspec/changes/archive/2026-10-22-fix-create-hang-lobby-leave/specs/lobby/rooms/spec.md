# Delta: lobby/rooms

Связано с main `openspec/specs/lobby/rooms/spec.md` (leave lobby on game enter). Этот change уточняет: успешный вход в `tourist` и навигация на игровой экран не блокируются ожиданием завершения close lobby subscription.

## Traceability

| ID | Coverage |
|----|----------|
| SC-LOBBY-04 | covered-by-reuse (main — create/join `tourist` + listing eligibility; **no server delta** in this change) |
| SC-LOBBY-05 | covered (client vitest `game.lobbyEnter.test.ts` — enter resolves without awaiting hang leave) |
| SC-LOBBY-06 | covered-by-reuse (unchanged — resubscribe on return) |
| SC-LOBBY-08 | covered-by-reuse (main — LobbyRoom no reconnect hold; hang leave / orphan WS cleanup unchanged on server) |
| SC-LOBBY-37 | covered (client vitest `game.lobbyEnter.test.ts` — create/join UX completes when leave hangs past bound) |

**FE→BE:** UI create/join/navigate опирается на уже описанный в main server/room контракт (`lobby` / `tourist` + listing leave via Colyseus `leave()`); этот delta не добавляет полей/сообщений/HTTP и не требует server-side leave-timeout — только client timing leave (initiate + short bound; orphan listing WS уходит по disconnect / next subscribe, SC-LOBBY-08). **BE→FE:** новых FE-потребляемых server claims нет (server §0 none) — `[be-without-fe]` n/a.

## MODIFIED Requirements

### Requirement: Leave lobby listing when entering a game

While the user is on the game screen in an active `tourist` session, the client SHALL NOT keep an active lobby listing subscription. When the user successfully enters a `tourist` game (create or join), the client MUST initiate closing the lobby listing subscription and MUST clear local lobby listing state as part of that initiate (so the hung leave does not keep the lobby connection as the active subscription), without waiting for the network close handshake. Successful entry for the user (end of create/join loading, navigation to the game screen with an active game room connection) MUST NOT wait for the lobby subscription close to finish on the network. The network close MAY complete asynchronously; if it does not finish within a short bound, the client MUST still treat entry as successful. The lobby listing subscription MAY be re-established when the user returns to the lobby screen.

#### Scenario [SC-LOBBY-05]: Unsubscribe before or on game enter

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **WHEN** the user successfully enters a `tourist` game (create / join by listed room)
- **THEN** the client initiates closing the lobby listing subscription
- **AND** the client navigates to the game screen with an active game room connection
- **AND** successful entry (end of create/join loading and navigation) does not require the lobby close handshake to complete first

#### Scenario [SC-LOBBY-06]: Resubscribe on return to lobby

- **GIVEN** the user left the lobby listing subscription when entering a game
- **WHEN** the user returns to the lobby screen without an active need for the game session listing
- **THEN** the client establishes a new lobby listing subscription and shows the current `tourist` room list

#### Scenario [SC-LOBBY-37]: Entry succeeds when lobby leave hangs

- **GIVEN** the user has an active lobby listing subscription on the lobby screen
- **AND** the lobby subscription close handshake will not complete promptly
- **WHEN** the user successfully creates or joins a `tourist` game
- **THEN** create/join loading ends and the client navigates to the game screen with an active game room connection
- **AND** local lobby listing state is cleared so the hung leave does not keep an active lobby subscription for the session
