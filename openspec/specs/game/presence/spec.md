# game/presence Specification

## Purpose

На экране Game показывает занятые места за столом кружками туристов: для seated — свой маркер в sticky нижней HUD-панели вместе с budgets и личной полосой, оппоненты в одном ряду сверху вплотную к доске; для spectator — все маркеры сверху без нижней presence-панели. Офлайн — круговой reconnect countdown; текущий ход в `playing` — countdown хода (синий / соло красный), при необходимости оба кольца сразу. Match status и leave — в общей шапке приложения. Значок места финиша, ready и say — угловые affordances на маркере (`game/finish`, `game/start`, `game/say`).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-01 | covered (client Game presence chrome) |
| SC-PRESENCE-02 | covered (client seated — opponents above board, own in bottom HUD) |
| SC-PRESENCE-03 | covered (client spectator — all markers above board) |
| SC-PRESENCE-04 | covered-by-reuse (client Game presence chrome — dual with turn) |
| SC-PRESENCE-05 | covered (client only occupied seats) |
| SC-PRESENCE-06 | modified — no numeric finish-place badge |
| SC-PRESENCE-07 | modified — no finish-place badge |
| SC-PRESENCE-08 | covered (client GamePage turn ring) |
| SC-PRESENCE-10 | modified — turn ring + reconnect dual when offline |
| SC-PRESENCE-11 | covered (client reserved outer chrome 96px around 72px avatar) |
| SC-PRESENCE-12 | covered (client sibling avatar img + rings) |
| SC-PRESENCE-13 | covered (client avatar size matches strip tourist) |
| SC-PRESENCE-14 | modified — no finish-place badge corners |
| SC-PRESENCE-22 | covered (client sticky bottom HUD — own+strip when seated; surface fill revised by SC-PRESENCE-34) |
| SC-PRESENCE-34 | covered (`AppHeaderChrome.test.ts` — sticky bottom HUD translucent underlay ~0.78 alpha) |
| SC-PRESENCE-23 | covered (client match-status slot) |
| SC-PRESENCE-24 | covered (client no room id in chrome) |
| SC-PRESENCE-26 | covered (client seated top presence min-height reserve) |
| SC-PRESENCE-30 | modified — focus @3 o’clock; no end-turn |
| SC-PRESENCE-31 | covered (client vitest — focuses nearest actionable) |
| SC-PRESENCE-32 | covered (client vitest — hidden when none actionable) |
| SC-PRESENCE-33 | modified — say @12 + focus @3; no end-turn |
| SC-PRESENCE-36 | modified — outcome toast top-center (Notify) |
| SC-PRESENCE-37 | modified — focus @3 stays fully visible |
| SC-PRESENCE-50 | pending — seat knowledge on avatar |
| SC-PRESENCE-51 | pending — +N knowledge grant animation |
| SC-PRESENCE-52 | modified — carried knowledge strip TR + Material lightbulb only |
| SC-PRESENCE-53 | pending — focus at former end-turn slot |
| SC-PRESENCE-54 | pending — turn countdown ring restored (60s) |
| SC-PRESENCE-55 | pending — no end-turn control |
| SC-PRESENCE-56 | pending — no finish-place badge in presence |
| SC-PRESENCE-57 | pending — all-jail skip toast for all clients |

Related: turn deadlines / knowledge / all-jail skip — `game/move`; peek modal — `game/board`; reconnect grace — `game/pieces`; bubbles — `game/say`; exit chrome — `game/leave`; board width — `game/board`; strip / selection — `game/pieces`; finish/cut outcome modal — `game/finish`.

## Requirements

### Requirement: Occupied seat presence on Game

While the user is on the Game screen in a tourist room, the system SHALL show a presence marker for every currently seated player (including seats held offline during reconnect grace). Empty seat slots MUST NOT be shown. Spectators MUST see the same set of occupied markers. Presence markers MUST use each seat’s tourist kind image.

