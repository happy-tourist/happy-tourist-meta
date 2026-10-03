## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-49 | covered (`GameBoardWire`) |

Related: shared peek modal SC-BOARD-43…47 (main `game/board` — correct order / sync unchanged); task compose reuse SC-PACK-263 (same change, `content/packs`).

**Server:** no delta. Existing `peekPlace` / `peekSubmit` already accept the same `answerCardId` in multiple slots; correctness remains ordered exact match.

## ADDED Requirements

### Requirement: Peek answer chips stay reusable without used-state chrome

While a shared peek modal is open, the peeker MUST be allowed to place the same pack answer card into any number of empty answer slots (including every slot). After a card is placed in one slot, its chip in the answer pool MUST remain selectable for other empty slots. The client MUST NOT disable that chip solely because it is already used in a slot, and MUST NOT present a distinct «already used» highlight or filled-vs-outline state that marks the card as consumed. Clearing a filled slot (peeker) remains allowed. Spectators still MUST NOT edit slots. Correct submit still requires the slot order to match the bound task’s defined slot order exactly (including when that definition repeats the same answer card).

#### Scenario [SC-BOARD-49]: Same answer card may fill multiple peek slots

- **GIVEN** a seated peeker has an open peek with two or more empty slots and at least one answer-card chip
- **WHEN** that peeker places the same answer card into more than one slot
- **THEN** each of those slots shows that card
- **AND** the answer-pool chip for that card remains available for further empty slots
- **AND** the chip is not shown in a distinct already-used disabled or highlighted consumed state solely because it appears in a slot
- **AND** other clients still see the placements in realtime and cannot edit slots
