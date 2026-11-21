# Delta: game/presence

Knowledge HUD вместо steps/peeks; focus на месте end-turn; без таймера/solo modal/end-turn.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-06…07 | modified — no numeric finish-place badge |
| SC-PRESENCE-09 | removed — solo leftover red ring |
| SC-PRESENCE-10 | modified — reconnect ring only (no turn ring) |
| SC-PRESENCE-14 | modified — no finish-place badge corners |
| SC-PRESENCE-15…16 | removed — steps/peeks counters |
| SC-PRESENCE-17…18 | removed — end-turn control |
| SC-PRESENCE-19…21 | removed — solo unlimited / solo end modals |
| SC-PRESENCE-25 | removed — end-turn vs budgets layout |
| SC-PRESENCE-30…33 | modified — focus @3 o’clock; no end-turn |
| SC-PRESENCE-37 | modified — focus @3 stays fully visible |
| SC-PRESENCE-38…42 | removed — single-player budget/end-turn gates |
| SC-PRESENCE-50 | pending — seat knowledge on avatar |
| SC-PRESENCE-51 | pending — +N knowledge grant animation |
| SC-PRESENCE-52 | pending — tourist carried knowledge yellow circle |
| SC-PRESENCE-53 | pending — focus at former end-turn slot |
| SC-PRESENCE-54 | pending — no turn countdown ring |
| SC-PRESENCE-55 | pending — no end-turn control |
| SC-PRESENCE-56 | pending — no finish-place badge in presence |

## ADDED Requirements

### Requirement: Own delivered knowledge beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s **delivered knowledge** total (points transferred from finished tourists, from private `knowledge.delivered`) in a counter **to the right of** that user’s own presence avatar in the bottom HUD. Spectators and other seats MUST NOT see another seat’s private delivered total — **own-seat only** (same privacy family as former budgets). Steps and peeks counters MUST NOT appear.

#### Scenario [SC-PRESENCE-50]: Seated user sees own knowledge total

- **GIVEN** two seated players in phase `playing` with different delivered knowledge totals
- **WHEN** each views Game presence
- **THEN** each sees their own knowledge total to the right of their own avatar
- **AND** neither sees steps or peeks counters

#### Scenario [SC-PRESENCE-51]: Knowledge grant animation lasts about two seconds

- **GIVEN** the user’s delivered knowledge increases because a tourist finished and transferred points
- **WHEN** the local +N fall animation plays into the avatar knowledge counter
- **THEN** the animation completes in about two seconds
- **AND** other clients do not see that private animation

### Requirement: Carried knowledge on the tourist chrome

While an unfinished own piece carries knowledge greater than 0 (from private `knowledge.pieces`), the Game board and personal strip MUST show that carried amount in a **yellow-background circle** at the **bottom-right** of that tourist’s chrome (product sense: panel bottom-right glow). When knowledge is 0, the circle MUST NOT appear. When the piece finishes and knowledge transfers to the avatar, the circle MUST clear with the finish presentation. Opponents’ carried amounts MUST NOT be shown from another seat’s private payload.

#### Scenario [SC-PRESENCE-52]: Tourist shows yellow knowledge circle

- **GIVEN** an own unfinished piece has knowledge 2
- **WHEN** the owner views the board and strip
- **THEN** that piece shows a yellow-background circular badge at bottom-right with value 2

### Requirement: Focus control at the former end-turn position

While the local user is a seated current-turn player in phase `playing` and at least one own unfinished free piece can take a legal move, return, or rescue, the Game presence chrome MUST show the **icon-only focus** control with its hit-center on the own avatar circumference at the **right-center** edge (product clock sense: **3 o’clock** — the former end-turn slot). Glyph and hit MUST remain **36×36** illustrated focus icon without solid colored circular chrome. Activating it MUST focus/select an actionable own piece as previously specified for focus. The end-turn control MUST NOT appear. Focus MUST NOT sit in the knowledge counter stack.

#### Scenario [SC-PRESENCE-53]: Focus sits at three o’clock

- **GIVEN** it is the user’s turn with at least one actionable own piece and no task modal open
- **WHEN** the user views Game chrome
- **THEN** the focus control is shown at 3 o’clock on the own avatar
- **AND** no end-turn control is shown

#### Scenario [SC-PRESENCE-55]: End-turn control never shown

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown

### Requirement: No turn countdown ring