#### Scenario [SC-PRESENCE-01]: Seated and spectators see occupied markers

- **GIVEN** a tourist room with two or more seated players
- **WHEN** a seated client and a spectator client view the Game screen
- **THEN** both see one presence marker per occupied seat
- **AND** each marker uses that seat’s tourist kind image

#### Scenario [SC-PRESENCE-05]: No empty seat placeholders

- **GIVEN** a tourist room with fewer than four seated players
- **WHEN** any client views the Game screen
- **THEN** only occupied seats have presence markers
- **AND** vacant places are not rendered as empty markers

### Requirement: Relative layout for seated players

For a seated viewer, presence markers of **other** occupied seats MUST appear in a row **above** the board, ordered left-to-right by earlier join time among those opponents. The viewer’s **own** presence marker MUST appear in the sticky bottom HUD panel under the board, together with that viewer’s private delivered-knowledge counter and the personal tourist strip when it exists (`game/pieces`). Opponent markers MUST NOT appear in the bottom HUD. Own marker MUST NOT appear above the board. Left/right columns beside the board MUST NOT be used for presence.

#### Scenario [SC-PRESENCE-02]: Seated viewer is always at home position

- **GIVEN** the user is seated and three other seated players exist, ordered by earlier join time among those others
- **WHEN** the Game presence layout is shown
- **THEN** the user’s marker is in the bottom HUD panel
- **AND** the three opponents appear above the board left-to-right in join order among those others
- **AND** no opponent marker is laid out in the bottom HUD
- **AND** no presence marker is laid out in a left or right column beside the board

### Requirement: Spectator presence layout

For a spectator viewer, all occupied seats MUST be laid out in a single row **above** the board, ordered left-to-right by join order among seated players. Missing seats simply omit markers. Spectators MUST NOT see an own-marker cluster, personal strip, or bottom presence row. Left/right columns beside the board MUST NOT be used for presence.

#### Scenario [SC-PRESENCE-03]: Spectator order top, bottom, left, right

- **GIVEN** a tourist room with four seated players in known join order and the user is a spectator
- **WHEN** the Game presence layout is shown
- **THEN** all four seated players’ markers appear above the board left-to-right in that join order
- **AND** no presence markers appear in a bottom HUD row for that spectator
- **AND** no own-marker cluster or personal tourist strip is shown for the spectator

### Requirement: Offline reconnect countdown on presence

While a seat is offline within reconnect grace, its presence marker MUST show a determinate circular progress reflecting remaining grace time (full at disconnect, empty at deadline) around the tourist image as the inner ring when a turn ring is also present, or as the sole active countdown ring when no turn deadline ring is active. While the seat is connected, the marker MUST NOT show that reconnect countdown as an active progress. Layout reservation for ring chrome remains per the reserved-marker requirement even when reconnect progress is inactive.

#### Scenario [SC-PRESENCE-04]: Offline seat shows circular countdown

- **GIVEN** a seated player’s seat is marked offline with a reconnect deadline in synced state
- **WHEN** any client views that seat’s presence marker on Game
- **THEN** a circular countdown progress is shown for the remaining grace time
- **AND** the tourist kind image remains visible in the marker
- **AND** when the seat becomes connected again, the reconnect countdown progress is not shown as active
- **AND** if that seat also holds an active turn deadline, the turn countdown remains visible together with the reconnect countdown

### Requirement: Tourist avatar always visible on occupied presence

Every occupied presence marker on the Game screen SHALL keep the seat’s tourist kind image visible in the marker at all times (connected or offline-in-grace, with or without active turn/reconnect rings). Rings MUST surround that image without replacing or omitting it. Implementations MUST NOT rely on placing the image only inside a Quasar `q-circular-progress` default slot without `show-value` (that slot is not rendered), and MUST NOT nest circular-progress components such that the image never appears in the DOM.

#### Scenario [SC-PRESENCE-12]: Occupied marker always shows tourist image

