## Purpose

Delta этого change: UX возврата с финиша — зелёная иконка над strip вместо модалки. Wire return и геометрия кольца — main `game/finish` / `game/move`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-13 | covered (client UX — green return icon; no confirm modal) |
| SC-FINISH-16 | covered (client UX — finished strip slot click does not start return) |

Related: return geometry and steps — `game/move`; strip dimming — main `game/finish`.

## MODIFIED Requirements

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST show a green return affordance centered above that tourist in the personal strip (same visual family as board rescue/push affordances). Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above) and MUST NOT show the return affordance. Activating the finished strip slot body (the tourist image area) MUST NOT open a confirmation dialog and MUST NOT enter return-mode by itself — only the return affordance starts return. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a green return affordance is shown centered above that tourist (same visual family as board rescue/push)
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation
