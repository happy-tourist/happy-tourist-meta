# game/finish Specification

## Purpose

Финиш партии в room `tourist`: турист, вошедший в центр, сходит с поля; игрок, проводивший всех четырёх, получает место, остаётся за столом без ходов (say/strip/presence) и освобождает seat только при выходе. Связано с `game/move`, `game/pieces`, `game/leave`, `game/presence`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-01 | covered (server mocha + client UX — enter-center travel ~550 ms then longer fade; no drop without travel) |
| SC-FINISH-02 | covered (server mocha) |
| SC-FINISH-03 | covered (modified — knowledge modal not place) |
| SC-FINISH-04 | covered (modified — later finisher knowledge modal not place 2) |
| SC-FINISH-05 | covered-by-reuse — room stays open |
| SC-FINISH-06 | covered-by-reuse (server mocha + client) |
| SC-FINISH-07 | covered (server mocha — no seat after leave in playing) |
| SC-FINISH-08 | covered-by-reuse (server mocha) |
| SC-FINISH-09 | covered (modified — strip dimming without steps gate) |
| SC-FINISH-10 | covered (modified — strip dimming without steps gate) |
| SC-FINISH-11 | covered (server mocha) |
| SC-FINISH-12 | covered (server mocha — return clears finished) |
| SC-FINISH-13 | covered (modified — return affordance without steps; 36×36; no confirm) |
| SC-FINISH-14 | covered (server mocha — strip selectable after return) |
| SC-FINISH-15 | covered (client UX — return travel from nearest center) |
| SC-FINISH-16 | covered (client UX — finished strip slot click does not start return) |
| SC-FINISH-17 | covered (client UX — same longer finish presentation as move→center) |
| SC-FINISH-18 | covered (client UX — same longer finish presentation) |
| SC-FINISH-19 | covered (server mocha + client UX — catapult→center: longer travel after vanish; no race-drop) |
| SC-FINISH-20 | covered (client UX — finish travel even when finish sync late; longer timing) |
| SC-FINISH-21 | covered (modified — full finish releases turn; knowledge modal) |
| SC-FINISH-30 | pending — piece finish transfers knowledge |
| SC-FINISH-31 | modified — full finish modal copy + delivered |
| SC-FINISH-32 | modified — penultimate cut burns last; cut copy |
| SC-FINISH-33 | pending — single-player no penultimate cut |
| SC-FINISH-34 | modified — one chrome; distinct finish vs cut i18n |
| SC-FINISH-35 | pending — finish travel anim click + catapult |

Related: seating gate — `game/pieces` (new seats only in `waiting`); return geometry and board action — `game/move`; push finish — `game/move` SC-MOVE-76; catapult fling onto center — `game/move` / `game/board`; knowledge / presence — `game/presence`.

## Requirements

### Requirement: Entering any center cell finishes a piece

When the room start phase is `playing` and the server accepts a legal **move or push** whose landing target is any of the four center cells of the tourist layout, the server SHALL mark that piece finished. A finished piece MUST NOT occupy any board cell for subsequent move validation (the center cell becomes free for other pieces immediately). Every client that displays the board MUST remove that piece from the board after travel onto the landing cell and a disappear animation on that cell (no travel toward the strip) — for finish caused by a **move** and by a **push**, with the same presentation. The travel onto the center cell MUST be clearly visible for about **550 ms** (product default), and the subsequent disappear fade MUST last about **300 ms** after that travel. The piece MUST NOT vanish from its pre-finish board cell without that travel. Finished piece state MUST be synchronized to all clients in the room.

#### Scenario [SC-FINISH-01]: Move onto a center cell finishes the piece

- **GIVEN** the room phase is `playing`, it is a seated non-finished player’s turn, and one of that player’s unfinished pieces has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell
- **THEN** that piece is marked finished in synced state
- **AND** that center cell is not occupied by that piece for later moves
- **AND** every client observes the piece travel onto the center cell for about 550 ms then leave the board after about a 300 ms disappear animation

