## Why

После PNG-скинов тайлов катапульта и решётка почти сливаются с коричневой клеткой (тот же тон). Одновременно презентация катапульты слишком короткая (~1000 ms), а иконки say / focus на своём аватаре стоят кучно справа — нужно развести по часам (12 / 9) при неизменном skip.

## What Changes

- Поднять читаемость revealed catapult / grille overlays на поле и на strip (chrome-решётка у trapped в полоске) без непрозрачной «наклейки» под PNG.
- Удлинить успешную презентацию катапульты до ~**1500 ms**; broken-тайминги ×1.5 (client + server presentation budgets в синхроне).
- Переставить affordances на своём аватаре: say на **12 часов**, focus на **9 часов**, end-turn (skip) оставить справа по центру (~**3**).

## Scope

- **Capabilities:** `game/board`, `game/presence`, `game/say`
- **Пакеты:** client (Game UX — overlays, presence chrome) + server (room `tourist`: only `CATAPULT_ANIM_MS` / `CATAPULT_BROKEN_PRESENTATION_MS` + `resetTrapPresentationBudgets` defaults — без смены messages/schema/HTTP/DB)
- **Страница:** Game (seated presence + board)

## Out of scope

- Новые PNG ассетов catapult / grille / tile skins; regenerate art
- Смена плотности / seed / fling / trap логики; grille drop/rise длительность (~1000 ms) без изменений
- MOVE_ANIM_MS / land travel; peek / rescue / push affordances на доске
- Lobby, auth, HTTP, DB; перестановка finish/ready badge (top-left)

## Capabilities

### New Capabilities

(нет)

### Modified Capabilities

- `game/board`: читаемость catapult/grille overlays; успешная катапульта ~1500 ms; broken ×1.5
- `game/presence`: focus на 9 часов; say↔focus↔end-turn расположение по часам; SC-PRESENCE-14/30/33 под новые углы
- `game/say`: say affordance на 12 часов (верхний край по центру), не top-right

## Impact

- Client Game presence + board presentation; server mirrors catapult presentation budgets only (`CATAPULT_ANIM_MS = 1500`, `CATAPULT_BROKEN_PRESENTATION_MS = 450+450+300` / D14 paced pipeline; no protocol change).
- Specs `game/board`, `game/presence`, `game/say` — MODIFIED scenarios (timing + placement; historic scenario/requirement titles kept — design D5).
- Skills notes (apply): `work-with-game-board` (+ focus.md), presence/say placement, server `work-with-game` presentation budgets.

## References

- Explore: contrast after `board-tile-skins`; D1 = CSS drop-shadow (не underlay)
- Sibling AGENTS: client / server; skills `.agents/skills/client/work-with-game-board`, `.agents/skills/server/work-with-game`