- **GIVEN** any occupied seated player’s presence marker on Game (with or without active turn deadline and/or reconnect grace countdown)
- **WHEN** any client views that marker
- **THEN** the tourist kind image for that seat is visible in the marker
- **AND** any active turn and/or reconnect countdown rings appear around that image without hiding it

### Requirement: Turn countdown ring on presence

While phase is `playing` and the synchronized turn deadline is active (`game/move`, 60s), presence markers for the **current** seat MUST show a determinate turn-budget countdown ring driven by `turnUntil` / `turnBudgetSeconds`. That ring MUST use the same blue accent previously used for the static current-turn outline. Reconnect grace countdown on offline seats remains as previously specified. When the current seat is offline within reconnect grace, dual turn+grace rings MAY apply. The static blue outline / box-shadow that previously marked the current-turn seat MUST NOT be used.

#### Scenario [SC-PRESENCE-08]: Current turn shows blue countdown instead of outline

- **GIVEN** a tourist room in phase `playing` with at least two non-finished seated players and a current-turn seat with a 60-second deadline
- **WHEN** any client views that seat’s presence marker
- **THEN** a blue circular countdown for remaining turn time is shown
- **AND** the tourist kind image remains visible in the marker
- **AND** no static blue outline-only turn indicator is shown on that marker

#### Scenario [SC-PRESENCE-54]: Current turn shows timer ring

- **GIVEN** it is a seated player’s turn in phase `playing` with an active 60-second deadline
- **WHEN** any client views that seat’s presence marker
- **THEN** a turn countdown ring is visible around the avatar

### Requirement: Dual turn and reconnect countdowns

When a seat is offline within reconnect grace, the presence marker MUST show the reconnect grace countdown ring. When that seat is also the current turn with an active turn deadline, dual turn+grace rings MAY apply. Spectators and seated viewers MUST see the same reconnect display when offline.

#### Scenario [SC-PRESENCE-10]: Offline current-turn seat shows both rings

- **GIVEN** a current-turn seat that is offline within reconnect grace with an active turn deadline
- **WHEN** any client views that seat’s presence marker
- **THEN** the reconnect grace countdown ring is visible
- **AND** the turn-deadline countdown ring is visible
- **AND** the tourist kind image remains visible in the marker

### Requirement: Reserved presence marker chrome size

Every occupied presence marker on the Game screen SHALL reserve stable space for the outer turn-ring chrome whether or not a turn or reconnect countdown is currently active, so that showing or hiding active countdown progress does not change the marker’s layout size or shift the board or neighboring markers.

#### Scenario [SC-PRESENCE-11]: Marker size stable when countdown appears

- **GIVEN** a connected seated player’s presence marker without an active reconnect countdown
- **WHEN** that seat becomes current turn and the turn countdown becomes active (or later becomes offline with reconnect countdown)
- **THEN** the marker’s reserved layout size does not jump
- **AND** the board layout does not reflow solely because the countdown appeared
- **AND** neighboring presence markers do not shift solely because the countdown appeared

### Requirement: Presence avatar matches strip tourist size

The tourist kind image inside every occupied presence marker SHALL use the same display size as one tourist image in the seated player’s personal strip on Game. Turn and reconnect rings MUST surround that image (outer ring larger than the avatar).

#### Scenario [SC-PRESENCE-13]: Presence avatar matches strip image size

- **GIVEN** the user is seated with a personal tourist strip visible and at least one occupied presence marker on Game
- **WHEN** any client compares the presence avatar image box to one strip tourist image box
- **THEN** those image boxes match in width and height
- **AND** any active countdown rings appear around the presence avatar without shrinking it below that size

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

#### Scenario [SC-PRESENCE-56]: Finished seat shows no place badge

- **GIVEN** a seated player has `finishPlace = 1` after full finish or penultimate cut
- **WHEN** any client views that seat’s presence marker
- **THEN** no numeric place badge is shown on that marker
- **AND** the finished seat’s own client uses the knowledge modal for outcome copy, not a place string