#### Scenario [SC-FINISH-02]: Another piece may reuse the same center cell

- **GIVEN** a piece has just finished on a specific center cell and no other unfinished piece occupies that cell
- **WHEN** another unfinished piece legally moves onto that same center cell on a later turn
- **THEN** the server accepts the move and finishes that second piece
- **AND** the first finished piece remains finished off the board

#### Scenario [SC-FINISH-17]: Push onto a center cell finishes with the same presentation

- **GIVEN** a legal push whose far-side cell is a free center cell
- **WHEN** the push is accepted and the target finishes
- **THEN** every client that displays the board observes the target travel onto that center cell then leave after the same ~550 ms travel / ~300 ms disappear presentation as a move finish
- **AND** the target does not vanish from its pre-push cell without that travel

#### Scenario [SC-FINISH-18]: Own move onto center travels from the pre-move cell

- **GIVEN** it is a seated player’s turn and an own unfinished piece has a free neighbor center cell
- **WHEN** that player submits a move onto that center cell and the server accepts
- **THEN** clients that display the board show that piece traveling from its pre-move cell onto the center cell
- **AND** then leaving after the same disappear presentation used for push→center
- **AND** the piece does not vanish from its pre-move cell without that travel

### Requirement: Center finish keeps travel animation

When an unfinished piece becomes finished by landing on a center cell (direct move click or catapult fling chain), the finishing client MUST present the existing finish travel (~550 ms) then fade (~300 ms) so the piece does not snap-vanish from its pre-finish cell — including the **first** finisher of a seat. Mid-join already-finished pieces MAY omit travel. Catapult→center MUST defer vanish until after catapult presentation settles, then run the same travel path. Tests MUST cover a behavioral path that fails if travel `from` is missing when finishing from a non-center cell (SFC source-only asserts are not sufficient alone).

#### Scenario [SC-FINISH-35]: Click and catapult finish animate to center

- **GIVEN** an own unfinished piece finishes onto center by direct move or by catapult fling (including the seat’s first finished tourist)
- **WHEN** the local client paints the finish
- **THEN** the piece slides/travels toward center before fading
- **AND** it does not disappear from the origin cell without travel

### Requirement: Finishing a piece transfers its knowledge to the seat

When an unfinished piece becomes finished (move, return path, or catapult onto center), the server MUST add that piece’s carried knowledge to the seat’s **delivered knowledge** total, then clear the piece’s carried knowledge to 0. The finishing client MUST play the same family of +N fall animation into the avatar knowledge counter as former budget grants (~2 s). Undelivered knowledge on other unfinished pieces MUST remain on those pieces.

#### Scenario [SC-FINISH-30]: Center finish transfers piece knowledge

- **GIVEN** a seated player’s unfinished piece has knowledge 3 and legally finishes onto a center cell
- **WHEN** that piece is marked finished
- **THEN** the seat’s delivered knowledge increases by 3
- **AND** that piece’s carried knowledge is 0
- **AND** the finishing client plays the +N knowledge animation into the avatar counter

### Requirement: Completing all four pieces assigns finish place

When a seated player’s last unfinished piece becomes finished (all pieces for that seat finished — count equals `touristsPerPlayer`, not always four), the server SHALL mark that seat finished (`finishPlace = 1`) and ineligible for turns while other eligible seats remain. The server MUST NOT present a monotonic finish **place** number in the congratulation UX. The client of that seat MUST show the delivered-knowledge modal per the full-finish modal requirement. Closing the modal MUST leave the player in the room as a finished seat. The room MUST NOT disconnect or dispose solely because one or all seats became finished. When that last-piece finish happens via a catapult fling onto a center cell under paced trap rules, the same finished marking and knowledge modal MUST apply; after the trap pipeline is idle, turn eligibility MUST follow `game/move` (finished seat MUST NOT keep the current turn while another eligible seat remains). If this full finish leaves exactly one eligible seat and `maxSeats ≥ 2`, the penultimate-cut requirement MUST run.

