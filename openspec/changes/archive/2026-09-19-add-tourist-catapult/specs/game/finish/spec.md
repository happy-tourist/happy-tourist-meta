## Purpose

Delta этого change: fling катапульты на center завершает фишку. Базовый finish / return — main `game/finish`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-FINISH-19 | covered (server mocha + client UX — finish after land + catapult vanish; paced hop) |
| SC-FINISH-20 | covered (client UX — always finish travel after vanish when paced finish arrives late) |

Related: fling rules / paced resolve — `game/move`; board anim — `game/board`.

## ADDED Requirements

### Requirement: Catapult fling onto center finishes like a move

When a catapult relocates a piece onto any center cell under `game/move` fling rules (after that hop’s presentation budget), the server SHALL finish that piece with the same authority and presentation expectations as an ordinary move or push onto that center cell (`game/finish` enter-center rules). Clients MUST run finish travel and disappear **after** the triggering catapult’s land arrival and overlay have fully completed (`game/board` successful presentation: land → ~1000 ms overlay → then finish travel), not in parallel with that overlay. Under server-paced sync the piece MAY become `finished` only when the hop’s relocate fires — **after** reveal enqueue — and clients MUST still present finish travel from the catapult cell after that hop’s vanish (they MUST NOT drop the piece from the board without travel solely because `finished` was not known at reveal time). In a multi-hop catapult chain ending on center, every hop MUST animate; finish travel MUST start only after the **last** successful catapult vanish. Turn advance, if pending after the pipeline, MUST wait until finish presentation for that hop is covered by the trap pipeline idle rules in `game/move`.

#### Scenario [SC-FINISH-19]: Catapult fling onto center finishes the piece

- **GIVEN** a free unfinished piece triggers a catapult whose chosen fling destination is a center cell
- **WHEN** that catapult’s arrival and overlay have fully completed and the deferred fling / finish presentation runs
- **THEN** that piece becomes finished when the paced relocate+finish is applied
- **AND** clients present finish travel and disappear consistent with move/push onto center, starting only after the catapult vanish

#### Scenario [SC-FINISH-20]: Finish travel even when finish sync arrives after reveal

- **GIVEN** a successful catapult reveal is presenting and the authoritative finish onto center is applied only after the overlay presentation budget (paced fire)
- **WHEN** the overlay fully vanishes
- **THEN** every client MUST animate finish travel from the catapult cell to center and then disappear
- **AND** MUST NOT leave the piece absent from the board without that travel solely because `finished` was false at reveal enqueue time
- **AND** in a catapult→…→center chain, prior hops complete their overlay and fling travel before the final hop’s finish travel