While phase is `playing`, presence markers MUST NOT show a determinate turn-budget countdown ring. Reconnect grace countdown on offline seats remains as previously specified. Dual turn+grace rings MUST NOT apply because there is no turn ring.

#### Scenario [SC-PRESENCE-54]: Current turn has no timer ring

- **GIVEN** it is a seated player’s turn in phase `playing`
- **WHEN** any client views that seat’s presence marker
- **THEN** there is no turn countdown ring around the avatar
- **AND** current-turn highlighting without a timer ring MAY remain

### Requirement: No finish-place badge in presence

While phase is `playing`, presence markers (top board markers and bottom own HUD) MUST NOT show a numeric finish-place badge derived from `finishPlace`. Schema `finishPlace > 0` remains the finished/cut gate for turn eligibility, but the client MUST NOT render place numbers or «Вы N-й!» congratulation chrome. Finished/cut outcome UX is the delivered-knowledge modal from `game/finish` only.

#### Scenario [SC-PRESENCE-56]: Finished seat shows no place badge

- **GIVEN** a seated player has `finishPlace = 1` after full finish or penultimate cut
- **WHEN** any client views that seat’s presence marker
- **THEN** no numeric place badge is shown on that marker
- **AND** the finished seat’s own client uses the knowledge modal for outcome copy, not a place string

## MODIFIED Requirements

### Requirement: Finish place badge on presence marker

When a seated player has synchronized `finishPlace > 0` (finished or cut gate), clients MUST NOT display a numeric finish-place badge (1, 2, …) or «Вы N-й!» chrome on that seat’s presence marker. Schema `finishPlace` remains the finished/cut eligibility gate. Outcome UX is the delivered-knowledge modal from `game/finish` only.

#### Scenario [SC-PRESENCE-06]: Finished seat shows place on presence

- **GIVEN** a seated player has `finishPlace = 1` after full finish or penultimate cut and is connected
- **WHEN** any client views that seat’s presence marker on Game
- **THEN** no numeric place badge is shown on that marker
- **AND** the tourist kind image remains visible in the marker

#### Scenario [SC-PRESENCE-07]: Non-finished seat has no place badge

- **GIVEN** a seated player who has unfinished pieces and `finishPlace` 0
- **WHEN** any client views that seat’s presence marker
- **THEN** no finish-place badge is shown on that marker

### Requirement: Dual turn and reconnect countdowns

When a seat is offline within reconnect grace, the presence marker MUST show the reconnect grace countdown ring. Because there is no turn deadline ring, dual turn+grace rings MUST NOT apply. Spectators and seated viewers MUST see the same reconnect display when offline.

#### Scenario [SC-PRESENCE-10]: Offline current-turn seat shows both rings

- **GIVEN** a current-turn seat that is offline within reconnect grace
- **WHEN** any client views that seat’s presence marker
- **THEN** the reconnect grace countdown ring is visible
- **AND** no turn-deadline countdown ring is shown
- **AND** the tourist kind image remains visible in the marker

### Requirement: Presence affordance and badge corners

While the local user may send a ready intent from the Game screen, the ready affordance MUST appear at the **top-left** of that user’s own presence marker only. While the local user may send a say intent, the say send affordance MUST appear at the **top-center** of that user’s own presence marker only (product clock sense: **12 o’clock** on the avatar). Finished/cut seats MUST NOT show a numeric finish-place badge at top-left (or elsewhere). When ready and say are both visible on the own marker, their hit boxes MUST NOT overlap. Other players’ markers MUST NOT show a say send affordance for the local user. Spectators MUST NOT see a say send affordance.

#### Scenario [SC-PRESENCE-14]: Finish, ready, and say corners

- **GIVEN** a seated connected user viewing Game with their own marker and at least one opponent marker that has `finishPlace = 1`
- **WHEN** the presence chrome is shown
- **THEN** the opponent marker shows no numeric finish-place badge
- **AND** if the ready affordance is available for the local user, it is at the top-left of the local user’s marker
- **AND** the say send affordance is at the top-center (12 o’clock) of the local user’s marker only
- **AND** when ready and say are both visible, their hit boxes do not overlap
- **AND** opponent markers do not show a say send affordance for the local user

### Requirement: Focus actionable tourist control

