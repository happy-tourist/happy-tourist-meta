## Purpose

Game presence caption «AI» for seats with synced `isBot: true`. Schema / bot policy — `game/ai`; create UX — `lobby/rooms`.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PRESENCE-60 | `src/pages/__tests__/GamePresenceAi.test.ts` (client) |

## ADDED Requirements

### Requirement: Bot seats display as AI

On the Game screen, every seat with synced `isBot: true` MUST show the product name **AI** as a short text caption **directly under** that seat’s presence tourist avatar (top-row opponent markers and, if ever mirrored there, bottom HUD — same caption rules). The client MAY localize the token via i18n (`game.presenceAi`); default Russian/English product sense remains the literal «AI». All bot seats in the same room MUST use that same label — no ordinal suffixes. Human seats MUST NOT gain a new name caption in this change (avatar-only identity as today). Tourist kind images and turn/reconnect chrome for bot seats follow existing presence rules; bots do not show reconnect grace (`game/ai`).

#### Scenario [SC-PRESENCE-60]: Opponent bot markers labeled AI

- **GIVEN** the human player is seated in an effective-AI room with two bot opponents (isBot true)
- **WHEN** the Game presence layout is shown
- **THEN** each bot opponent marker shows the caption AI under its avatar
- **AND** both bots use the same AI label without “AI 2” style suffixes
- **AND** the human’s own presence marker has no AI caption