#### Scenario [SC-FINISH-03]: First full finisher gets place one

- **GIVEN** a tourist room in phase `playing` where no seat is finished yet, and a seated player has exactly one unfinished piece adjacent to a free center cell with positive delivered knowledge already accrued
- **WHEN** that player’s move finishes the last piece
- **THEN** that seat is marked finished (internal finished gate; UI does not congratulate place `1`)
- **AND** that player’s client shows the delivered-knowledge modal
- **AND** the seat remains in the room after the modal is closed

#### Scenario [SC-FINISH-04]: Later full finisher gets the next place

- **GIVEN** a tourist room where one seat is already fully finished and another seated player finishes their last piece
- **WHEN** that last piece becomes finished
- **THEN** that second seat is marked finished
- **AND** that player’s client shows the delivered-knowledge modal (not place `2`)

#### Scenario [SC-FINISH-05]: Room stays open when all seats are finished

- **GIVEN** every seated player in the room has finished (or been cut)
- **WHEN** no further consented leave or grace timeout has removed seats
- **THEN** the room remains available
- **AND** seated clients are not forcibly disconnected solely because all seats are finished

#### Scenario [SC-FINISH-21]: Catapult fourth-piece finish assigns place and releases turn

- **GIVEN** a tourist room in phase `playing` with at least two eligible seated players
- **AND** seated player A has exactly one unfinished free piece whose paced catapult fling lands on a free center cell
- **WHEN** that catapult relocate finishes A’s last piece and the presentation pipeline then becomes idle
- **THEN** A’s client shows the delivered-knowledge modal
- **AND** after the trap pipeline is idle the synchronized current turn is not A while another eligible seat remains

### Requirement: Full finish modal shows delivered knowledge

When a seated player’s last unfinished piece becomes finished (all of that seat’s pieces finished), the server MUST mark the seat finished (ineligible for turns) by setting `finishPlace = 1` (finished-seat skip gate; not a ranking place) and re-send private `knowledge` with `outcome: 'finish'`. The client of that seat MUST show the **knowledge modal**: product Russian sense — all of that player’s tourists reached finish and earned `{n}` delivered knowledge points. The modal MUST NOT congratulate a numeric finish place (1st/2nd/…). Closing the modal MUST leave the player in the room as a finished seat. The room MUST NOT disconnect solely because seats finished. Presence MUST NOT show a finish-place badge (`game/presence`).

#### Scenario [SC-FINISH-31]: Full finish shows knowledge total modal

- **GIVEN** a seated player has exactly one unfinished piece left, delivered knowledge already 4, and that last piece carries 2 knowledge and finishes onto center
- **WHEN** that last piece becomes finished
- **THEN** the seat’s delivered knowledge is 6
- **AND** that player’s client shows a modal with full-finish wording and 6 knowledge points
- **AND** the modal does not present a finish place number
- **AND** the seat remains in the room after the modal is closed

### Requirement: One knowledge-modal chrome; distinct finish vs cut copy

Full finish and penultimate cut MUST share **one** client modal chrome bound to private `knowledge.delivered` and `knowledge.outcome` (`finish` | `cut`). Distinct visual layouts are NOT required, but i18n copy MUST differ: full finish communicates that all of the player’s tourists reached finish and earned `{n}`; cut MUST use product Russian sense «Соперники финишировали. Ваши туристы, **которые** успели, доставили {n} знаний» (MUST NOT use «что успели»). The client MUST open that modal when the **local** seat’s synchronized `finishPlace` transitions 0→1, reading the latest private knowledge payload. The client MUST retire place congratulation copy (`finishPlaceModal` / «Вы N-й!») and MUST NOT render numeric `finishPlace` badges on presence. When the party ends this way, clients MUST NOT show an all-jail skip toast for that outcome (`game/move`).

#### Scenario [SC-FINISH-34]: Cut and full finish use delivered-only modal with distinct copy

