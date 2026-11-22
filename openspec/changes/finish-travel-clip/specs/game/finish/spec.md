# Delta: game/finish

Надёжный presentation-owned travel на enter-center finish и return-from-finish (все клиенты; без snap-vanish от sync).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-15 | modified — return travel clip; no snap; all viewers |
| SC-FINISH-18 | covered-by-reuse — own move→center travel from pre-move cell |
| SC-FINISH-19 | modified — catapult→center travel after vanish; sync-independent presentation |
| SC-FINISH-20 | modified — finish travel when finished sync late (false at reveal enqueue) |
| SC-FINISH-35 | modified — click + catapult finish travel on every board viewer; behavioral test |
| SC-FINISH-36 | **added** — finished piece stays visible while finish travel presents (early/late sync) |
| SC-FINISH-37 | **added** — early `finished` during catapult overlay keeps piece until finish travel starts |

## MODIFIED Requirements

### Requirement: Center finish keeps travel animation

When an unfinished piece becomes finished by landing on a center cell (direct move click or catapult fling chain), **every client that displays the board** MUST present finish travel (~550 ms slide) then fade (~300 ms) so the piece does not snap-vanish from its pre-finish cell — including the **first** finisher of a seat and including opponent/spectator viewers. Mid-join already-finished pieces (no observed transition into finished) MAY omit travel. Catapult→center MUST start finish travel only after that hop’s catapult presentation has fully settled (land → overlay → vanish), then run the same travel path from the catapult cell. While finish travel is presenting **or** while catapult presentation still owns the piece before that finish travel starts, synchronized `finished` / center coordinates MUST NOT remove the piece from the board without that travel. Tests MUST cover a **behavioral** path that fails if travel origin is missing when finishing from a non-center cell (SFC source-only asserts are not sufficient alone).

#### Scenario [SC-FINISH-35]: Click and catapult finish animate to center

- **GIVEN** an unfinished piece finishes onto center by direct move or by catapult fling (including a seat’s first finished tourist)
- **WHEN** any client that displays the board paints that finish
- **THEN** that client shows the piece traveling from the pre-finish (or catapult) cell toward center before fading
- **AND** the piece does not disappear from the origin cell without that travel
- **AND** a behavioral test fails if the travel origin is missing for a non-center finish

#### Scenario [SC-FINISH-36]: Sync finish does not drop the piece mid-travel

- **GIVEN** a piece is presenting finish travel toward center on a client
- **WHEN** synchronized state marks that piece finished at a center cell (early or late relative to presentation start)
- **THEN** the piece remains visible along the finish travel path until slide+fade complete
- **AND** the client MUST NOT leave the piece absent from the board without that travel solely because sync already shows finished at center

### Requirement: Return-from-finish travel animation

When the server accepts a legal `returnFromFinish` (**existing** room `tourist` authority — unchanged in this change), every client that displays the board MUST animate that piece traveling from the **nearest** of the four center finish cells to the chosen ring cell (Chebyshev distance to the ring cell; ties broken by lower row, then lower column), using the same travel timing family as ordinary piece moves. The animation MUST be visible to all clients in the room. While return travel is presenting, synchronized piece coordinates MUST NOT snap the piece to the ring cell without that travel from the nearest center. Input locking for the submitting client during travel follows the same rules as move travel (`game/move`). Mid-join that never observed the return transition MAY omit travel.

#### Scenario [SC-FINISH-15]: Everyone sees return leave the finish block

- **GIVEN** a seated player successfully returns a finished piece onto a legal ring cell
- **WHEN** clients update the board
- **THEN** every client shows that piece animating from the nearest center cell to that ring cell
- **AND** the piece does not appear only at the ring cell without that travel from nearest center
- **AND** after the animation the piece occupies the ring cell as an unfinished piece

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules (after that hop’s presentation budget), the server SHALL finish that piece with the same **authority** as an ordinary move onto that center cell (`game/finish` enter-center rules) — **unchanged** in this change (no new schema/messages, no delayed `finished`). Clients MUST own presentation: run finish travel and disappear **after** the triggering catapult’s land arrival and overlay have fully completed (`game/board` successful presentation: land → overlay → then finish travel), not in parallel with that overlay. Finish travel MUST use the same about **550 ms** slide and about **300 ms** disappear family as move onto center. Under existing server-paced sync the piece MAY become `finished` only when the hop’s relocate fires — **after** reveal enqueue — and clients MUST still present finish travel from the catapult cell after that hop’s vanish (they MUST NOT drop the piece from the board without travel solely because `finished` was not known at reveal time, and MUST NOT drop it without travel when `finished` arrives early — including while the overlay is still presenting before finish travel starts). In a multi-hop catapult chain ending on center, every hop MUST animate; finish travel MUST start only after the last successful catapult vanish. Presentation of that finish travel MUST follow the same all-viewer and sync-independence rules as the center-finish travel requirement.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** that catapult’s arrival and overlay have fully completed and the deferred fling / finish presentation runs
- **THEN** the piece is finished with the same authority as a move onto that center
- **AND** every client that displays the board presents finish travel (~550 ms) and disappear (~300 ms) consistent with move onto center, starting only after the catapult vanish
- **AND** the piece does not disappear from the catapult cell without that travel

#### Scenario [SC-FINISH-20]: Finish travel even when finish sync arrives after reveal

- **GIVEN** a successful catapult reveal is presenting and the authoritative finish onto center is applied only after the overlay presentation budget (paced fire)
- **WHEN** the overlay vanish completes
- **THEN** every client MUST animate finish travel from the catapult cell to center and then disappear
- **AND** MUST NOT leave the piece absent from the board without that travel solely because `finished` was false at reveal enqueue time
- **AND** in a catapult→…→center chain, prior hops complete their overlay and fling travel before the final hop’s finish travel

#### Scenario [SC-FINISH-37]: Early finished during catapult overlay keeps the piece

- **GIVEN** a catapult→center hop overlay is presenting and synchronized state already marks that piece `finished` at a center cell before vanish completes
- **WHEN** any client that displays the board paints that overlay window
- **THEN** the piece remains visible under catapult presentation ownership until vanish completes and finish travel starts
- **AND** the client MUST NOT leave the piece absent from the board solely because sync already shows finished at center before finish travel begins
