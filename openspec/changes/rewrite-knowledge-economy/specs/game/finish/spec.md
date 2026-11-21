# Delta: game/finish

Knowledge transfer на финише туриста; модалка знаний вместо места; penultimate cut последнего.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-03 | modified — knowledge modal not place |
| SC-FINISH-04 | modified — later finisher knowledge modal (not place 2) |
| SC-FINISH-05 | covered-by-reuse — room stays open |
| SC-FINISH-09…10 | modified — strip dimming without steps gate |
| SC-FINISH-13 | modified — return affordance without steps |
| SC-FINISH-21 | modified — full finish releases turn; knowledge modal |
| SC-FINISH-30 | pending — piece finish transfers knowledge |
| SC-FINISH-31 | pending — full finish modal shows delivered knowledge |
| SC-FINISH-32 | pending — penultimate cut burns last undelivered |
| SC-FINISH-33 | pending — single-player no penultimate cut |
| SC-FINISH-34 | pending — one knowledge-modal chrome; no place badge |

## ADDED Requirements

### Requirement: Finishing a piece transfers its knowledge to the seat

When an unfinished piece becomes finished (move, return path, or catapult onto center), the server MUST add that piece’s carried knowledge to the seat’s **delivered knowledge** total, then clear the piece’s carried knowledge to 0. The finishing client MUST play the same family of +N fall animation into the avatar knowledge counter as former budget grants (~2 s). Undelivered knowledge on other unfinished pieces MUST remain on those pieces.

#### Scenario [SC-FINISH-30]: Center finish transfers piece knowledge

- **GIVEN** a seated player’s unfinished piece has knowledge 3 and legally finishes onto a center cell
- **WHEN** that piece is marked finished
- **THEN** the seat’s delivered knowledge increases by 3
- **AND** that piece’s carried knowledge is 0
- **AND** the finishing client plays the +N knowledge animation into the avatar counter

### Requirement: Full finish modal shows delivered knowledge

When a seated player’s last unfinished piece becomes finished (all of that seat’s pieces finished), the server MUST mark the seat finished (ineligible for turns) by setting `finishPlace = 1` (finished-seat skip gate; not a ranking place). The client of that seat MUST show the **knowledge modal** (same chrome as penultimate cut): product Russian sense reports **how many knowledge points** that player delivered (`knowledge.delivered`). The modal MUST NOT congratulate a numeric finish place (1st/2nd/…). Closing the modal MUST leave the player in the room as a finished seat. The room MUST NOT disconnect solely because seats finished. Presence MUST NOT show a finish-place badge (`game/presence`).

#### Scenario [SC-FINISH-31]: Full finish shows knowledge total modal

- **GIVEN** a seated player has exactly one unfinished piece left, delivered knowledge already 4, and that last piece carries 2 knowledge and finishes onto center
- **WHEN** that last piece becomes finished
- **THEN** the seat’s delivered knowledge is 6
- **AND** that player’s client shows a modal reporting 6 knowledge points
- **AND** the modal does not present a finish place number
- **AND** the seat remains in the room after the modal is closed

### Requirement: One knowledge-modal chrome; no place UX

Full finish and penultimate cut MUST share **one** client modal chrome bound to private `knowledge.delivered` (integer). Distinct cut-vs-finish layouts are NOT required. The client MUST open that modal when the **local** seat’s synchronized `finishPlace` transitions 0→1 (reuse today’s finish-place watch; no new room event), reading the latest private `knowledge.delivered` for the body. The client MUST retire place congratulation copy (`finishPlaceModal` / «Вы N-й!») and MUST NOT render numeric `finishPlace` badges on presence. Exact Russian wording MAY vary by i18n key, but MUST communicate delivered knowledge count without ranking place.

#### Scenario [SC-FINISH-34]: Cut and full finish use delivered-only modal

- **GIVEN** a seat reaches full finish or is cut by the penultimate rule
- **WHEN** that seat’s client shows the outcome modal
- **THEN** the modal body is driven by that seat’s `knowledge.delivered` only
- **AND** no finish-place number appears in the modal or on that seat’s presence badge

### Requirement: Penultimate full finish ends the last player

When `maxSeats ≥ 2` and a seated player becomes fully finished while exactly **one** other seated player remains eligible (non-finished), the server MUST immediately end that last eligible seat’s run: set that last seat `finishPlace = 1` (cut), discard (burn) all knowledge still carried on that last seat’s unfinished pieces to 0 without adding it to delivered knowledge, set those pieces `finished = true` so they cannot move, and re-send private `knowledge` to that seat. Every affected client for the cut seat MUST see the **same knowledge-modal chrome** reporting that seat’s **delivered** knowledge only (burned points excluded). Turn deadlines and solo leftover MUST NOT apply. When `maxSeats === 1`, this penultimate cut MUST NOT run — the sole seat plays until its own full finish.

#### Scenario [SC-FINISH-32]: Penultimate finisher cuts the last seat

- **GIVEN** a tourist room with maxSeats 2, seat A has just finished all pieces, and seat B still has unfinished pieces carrying knowledge 5 with delivered knowledge 1
- **WHEN** A’s full finish is applied
- **THEN** B is marked finished/cut and cannot move
- **AND** B’s delivered knowledge remains 1
- **AND** the 5 carried knowledge is not added to B’s delivered total
- **AND** B’s client shows a modal reporting 1 knowledge point

#### Scenario [SC-FINISH-33]: Single-player is not cut by penultimate rule

- **GIVEN** a tourist room with maxSeats 1 and one seated player who finishes all pieces
- **WHEN** that full finish is applied
- **THEN** that player sees the knowledge total modal for their delivered knowledge
- **AND** no other seat is cut
- **AND** the penultimate-cut path does not run

## MODIFIED Requirements

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