- **GIVEN** a seat reaches full finish or is cut by the penultimate rule
- **WHEN** that seat’s client shows the outcome modal
- **THEN** the modal body is driven by that seat’s `knowledge.delivered` and `outcome`
- **AND** full-finish wording is used when `outcome` is `finish`
- **AND** cut wording uses «которые успели» when `outcome` is `cut`
- **AND** no finish-place number appears in the modal or on that seat’s presence badge
- **AND** no all-jail skip toast is shown solely because seats finished

### Requirement: Penultimate full finish ends the last player

When `maxSeats ≥ 2` and a seated player becomes fully finished while exactly **one** other seated player remains eligible (non-finished), the server MUST immediately end that last eligible seat’s run: set that last seat `finishPlace = 1` (cut), discard (burn) all knowledge still carried on that last seat’s unfinished pieces to 0 without adding it to delivered knowledge, set those pieces `finished = true` so they cannot move, and re-send private `knowledge` with `outcome: 'cut'`. Every affected client for the cut seat MUST see the knowledge-modal chrome with **cut** wording reporting that seat’s **delivered** knowledge only (burned points excluded). Solo leftover permanent time-expired MUST NOT apply. When `maxSeats === 1`, this penultimate cut MUST NOT run — the sole seat plays until its own full finish.

#### Scenario [SC-FINISH-32]: Penultimate finisher cuts the last seat

- **GIVEN** a tourist room with maxSeats 2, seat A has just finished all pieces, and seat B still has unfinished pieces carrying knowledge 5 with delivered knowledge 1
- **WHEN** A’s full finish is applied
- **THEN** B is marked finished/cut and cannot move
- **AND** B’s delivered knowledge remains 1
- **AND** the 5 carried knowledge is not added to B’s delivered total
- **AND** B’s client shows a modal with cut wording reporting 1 knowledge point

#### Scenario [SC-FINISH-33]: Single-player is not cut by penultimate rule

- **GIVEN** a tourist room with maxSeats 1 and one seated player who finishes all pieces
- **WHEN** that full finish is applied
- **THEN** that player sees the knowledge total modal for their delivered knowledge
- **AND** no other seat is cut
- **AND** the penultimate-cut path does not run

### Requirement: Finished seat keeps seat capacity and limited agency

A seat that has a finish place SHALL remain a seated player: it MUST keep its tourist kind, personal strip, presence marker, and eligibility to send whitelist say intents per `game/say`. That seat MUST NOT successfully submit board moves. The seat MUST continue to count toward `maxSeats` occupancy until permanently removed by consented leave or reconnect grace timeout. While phase is `countdown` or `playing`, a joining client MUST NOT receive a new seat even when `seats.size < maxSeats` after counting finished seats (`game/pieces`). Spectators remain connections without seats and MUST NOT keep the room alive: when seated count reaches zero the room is disposed even if spectators remain (unchanged dispose rule).

#### Scenario [SC-FINISH-06]: Finished player keeps strip and may say

- **GIVEN** a seated player who has finish place `1` and is connected
- **WHEN** that player views the Game screen and submits a whitelist say intent
- **THEN** the personal four-slot strip remains visible for that player
- **AND** the say is accepted and broadcast per `game/say`
- **AND** a board move from that seat is rejected without changing piece positions

#### Scenario [SC-FINISH-07]: Finished seat blocks mid-game seating until leave

- **GIVEN** a tourist room in phase `playing` with `maxSeats` equal to 2 and both seats finished (each has a finish place)
- **WHEN** another authenticated client joins the room
- **THEN** that client receives no seat and no pieces (spectator)
- **AND** after one finished seat permanently leaves, a subsequent joiner still receives no seat and no pieces while phase remains `playing`

#### Scenario [SC-FINISH-08]: Dispose only when no seats remain