While the local user is a seated current-turn player in phase `playing` and at least one of their unfinished free pieces can currently take a legal move, return, or rescue (and the local board-action spent flag is clear), the Game presence chrome MUST show an **icon-only** focus control with its hit-center on the own avatar circumference at the **right-center** edge (product clock sense: **3 o’clock** — the former end-turn slot). The control MUST use the **illustrated board-action focus icon** and MUST NOT paint a solid colored circular chrome behind the icon. Glyph and hit MUST both be **36×36 CSS pixels**. Focus MUST NOT sit in the knowledge counter stack and MUST NOT sit at the old bottom-right-under-end-turn placement. Activating it MUST select / focus an actionable own unfinished free piece. When no own piece has an available action (including after board-action spent, or while a task modal is open), the control MUST be hidden or disabled. Spectators and non-current seats MUST NOT see an enabled control for themselves. Say remains on the **top-center** circumference (~**12 o’clock**). End-turn MUST NOT appear.

#### Scenario [SC-PRESENCE-30]: Focus control sits between say and end-turn

- **GIVEN** it is the local seated user’s turn with at least one actionable own tourist and no task modal open
- **WHEN** the own presence chrome is shown
- **THEN** a focus control with the illustrated focus icon is present at right-center (3 o’clock) on the own avatar
- **AND** the glyph / hit use the 36×36 glyph-fills-hit family
- **AND** no end-turn control is shown
- **AND** say (when visible) is at top-center (12) on the marker circumference
- **AND** say and focus hits do not overlap

#### Scenario [SC-PRESENCE-31]: Focus selects nearest actionable tourist

- **GIVEN** it is the local user’s turn with two own unfinished free pieces where only piece B can move or be rescued and piece A cannot
- **WHEN** the user activates the focus control
- **THEN** piece B becomes selected / focused for board actions
- **AND** piece A is not selected

#### Scenario [SC-PRESENCE-32]: Focus hidden when nothing actionable

- **GIVEN** it is the local user’s turn and no own unfinished free piece has a legal move, rescue, or return (or the board-action spent flag is set)
- **WHEN** the own presence chrome is shown
- **THEN** the focus control is not available as an enabled action

### Requirement: Focus under end-turn stays fully visible

While the focus control is visible on the own presence marker in the sticky bottom HUD (at 3 o’clock on the avatar circumference), every client MUST keep the entire focus hit target inside the visible, pointer-reachable chrome of that HUD (not clipped by the panel or its horizontal scrollport).

#### Scenario [SC-PRESENCE-37]: Focus hit under end-turn stays fully visible

- **GIVEN** it is the local seated user’s turn with the focus control visible at 3 o’clock on the own marker
- **WHEN** the user views the own presence chrome
- **THEN** the entire focus hit is visible without being clipped by the HUD panel or its horizontal scrollport
- **AND** the user can activate the focus control

### Requirement: Say affordance sits above the focus control

On the local seated user’s own presence avatar, when say is visible its **hit-center MUST sit on the marker circumference** at **12** (top-center). Focus, when visible, MUST sit at **3** (right-center) on the circumference (former end-turn slot) and MUST NOT overlap say. End-turn MUST NOT appear. This does not change say whitelist or `game/say` message rules. Hits and glyphs are **36×36** (glyph fills the hit).

#### Scenario [SC-PRESENCE-33]: Say is above focus without overlap

- **GIVEN** it is the local seated user’s turn with say available and at least one actionable own tourist (focus visible)
- **WHEN** the own presence chrome is shown
- **THEN** the say affordance hit-center is at top-center (12) and focus at right-center (3) on the avatar circumference
- **AND** no end-turn control is shown
- **AND** the say and focus hit areas do not overlap

## REMOVED Requirements

### Requirement: Own step and peek counters beside the avatar

**Reason**: Replaced by delivered knowledge counter.
**Migration**: Show knowledge; remove steps/peeks UI and ∞ solo display.

### Requirement: End-turn control next to own avatar

**Reason**: End-turn removed; focus uses that slot.
**Migration**: Focus at 3 o’clock; no end-turn submit.

### Requirement: Solo peeks-unlimited modal

**Reason**: Solo leftover mode removed.
**Migration**: Do not show unlimited modal when one eligible seat remains.

### Requirement: Distinct solo end copy for timer vs steps exhaustion

**Reason**: Solo timer / time-expired path removed.
**Migration**: Last player is cut by penultimate finish (`game/finish`) or finishes normally; no solo timer end modal.

### Requirement: Turn countdown ring on presence

**Reason**: Turn timer removed.
**Migration**: No turn ring; keep reconnect grace ring only when offline.
