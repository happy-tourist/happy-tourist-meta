## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-17 | modified (end-turn ~2×; center of hit on avatar circumference at 3 o’clock) |
| SC-PRESENCE-23 | modified (match-status slot can show peek outcome flash for ~10s) |
| SC-PRESENCE-25 | modified (end-turn not in budgets; focus may sit under peeks) |
| SC-PRESENCE-30 | modified (focus ~2× under peeks budget; not on marker 9 o’clock) |
| SC-PRESENCE-33 | modified (say above focus; say @12 + end-turn @3; focus under peeks; no overlap) |
| SC-PRESENCE-36 | added (peek outcome flash texts/colors; all clients; 10s then restore) |
| SC-PRESENCE-37 | added (focus under peeks stays fully visible / activatable in HUD) |

## MODIFIED Requirements

### Requirement: End-turn control next to own avatar

While it is the seated user’s multiplayer turn in phase `playing` (two or more eligible seats) and the user is not time-expired, the Game presence chrome MUST show an **icon-only** end-turn control whose **hit-center sits on the circumference** of that user’s own presence marker at the **right-center** edge (product clock sense: **3 o’clock**). The control MUST use the **illustrated board-action end-turn icon** and MUST NOT paint a solid colored circular chrome behind the icon. The illustrated glyph MUST render at about **twice** the prior presence board-action glyph size, and the hit target MUST be enlarged in the same family (~56–64 CSS pixels) while remaining non-overlapping with say when say is visible. Activating it MUST submit end-turn per `game/move` **immediately** — no confirmation dialog. The control MUST NOT show a visible text label (tooltips later are out of scope). The control MUST NOT appear in the budgets stack beside the avatar. The control MUST NOT appear as a labeled dock above the sticky bottom HUD. The control MUST NOT appear for spectators, for seats that are not current turn, during solo play, or for finished / time-expired seats. While a peek modal is open, the control MUST NOT be activatable (board-busy / peek lock as already required).

#### Scenario [SC-PRESENCE-17]: Current multiplayer seat sees «Завершить ход»

- **GIVEN** it is the user’s turn in a multiplayer `playing` room and no peek modal is open
- **WHEN** the user views Game chrome
- **THEN** an icon-only end-turn control with the illustrated end-turn icon is shown with its hit-center on the own avatar circumference at 3 o’clock
- **AND** the glyph / hit use the enlarged (~2×) presence family
- **AND** the control has no solid colored circular chrome behind the icon
- **AND** activating it submits end-turn without a confirmation dialog

#### Scenario [SC-PRESENCE-18]: Solo hides end-turn

- **GIVEN** the user is the sole eligible seated player in solo play
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown

#### Scenario [SC-PRESENCE-25]: End-turn not inline with budgets

- **GIVEN** it is the local seated user’s turn in multi-seat play and end-turn is available
- **WHEN** the bottom HUD budgets stack beside the own avatar is shown
- **THEN** steps and peeks appear in that stack without the end-turn control inline
- **AND** the end-turn control remains on the right edge of the avatar (not a separate above-panel text dock)
- **AND** when focus is available it may appear under the peeks counter in that stack (not on the marker’s left edge)

### Requirement: Match status in shared application header

While the user is on the Game screen, the shared application header MUST show a concise match-status label centered in the header (product Russian senses such as «Ваш ход», «Ход соперника», «Ход игрока», «Ожидание соперника», connection/finished phrases as already used on Game). On Login and Lobby screens that match-status label MUST NOT appear. The status MUST derive from room phase / turn / connection state already mirrored for Game — not from a separate server channel — **except** while a peek-outcome flash is active (see peek outcome flash requirement), during which the same centered header slot MUST show the flash text instead of the ordinary match-status phrase. After the flash duration ends, the ordinary match-status MUST return.

#### Scenario [SC-PRESENCE-23]: Status centered in shared header on Game

- **GIVEN** the user is on the Game screen during their multiplayer turn and no peek-outcome flash is active
- **WHEN** the shared application header is shown
- **THEN** a centered status label conveys that it is the user’s turn
- **AND** on Lobby the same match-status label is not shown

### Requirement: Focus actionable tourist control

While the local user is a seated current-turn player in phase `playing` and at least one of their unfinished free pieces can currently take an action (legal move with steps remaining, fresh peek, flipped peek, rescue, push, or return-from-finish when applicable), the Game presence chrome MUST show an **icon-only** focus control placed in the **own budgets stack immediately under the peeks counter** (the peeks row with the visibility glyph and peeks count). The control MUST use the **illustrated board-action focus icon** and MUST NOT paint a solid colored circular chrome behind the icon. The illustrated glyph MUST render at about **twice** the prior presence board-action glyph size, and the hit target MUST be enlarged in the same family (~56–64 CSS pixels). The focus control MUST NOT sit on the presence-marker circumference at 9 o’clock (that left edge MUST remain empty of focus). Activating it MUST select / focus the **nearest** (or otherwise first) own unfinished free piece that has at least one such available action, without requiring a confirm dialog. When no own piece has an available action, the control MUST be hidden or disabled. Spectators and non-current seats MUST NOT see an enabled control for themselves. Say remains on the **top-center** circumference (~**12 o’clock**); end-turn remains on the **right-center** circumference (~**3 o’clock**) per their requirements.