- **GIVEN** a tourist room whose only remaining seats are finished seats, plus zero or more spectators
- **WHEN** every seated player permanently leaves until seated count is zero
- **THEN** the room is disposed
- **AND** spectators alone MUST NOT prevent disposal

### Requirement: Finished strip slots and place chrome

For a seated client, each personal strip slot whose matching piece is finished MUST show a finish indicator at the top-right of that slot and MUST NOT be usable to select or submit a **move**. When return-from-finish is **not** currently available for that side (not the user’s interactive turn, board-action already spent / local spent flag set, `finishPlace ≠ 0`, or no legal free non-hole ring cell), that finished slot MUST be visually dimmed. When return **is** available for that side under the one-board-action economy (`game/move`), the slot MUST NOT be dimmed solely because the piece is finished. After the seat has `finishPlace > 0`, all strip slots MUST remain visible with finish indicators (typically dimmed because return is unavailable). Numeric finish-place chrome on the strip is NOT required.

#### Scenario [SC-FINISH-09]: Finished strip slot is inactive with indicator

- **GIVEN** a seated player has finished exactly one of their pieces and return is not available for that side
- **WHEN** that player views the personal strip
- **THEN** the finished piece’s strip slot shows a finish indicator at the top-right
- **AND** that slot is visually dimmed
- **AND** activating that finished slot does not select a piece, open a return modal, or submit a move

#### Scenario [SC-FINISH-10]: All strip slots stay after full finish

- **GIVEN** a seated player has `finishPlace > 0` and all of that seat’s pieces finished
- **WHEN** that player views the personal strip
- **THEN** all strip slots remain visible with finish indicators

### Requirement: Reconnect grace for finished seats

An unexpected disconnect of a finished seat MUST hold the seat for the same reconnect grace as an active seat (`game/pieces` / `game/presence`). Turn handling for finished seats follows `game/move` (finished seats are skipped and MUST NOT hold the current turn while any non-finished seated player remains).

#### Scenario [SC-FINISH-11]: Finished disconnect keeps seat for grace

- **GIVEN** a finished seated player disconnects unexpectedly
- **WHEN** the reconnect grace has not yet expired
- **THEN** the seat remains occupied with offline grace state
- **AND** the tourist kind and finish place remain assigned to that seat until reconnect or timeout

### Requirement: Return from finish puts the piece back in play

When the server accepts a legal return-from-finish for a seat with finish place **0**, the targeted piece MUST become unfinished and MUST occupy the chosen ring cell. That piece MUST again count toward board occupancy and MUST be eligible for later moves under `game/move`. The strip slot for that side MUST no longer show a finished-only lock for selection once the piece is unfinished (subject to turn rules).

#### Scenario [SC-FINISH-12]: Returned piece is unfinished on the board

- **GIVEN** a seated player with finish place 0 has a finished piece and successfully returns it onto a legal ring cell
- **WHEN** synced state updates
- **THEN** that piece’s finished flag is false
- **AND** the piece occupies the chosen cell
- **AND** a later legal move of that piece MAY be accepted on a subsequent action of that turn or a later turn

#### Scenario [SC-FINISH-14]: Returned strip slot is usable again on turn

- **GIVEN** the same seat as SC-FINISH-12 still has the current turn after return
- **WHEN** that player activates the strip slot for the returned side
- **THEN** the slot is treated as an unfinished piece slot for selection
- **AND** it does not remain locked as a finished-only slot

### Requirement: Return affordance beside finished strip indicator

For a seated client on their own turn with `finishPlace` 0 and board action not yet spent, each finished side that has at least one legal ring cell MUST show a return affordance centered above that tourist in the personal strip (same visual family as board rescue affordances; no push). The affordance MUST use the **illustrated board-action return icon** and MUST NOT paint a solid colored circular chrome behind the icon. Glyph and hit MUST be **36×36 CSS pixels**. The return hit MUST sit entirely **above** that strip tourist figure. Activating that affordance MUST enter ring-target selection on the board without a confirmation dialog. Choosing a legal ring cell MUST submit the return, which spends the one board action (`game/move`). When return is not available, the finished slot MUST remain non-actionable and MUST NOT show the return affordance. Legal return target cells MUST use the **same** destination highlight presentation as ordinary legal move targets (`game/move`).