### Requirement: Own delivered knowledge beside the avatar

While the user is a seated player on the Game screen in phase `playing`, the system SHALL show that user’s **delivered knowledge** total (points transferred from finished tourists, from private `knowledge.delivered`) in a counter **to the right of** that user’s own presence avatar in the bottom HUD, with a **visible** knowledge icon using Quasar Material Icons name `lightbulb` (from already-loaded `material-icons` extras — MUST NOT use `mdi-*` names unless that font set is enabled). Spectators and other seats MUST NOT see another seat’s private delivered total — **own-seat only** (same privacy family as former budgets). Steps and peeks counters MUST NOT appear.

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

### Requirement: Carried knowledge on the strip only

While an unfinished own piece carries knowledge greater than 0 (from private `knowledge.pieces`), the personal **bottom strip** slot for that piece MUST show that carried amount in a **yellow-background** badge at the **top-right** of the strip tourist chrome, accompanied by a **visible** Material `lightbulb` icon (same constraint as avatar — no blank `mdi-*` glyph that leaves empty padding). The **board** piece chrome MUST NOT show a carried-knowledge badge. When knowledge is 0, the strip badge MUST NOT appear. When the piece finishes and knowledge transfers to the avatar, the strip badge MUST clear with the finish presentation. Opponents’ carried amounts MUST NOT be shown from another seat’s private payload.

#### Scenario [SC-PRESENCE-52]: Strip shows yellow knowledge badge top-right

- **GIVEN** an own unfinished piece has knowledge 2
- **WHEN** the owner views the board and strip
- **THEN** the strip slot shows a yellow-background badge at top-right with value 2 and a visible Material lightbulb icon
- **AND** the board piece for that tourist does not show a carried-knowledge badge
- **AND** the badge does not reserve empty space for a missing icon font

### Requirement: Sticky bottom game HUD panel

While the user is on the Game screen as a **seated** viewer, the bottom HUD panel that holds the own presence marker, delivered-knowledge counter beside the avatar, and personal strip (when present) MUST remain pinned to the bottom of the viewport so it stays visible while the board area above may scroll or resize. Opponent markers MUST use the row above the board, not this sticky panel — laid out tightly against the board (no large reserved empty band under the markers for say chrome). For a **spectator**, there MUST NOT be a bottom presence HUD row; markers stay above the board. The sticky bottom HUD surface MUST use a **translucent** fill so the shell / page artwork remains visible through it — product sense the same alpha as pack card-grid underlay (about three-quarters opacity light white / dark near-black). The panel MUST NOT use an opaque solid white / near-black (or equivalent fully opaque theme canvas) that fully hides the art behind the HUD.

#### Scenario [SC-PRESENCE-22]: Bottom HUD stays pinned while board scrolls

- **GIVEN** the seated user is on the Game screen with the bottom HUD panel visible
- **WHEN** the board content area above the panel is scrolled or the board height changes
- **THEN** the bottom HUD panel remains pinned to the bottom of the viewport
- **AND** the user’s own marker and strip remain inside that panel
- **AND** opponent markers remain above the board

#### Scenario [SC-PRESENCE-34]: Bottom HUD underlay is translucent

- **GIVEN** a seated user on the Game screen over the shell page background
- **WHEN** the sticky bottom seated HUD panel renders
- **THEN** the HUD background is translucent (artwork visible through it) at approximately three-quarters opacity matching SC-PACK-285 underlay alphas
- **AND** MUST NOT present as a fully opaque white/near-black bar that completely hides the page art behind the panel

### Requirement: Match status in shared application header

