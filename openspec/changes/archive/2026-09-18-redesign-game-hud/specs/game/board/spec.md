# game/board Delta

Related: trap chrome on strip — `game/pieces`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-BOARD-18 | implemented (client drop/rise ~1000 ms) |
| SC-BOARD-19 | implemented (client rise ~1000 ms — covered with 18) |
| SC-BOARD-21 | covered-by-reuse (leave clear still public rise) |

## MODIFIED Requirements

### Requirement: Revealed grille drop and clear animations are public

When a piece lands on an unspent grille, every client that displays the board MUST show the grille lowering onto that cell (product sense: drops from above downward) for about **1000 ms**. When that grille is later cleared (rescue, all-jail holding clear, or permanent leave of the seat whose piece held that grille), every such client MUST show the grille rising and disappearing for about **1000 ms**. Cleared grilles MUST NOT remain visible afterward. Personal tourist chrome that mirrors a trapped piece (`game/pieces`) MUST use the same about **1000 ms** timing for its grille drop/rise presentation.

#### Scenario [SC-BOARD-18]: Everyone sees the drop

- **GIVEN** phase is `playing` and a piece lands on a cell with an unspent grille
- **WHEN** the move settles on that cell
- **THEN** every client shows the grille drop animation on that cell lasting about 1000 ms

#### Scenario [SC-BOARD-19]: Everyone sees rise and vanish on clear

- **GIVEN** a revealed grille is holding a trapped piece and is then cleared by a successful rescue
- **WHEN** the holding grille is cleared
- **THEN** every client shows the grille rise and vanish lasting about 1000 ms

#### Scenario [SC-BOARD-21]: Everyone sees rise when leave clears holding

- **GIVEN** a revealed holding grille is cleared because that seat permanently left
- **WHEN** clients update the board
- **THEN** every client shows the grille rise and vanish lasting about 1000 ms