#### Scenario [SC-FINISH-13]: Return control appears next to the flag when steps remain

- **GIVEN** it is the user’s turn with `finishPlace` 0, board action not spent, a finished side, and at least one legal free non-hole ring cell
- **WHEN** the user views that finished strip slot
- **THEN** a return affordance with the illustrated return icon is shown centered above that tourist (same visual family as board rescue; no solid colored circular chrome)
- **AND** the return glyph / hit are the 36×36 glyph-fills-hit size family
- **AND** the return hit does not cover that strip tourist figure
- **AND** the finished-flag indicator size is unchanged by this scale
- **AND** activating the affordance highlights legal ring cells like ordinary move targets without a confirmation dialog
- **AND** choosing a highlighted ring cell submits the return
- **AND** no confirmation dialog is shown

#### Scenario [SC-FINISH-16]: Finished strip slot click alone does not start return

- **GIVEN** the same setup as SC-FINISH-13
- **WHEN** the user activates the finished strip slot body without using the return affordance
- **THEN** no confirmation dialog appears
- **AND** return-mode is not entered solely from that slot-body activation

### Requirement: Return-from-finish travel animation

When the server accepts a legal return-from-finish, every client that displays the board MUST animate that piece traveling from the **nearest** of the four center finish cells to the chosen ring cell (Chebyshev distance to the ring cell; ties broken by lower row, then lower column), using the same travel timing family as ordinary piece moves. The animation MUST be visible to all clients in the room. Input locking for the submitting client during travel follows the same rules as move travel (`game/move`).

#### Scenario [SC-FINISH-15]: Everyone sees return leave the finish block

- **GIVEN** a seated player successfully returns a finished piece onto a legal ring cell
- **WHEN** clients update the board
- **THEN** every client shows that piece animating from the nearest center cell to that ring cell
- **AND** after the animation the piece occupies the ring cell as an unfinished piece

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules (after that hop’s presentation budget), the server SHALL finish that piece with the same authority and presentation expectations as an ordinary move or push onto that center cell (`game/finish` enter-center rules). Clients MUST run finish travel and disappear **after** the triggering catapult’s land arrival and overlay have fully completed (`game/board` successful presentation: land → overlay → then finish travel), not in parallel with that overlay. Finish travel MUST use the same about **550 ms** slide and about **300 ms** disappear family as move/push onto center. Under server-paced sync the piece MAY become `finished` only when the hop’s relocate fires — **after** reveal enqueue — and clients MUST still present finish travel from the catapult cell after that hop’s vanish (they MUST NOT drop the piece from the board without travel solely because `finished` was not known at reveal time, and MUST NOT drop it without travel when `finished` arrives early). In a multi-hop catapult chain ending on center, every hop MUST animate; finish travel MUST start only after the last successful catapult vanish.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** that catapult’s arrival and overlay have fully completed and the deferred fling / finish presentation runs
- **THEN** the piece is finished with the same authority as a move onto that center
- **AND** clients present finish travel (~550 ms) and disappear (~300 ms) consistent with move/push onto center, starting only after the catapult vanish
- **AND** the piece does not disappear from the catapult cell without that travel

#### Scenario [SC-FINISH-20]: Finish travel even when finish sync arrives after reveal

- **GIVEN** a successful catapult reveal is presenting and the authoritative finish onto center is applied only after the overlay presentation budget (paced fire)
- **WHEN** the overlay vanish completes
- **THEN** every client MUST animate finish travel from the catapult cell to center and then disappear
- **AND** MUST NOT leave the piece absent from the board without that travel solely because `finished` was false at reveal enqueue time
- **AND** in a catapult→…→center chain, prior hops complete their overlay and fling travel before the final hop’s finish travel
