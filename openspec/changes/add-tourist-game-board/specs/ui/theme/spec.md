## MODIFIED Requirements

### Requirement: Tourist board appearance is unchanged by theme

Changing the application chrome theme MUST NOT alter the tourist board tile colors (start, task, and center fills). Board holes MUST continue to show the Game screen page background under the active light or dark chrome.

#### Scenario [SC-THEME-07]: Board looks the same in light and dark chrome

- **GIVEN** the user is on the Game screen with a visible tourist board
- **WHEN** the chrome theme is switched between light and dark
- **THEN** start, task, and center tile colors keep their gameplay fills
- **AND** board holes remain visually aligned with the current page background

## RENAMED Requirements

- FROM: `### Requirement: Checkers board appearance is unchanged by theme`
- TO: `### Requirement: Tourist board appearance is unchanged by theme`
