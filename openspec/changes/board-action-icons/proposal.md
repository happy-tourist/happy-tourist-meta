## Why

На Game affordances (peek / rescue / push / return / flag / focus / end-turn / say) сейчас рисуются Material `q-icon` внутри цветных кругов. Есть готовый набор иллюстрированных PNG под эти слоты — нужно подключить их вместо глифов и убрать цветной chrome, чтобы действия читались в одном стиле с полем.

## What Changes

- Заменить Material-глифы на иллюстрированные иконки действий поля для восьми affordances: peek, rescue, push, return, finished-flag, focus, end-turn, say.
- Убрать цветной круговой фон у этих кнопок; hit-area и часовая раскладка (12 / 9 / 3) без изменений.
- Бюджеты steps/peeks в HUD и ready — без смены иконок (Material как сейчас).

## Scope

- **Capabilities:** `game/board`, `game/move`, `game/finish`, `game/presence`, `game/say`
- **Пакеты:** client only (Game UX — affordance chrome)
- **Страница:** Game (board + strip + own presence)

## Out of scope

- Server / room messages / schema / HTTP / DB
- Новые ассеты или перерисовка art (исходники уже подготовлены)
- Budgets HUD (`directions_walk` / `visibility`), ready affordance
- Peek modal chrome, say-picker список пресетов
- Позиции / hit-size / логика активации affordances (только визуал)

## Capabilities

### New Capabilities

(нет)

### Modified Capabilities

- `game/board`: peek affordance — иллюстрированная иконка без цветного круга (вместо eye / Material `visibility`)
- `game/move`: rescue / push affordances — иллюстрированные иконки без зелёного круга
- `game/finish`: return affordance и finished-flag — иллюстрированные иконки; return без «зелёного» chrome
- `game/presence`: focus и end-turn — иллюстрированные иконки без цветного круга (вместо Material `center_focus_strong` / `skip_next`)
- `game/say`: say affordance на аватаре — иллюстрированная иконка без цветного круга (вместо speech-bubble Material)

## Impact

- Client Game affordance visuals only; no protocol change.
- **Server / contract:** no new or changed room messages, schema, HTTP, or DB. Affordance activation still submits existing `peek` / `rescue` / `push` / `returnFromFinish` / `endTurn` / `say` (and local focus) per main `game/*` specs — this change does not add BE claims.
- Specs выше — MODIFIED visual wording (placement / activation scenarios stay; Material / «green circle» / «eye» / «speech-bubble glyph» → illustrated board-action icons).
- Skills notes (apply / pre-aligned in meta): `work-with-game-board` (+ `focus.md` / `peek.md`), `work-with-styles/board.md`, `work-with-pages/game-page.md` — illustrated icons, no colored circular chrome.

## References

- Explore: rename/move/crop PNG → `src/assets/board/`; D1 = без цветного фона; budgets/ready out
- Sibling AGENTS: client; skills `.agents/skills/client/work-with-game-board`
