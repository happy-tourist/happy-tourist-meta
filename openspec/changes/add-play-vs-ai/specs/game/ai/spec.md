## Purpose

Режим партии против AI в room `tourist`: один человек и synthetic bot seats до `maxSeats`, Easy-соперники с human-like задержками, всегда верные ответы на задания; без отдельных клиентских процессов ботов.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-AI-01 | `test/MyRoom.test.ts` — "SC-AI-01: four-seat AI room seats one human and three bots" |
| SC-AI-02 | `test/MyRoom.test.ts` — "SC-AI-02 / SC-AI-10: maxSeats 1 coerces away from effective AI" |
| SC-AI-03 | `test/MyRoom.test.ts` — "SC-AI-03: bot rescues then moves after separate move-delays" |
| SC-AI-04 | `test/MyRoom.test.ts` — "SC-AI-04: bot submits a legal board action well before turn deadline"; `test/botEasy.test.ts` — "SC-AI-04: pickBestBoardAction prefers center finish over side step" |
| SC-AI-05 | `test/MyRoom.test.ts` — "SC-AI-05: bot answers open peek correctly after answer-delay" |
| SC-AI-06 | `test/MyRoom.test.ts` — "SC-AI-06: bot waits a move-delay before board action" |
| SC-AI-07 | `test/botEasy.test.ts` — "SC-AI-07: answer delay set is longer than move delay set" |
| SC-AI-08 | `test/MyRoom.test.ts` — "SC-AI-08: bot seats stay connected without reconnect grace" |
| SC-AI-09 | `test/MyRoom.test.ts` — "SC-AI-09: omitted opponentMode is people (no bots)" |
| SC-AI-10 | `test/MyRoom.test.ts` — "SC-AI-02 / SC-AI-10: maxSeats 1 coerces away from effective AI" |

Related: create / listing — `lobby/rooms`; fill seats / countdown — `game/start`; presence label — `game/presence`; seating / join gate — `game/pieces`; dispose on human leave — `game/leave`; turn / peek rules for humans unchanged — `game/move` / `game/board`; cut / knowledge rules unchanged — `game/finish`.

## ADDED Requirements

### Requirement: Create option opponentMode selects AI vs people

Create options for room `tourist` MUST accept `opponentMode` with values `'people' | 'ai'`. When the option is omitted or any other value, the server MUST treat the room as `'people'` (backward compatible with today’s create). **Effective AI mode** MUST mean `opponentMode === 'ai'` **and** `maxSeats` ≥ 2. When `maxSeats` equals 1, the server MUST coerce to effective people/solo: no bot seats, no AI unlist/private gate, no AI-only join deny — even if the client sent `opponentMode: 'ai'`.

#### Scenario [SC-AI-09]: Omitted opponentMode is people

- **GIVEN** an authenticated client creates a tourist room without opponentMode and with maxSeats ≥ 2
- **WHEN** the room is created
- **THEN** the room is effective people mode
- **AND** no bot seats are spawned

#### Scenario [SC-AI-10]: MaxSeats one coerces away from effective AI

- **GIVEN** an authenticated client creates a tourist room with maxSeats 1 and opponentMode `ai`
- **WHEN** the creator is seated
- **THEN** the room is effective people/solo (no bots)
- **AND** AI unlist / AI join-deny gates do not apply beyond normal solo seating

### Requirement: AI opponent mode fills remaining seats with bots

When a `tourist` room is in **effective AI mode**, the server MUST seat the creating authenticated human client and MUST fill every remaining seat up to `maxSeats` with **bot seats** that count toward occupancy the same as human seats. Each bot seat MUST sync `isBot: true` (human seats `isBot: false`). Bot seats MUST NOT be backed by separate Colyseus client connections. Synthetic seat map keys MAY use a stable `bot:N` form; they MUST participate in turn order like human sessionIds. Each bot seat MUST receive a unique `touristId` from the same `1…4` pool used for human seats (no duplicate kinds at the table). Board/peek mutations for bots MUST go through the same sessionId-scoped authoritative helpers as human message handlers (no fake `Client`).

#### Scenario [SC-AI-01]: Four-seat AI room seats one human and three bots

- **GIVEN** an authenticated client creates a tourist room with opponentMode `ai` and maxSeats 4
- **WHEN** the creator is seated
- **THEN** the room has exactly four seats
- **AND** exactly one seat is the human creator with isBot false
- **AND** the other three seats are bot seats with isBot true and without separate client connections

