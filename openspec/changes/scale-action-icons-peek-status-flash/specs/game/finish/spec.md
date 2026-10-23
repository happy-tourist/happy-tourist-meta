## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-09 | unchanged (flag indicator size stays as after board-action-icons wire — not scaled in this change) |
| SC-FINISH-13 | modified (return affordance ~2× illustrated glyph / larger hit; flag size unchanged) |

## MODIFIED Requirements

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with steps ≥ 1 and finish place 0, each finished side that has at least one legal ring cell MUST show a return affordance centered above that tourist in the personal strip (same visual family as board rescue/push affordances). The affordance MUST use the **illustrated board-action return icon** and MUST NOT paint a solid colored (including green) circular chrome behind the icon. The illustrated return glyph MUST render at about **twice** the prior board-action glyph size, and the hit target MUST be enlarged in the same family (~48–56 CSS pixels). The finished-strip **flag** indicator MUST keep its existing size (not scaled by this change). Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return. Steps MUST decrease only when the server accepts that return (`game/move`), not when ring highlights appear. Switching selection to another tourist MUST cancel return-mode without spending a step. When return is not available, the finished slot MUST remain non-actionable (see dimming above) and MUST NOT show the return affordance. Activating the finished strip slot body (the tourist image area) MUST NOT open a confirmation dialog and MUST NOT enter return-mode by itself — only the return affordance starts return. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with steps ≥ 1, finish place 0, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a return affordance with the illustrated return icon is shown centered above that tourist (same visual family as board rescue/push; no solid colored circular chrome)
- **AND** the return glyph / hit are about twice the prior board-action size family
- **AND** the finished-flag indicator size is unchanged by this scale
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation
