## Why

После correct peek снятые task-клетки становятся визуальными дырами, на которые нельзя ходить (move / push / return / fling) — доска «рвётся», туристам сложнее добираться до центра. В соло остаётся жёсткий лимит шагов и пятиминутный бюджет, из‑за чего партия часто заканчивается exhaustion’ом, а не таймером или финишем. Нужно вернуть проходимость снятых тайлов и сделать соло короче и свободнее по бюджетам.

## What Changes

- Снятые task-тайлы снова **проходимы** везде (multi + solo): обычный ход, push, return-from-finish, catapult fling.
- Layout-дыры карты (клетки, которые никогда не были тайлом) остаются непроходимыми.
- Соло: deadline **1 минута** вместо 5; **ходы и просмотры бесконечны**; ветка «шаги кончились → time-expired» убирается.
- Модал соло и HUD: оба бюджета как ∞; копия про «1 минуту» и безлимит ходов/просмотров.

## Scope

- **Capability ID:** `game/move`, `game/board`, `game/presence`
- **Server:** room `tourist` — валидация посадки на `removedTaskKeys`, соло-deadline 60 с, ∞ steps + peeks в соло, без steps-exhaustion gate.
- **Client:** Game — red destination hints на снятые тайлы; соло-модал / счётчики ∞ / red ring под 1 мин (детект соло по одному eligible seat, не по длительности бюджета); без steps-exhausted end copy.

## Out of scope

- Восстановление / «магазин» снятых тайлов.
- Изменение multiplayer turn budget (60 с) и конечных steps/peeks при ≥2 eligible.
- Новые типы клеток, карт или trap-правил сверх проходимости снятых task-клеток.
- Auth, lobby, leave, say.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: проходимость снятых task-клеток для move/push/return/fling; соло 60 с; ∞ steps + peeks; удаление steps-exhaustion → time-expired.
- `game/board`: снятый тайль остаётся визуальной дырой, но **landable** (в т.ч. client hints).
- `game/presence`: соло HUD оба ∞; модал «1 минута / безлимит»; убрать distinct steps-exhausted copy / scenarios.

## Impact

- Контракт room `tourist`: поведение `move` / `push` / `returnFromFinish` / catapult land + private `budgets` (`infinite: true` = ∞ peeks **и** ∞ steps в соло) + `turnBudgetSeconds` соло = 60; schema sync полей не расширяется новыми ключами (ожидаемо).
- Client Game UX: landable holes, i18n соло-модала, presence counters / end modals.
- Тесты: server mocha (`touristMove` + `MyRoom`) и client vitest (hints / soloEndModalKind / presence).

## References

- Explore/design: D1–D5 + soft 1–3 (проходимость везде; layout-дыры нет; соло 1 мин / ∞ / без exhaustion; текст модала; client landable + red ring по eligible seats; stale OpenSpec titles → bodies канон).
- Main specs: `openspec/specs/game/move/spec.md`, `game/board/spec.md`, `game/presence/spec.md`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