#### Scenario [SC-PRESENCE-30]: Focus control sits under peeks (say@12 / end-turn@3)

- **GIVEN** it is the local seated user’s multiplayer turn with at least one actionable own tourist
- **WHEN** the own presence chrome is shown
- **THEN** a focus control with the illustrated focus icon is present immediately under the peeks counter in the own budgets stack
- **AND** the glyph / hit use the enlarged (~2×) presence family
- **AND** no focus control is on the avatar circumference at 9 o’clock
- **AND** say (when visible) is at top-center (12) and end-turn (when visible) at right-center (3) on the marker circumference
- **AND** the three controls remain non-overlapping (say and end-turn on the marker; focus under peeks)

#### Scenario [SC-PRESENCE-31]: Focus selects nearest actionable tourist

- **GIVEN** it is the local user’s turn with two own unfinished free pieces where only piece B can peek or move and piece A cannot
- **WHEN** the user activates the focus control
- **THEN** piece B becomes selected / focused for board actions
- **AND** piece A is not selected

#### Scenario [SC-PRESENCE-32]: Focus hidden when nothing actionable

- **GIVEN** it is the local user’s turn and no own unfinished free piece has a legal move, peek, rescue, push, or return
- **WHEN** the own presence chrome is shown
- **THEN** the focus control is not available as an enabled action

### Requirement: Say affordance sits above the focus control

On the local seated user’s own presence avatar, when say and/or end-turn are visible, their **hit-centers MUST sit on the marker circumference** at distinct clock positions — say at **12** (top-center), end-turn at **3** (right-center) — so their hit areas do not overlap. Focus, when visible, MUST sit under the peeks counter in the budgets stack (not on the marker at 9 o’clock) and MUST NOT overlap say or end-turn hits. Say remains on the avatar above the budgets stack where focus lives. This does not change say whitelist or `game/say` message rules, nor end-turn submission rules. Circumference hits use the enlarged presence family (~56–64 CSS pixels).

#### Scenario [SC-PRESENCE-33]: Say is above focus without overlap

- **GIVEN** it is the local seated user’s multiplayer turn with say available, at least one actionable own tourist (focus visible), and end-turn available
- **WHEN** the own presence chrome is shown
- **THEN** the say affordance hit-center is at top-center (12) and end-turn at right-center (3) on the avatar circumference
- **AND** the focus control is under the peeks counter in the budgets stack
- **AND** the three hit areas do not overlap

## ADDED Requirements

### Requirement: Peek outcome flash in match-status slot

When any client in the room receives a peek-close that reports a **correct** resolve, **every** client on the Game screen MUST replace the centered match-status label with the product Russian text «Правильно!» in a green emphasis for about **ten seconds**, then restore the ordinary match-status. When the peek-close reports an **incorrect wrong submit**, every such client MUST show «Ошибка!» in a red emphasis for about ten seconds, then restore. When the peek-close reports an **incorrect turn-timeout** while a peek was open, every such client MUST show «Не успел!» in a yellow emphasis for about ten seconds, then restore. The flash MUST use the same centered header slot as ordinary match-status (it replaces that text for the duration — not a second permanent line). Spectators and seated players alike MUST see the same flash. Leave mid-peek and other non-timeout force-closes MUST NOT show «Не успел!».

#### Scenario [SC-PRESENCE-36]: All clients see peek outcome flash then restore status

- **GIVEN** two clients are on the Game screen in the same room with an open peek
- **WHEN** the peek closes as correct
- **THEN** both clients show «Правильно!» in green in the centered match-status slot for about ten seconds
- **AND** afterward each client again shows the ordinary match-status for the current room/turn state
- **AND** when a peek instead closes as a wrong submit, both clients show «Ошибка!» in red for about ten seconds
- **AND** when a peek instead closes because the turn timed out with the modal open, both clients show «Не успел!» in yellow for about ten seconds

### Requirement: Focus under peeks stays fully visible

While the focus control is visible under the peeks counter in the sticky bottom HUD, every client MUST keep the entire focus hit target inside the visible, pointer-reachable chrome of that HUD (not clipped by the panel or its horizontal scrollport).

#### Scenario [SC-PRESENCE-37]: Focus hit under peeks stays fully visible

- **GIVEN** it is the local seated user’s turn with the focus control visible under the peeks counter
- **WHEN** the user views the own presence chrome
- **THEN** the entire focus hit is visible without being clipped by the HUD panel or its horizontal scrollport
- **AND** the user can activate the focus control