While the user is on the Game screen, the shared application header MUST show a concise match-status label centered in the header (product Russian senses such as «Ваш ход», «Ход соперника», «Ход игрока», «Ожидание соперника», connection/finished phrases as already used on Game). On Login and Lobby screens that match-status label MUST NOT appear. The status MUST derive from room phase / turn / connection state already mirrored for Game — not from a separate server channel. Peek outcomes MUST use the top-center Notify toast requirement; relying solely on this quiet match-status line for peek results is NOT sufficient.

#### Scenario [SC-PRESENCE-23]: Status centered in shared header on Game

- **GIVEN** the user is on the Game screen during their multiplayer turn
- **WHEN** the shared application header is shown
- **THEN** a centered status label conveys that it is the user’s turn
- **AND** on Lobby the same match-status label is not shown

### Requirement: Peek outcome toast top-center

When clients receive a peek-close outcome (correct, wrong submit, or turn-timeout), they MUST show a conspicuous **top-center toast** (Quasar Notify or equivalent fixed overlay) with the product Russian sense «Правильно!» / «Ошибка!» / «Не успел!» so the result is obvious. Relying solely on the quiet header match-status line is NOT sufficient.

#### Scenario [SC-PRESENCE-36]: Timeout shows Не успел toast

- **GIVEN** a task modal closes with timeout reason
- **WHEN** any client updates
- **THEN** a top-center toast «Не успел!» is shown

### Requirement: All-jail skip toast for every client

When the room broadcasts an all-jail turn-skip signal (`game/move` — seat cannot act, another eligible seat remains), **every** connected client MUST show a top-center Notify toast with product Russian sense «Все туристы в решётке — ход пропущен». The retired start-reset all-jail modal MUST NOT appear. When the party is over (knowledge modal path / fewer than two eligible seats), clients MUST NOT show this toast.

#### Scenario [SC-PRESENCE-57]: All-jail skip toast reaches the whole table

- **GIVEN** seat A’s turn is skipped because all of A’s unfinished pieces are trapped and another eligible seat remains
- **WHEN** the room broadcasts the skip signal
- **THEN** seat A, seat B, and a spectator each see the top-center toast «Все туристы в решётке — ход пропущен»
- **AND** no start-reset all-jail modal is shown

### Requirement: No room identifier in Game chrome

The Game screen and shared application header MUST NOT display the tourist room identifier (full or truncated) as visible chrome. Routing and reconnect MAY continue to use the room id without showing it to the user.

#### Scenario [SC-PRESENCE-24]: Room id not shown on Game

- **GIVEN** the user is on the Game screen of a tourist room
- **WHEN** the Game chrome and shared application header are shown
- **THEN** no visible room identifier (full or truncated) is presented as page chrome

### Requirement: Seated top presence row reserves height

For a **seated** viewer, the layout region above the board that holds opponent presence markers MUST reserve a stable vertical space even when no opponent markers are currently shown (for example the viewer is alone in the room). When an opponent later appears, the board MUST NOT jump downward solely because that reserved region appears for the first time. Empty seat **avatar placeholders** MUST NOT be introduced (SC-PRESENCE-05 remains). Spectators keep their existing top-marker layout without this alone-in-room reservation rule.

#### Scenario [SC-PRESENCE-26]: Alone seated viewer keeps top band reserved

- **GIVEN** the local user is seated alone in a tourist room on the Game screen
- **WHEN** a second player becomes seated and an opponent marker appears above the board
- **THEN** the board’s vertical position does not jump by approximately one presence-row height solely due to the top row appearing
- **AND** no empty seat avatar placeholders are shown

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

#### Scenario [SC-PRESENCE-53]: Focus sits at three o’clock

- **GIVEN** it is the user’s turn with at least one actionable own piece and no task modal open
- **WHEN** the user views Game chrome
- **THEN** the focus control is shown at 3 o’clock on the own avatar
- **AND** no end-turn control is shown

#### Scenario [SC-PRESENCE-55]: End-turn control never shown

- **GIVEN** it is the user’s turn in a multiplayer `playing` room
- **WHEN** the user views Game chrome
- **THEN** the end-turn control is not shown

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
