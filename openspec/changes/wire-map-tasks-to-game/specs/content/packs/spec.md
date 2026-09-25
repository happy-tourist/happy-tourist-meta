## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-32 | covered (unchanged — difficulty stored) |
| SC-PACK-33 | modified (packs feed peeks) |
| SC-PACK-200 | pending (server mocha — published sets in create) |
| SC-PACK-201 | pending (server mocha — soft-unpublished set excluded) |

Related: create picker — `lobby/rooms`; peek Q&A — `game/board`. Difficulty remains the peek step reward (`SC-PACK-32`).

## MODIFIED Requirements

### Requirement: Difficulty is stored for future peek rewards

Task difficulty values `1`, `2`, and `3` MUST be persisted with each task as the peek step reward when that task is bound and answered correctly in a tourist room (`game/board`). This capability’s CMS rules for storing difficulty MUST remain. Tourist-room peek MUST use published pack task content (question, ordered slots, answer cards) instead of the legacy stub Correct/Wrong controls.

#### Scenario [SC-PACK-32]: Difficulty is stored on the task

- **GIVEN** a verified editor saves a task with difficulty `2`
- **WHEN** the task is later retrieved
- **THEN** the stored difficulty is `2`

#### Scenario [SC-PACK-33]: Peek stub behavior unchanged

- **GIVEN** this capability is deployed and an in-catalog pack has a published task set selected at room create
- **WHEN** a player opens a peek in that tourist room
- **THEN** the peek uses that pack’s task question and answer slots per `game/board`
- **AND** the legacy stub-only Correct/Wrong modal without content MUST NOT be the sole peek UI

## ADDED Requirements

### Requirement: Only published task sets are playable at create

Create-game MUST offer only **published** (not soft-unpublished) task sets of an in-catalog pack. Soft-unpublished task sets MUST NOT appear in the create picker and MUST NOT be accepted in create options. The room MUST snapshot the selected sets’ tasks and the pack’s answer cards at create so later CMS edits do not change an in-progress match.

#### Scenario [SC-PACK-200]: Published set is selectable at create

- **GIVEN** in-catalog pack P with published task set S
- **WHEN** a user opens create-game and selects P
- **THEN** S appears in the task-set picker

#### Scenario [SC-PACK-201]: Soft-unpublished set is not selectable

- **GIVEN** in-catalog pack P with soft-unpublished task set S2 and published set S1
- **WHEN** a user opens create-game and selects P
- **THEN** S1 appears
- **AND** S2 does not appear
- **AND** create options referencing S2 are rejected by the server