#### Scenario [SC-AI-02]: Single-seat create ignores AI mode for bots

- **GIVEN** an authenticated client creates a tourist room with maxSeats 1 and opponentMode `ai`
- **WHEN** the creator is seated
- **THEN** the room has exactly one human seat
- **AND** no bot seats exist
- **AND** unlist / AI join-deny gates do not apply (same coerce as SC-AI-10)

### Requirement: Easy bot plays as a selfish human-like rival

While phase is `playing` and it is a bot seat’s turn, the server MUST drive that seat’s board play under one **Easy** difficulty using selfish rival heuristics: free own rescue when legal before the board action; legal return-from-finish when that improves delivered/carried knowledge prospects; otherwise a legal move that advances knowledge farming and/or finish/cut pressure against remaining eligible seats. Bots MUST NOT cooperate as a team against the human. Harder difficulties and full optimal search MUST NOT be required in this capability.

#### Scenario [SC-AI-03]: Bot rescues then takes a board action

- **GIVEN** it is a bot seat’s turn, that seat has a trapped own piece with a legal free own rescue, and a legal move exists afterward
- **WHEN** the bot controller acts for that turn
- **THEN** the bot waits a move-delay sample, performs the rescue before spending its board action
- **AND** then waits another move-delay sample and performs one legal move or return-from-finish

#### Scenario [SC-AI-04]: Bot prefers knowledge and cut pressure over idle timeout

- **GIVEN** it is a bot seat’s turn with at least one legal board action and no pending open task modal for that seat
- **WHEN** the bot controller chooses an action
- **THEN** the bot submits a legal rescue/move/return rather than waiting for turn-deadline expiry as its only strategy
- **AND** the choice follows Easy selfish heuristics (knowledge delivery / finish / cut pressure), not random no-op

### Requirement: Bot resolves open tasks correctly after a human-like delay

When an open-on-land task modal belongs to a bot seat, the server MUST resolve that task as **correct** after a human-like delay sampled from a configured answer-delay set (longer than typical bot move delays). The bot MUST NOT submit an incorrect answer as Easy policy. Resolution MAY set correct placements and call the same internal correct-path as human `peekSubmit` / `resolveOpenPeek(true)`. Human task rules and messages remain as in `game/board` / `game/move`.

#### Scenario [SC-AI-05]: Bot answers correctly after delay

- **GIVEN** a bot seat’s piece has opened a shared task modal on its turn
- **WHEN** the bot answer delay elapses
- **THEN** the task resolves as correct for that bot seat
- **AND** knowledge / cell outcomes follow the existing correct-path rules

### Requirement: Bot move timing uses human-like delays

Before a bot performs a rescue, move, or return-from-finish on its turn, the server MUST wait a human-like delay sampled from a configured move-delay set. When a turn needs both a rescue and a subsequent board action, **each** of those actions MUST wait its own independent sample from the move-delay set. Move delays MUST typically be shorter than answer delays so answering feels slower than clicking a destination. Delays MUST be chosen from finite arrays (random pick), not a single fixed constant for every action. Canonical Easy arrays (design): `MOVE_DELAY_MS = [600, 900, 1100, 1400]`, `ANSWER_DELAY_MS = [1200, 1600, 2000, 2500, 3000]`.

#### Scenario [SC-AI-06]: Bot waits before moving

- **GIVEN** it becomes a bot seat’s turn with a legal board action available and no open task modal for that seat
- **WHEN** the bot is about to act
- **THEN** the action is not applied instantly at turn start
- **AND** the wait is drawn from the move-delay set

#### Scenario [SC-AI-07]: Answer delays are longer than move delays

- **GIVEN** the configured Easy delay sets for moves and answers
- **WHEN** those sets are compared
- **THEN** typical answer delays are longer than typical move delays
- **AND** both sets contain more than one value for random selection

### Requirement: Bots stay connected; bots alone do not keep the room

Bot seats MUST remain treated as connected for gameplay (no reconnect grace for bots). For dispose rules in effective AI mode (`game/leave` SC-LEAVE-20…22): after the sole human has permanently left, remaining bot seats MUST NOT keep the room alive — the server MUST dispose immediately even though bot seats are still present in state until dispose.

#### Scenario [SC-AI-08]: Bot seat does not enter reconnect grace

- **GIVEN** an effective-AI tourist room in phase `playing` with bot seats
- **WHEN** gameplay continues without a human disconnect path for those bots
- **THEN** bot seats remain eligible connected seats for turns
- **AND** bots do not arm human reconnect-grace timers
