## Why

В соло после обычного хода последним шагом на центр (один турист финиширует) при остальных на старте и одном в тюрьме игрок получает **0 шагов**, **нет легального просмотра**, UI мёртвый — но **пятиминутный соло-таймер продолжает тикать**, модалка конца не появляется. По канону SC-MOVE-48 seat должен сразу стать time-expired; сейчас игрок ждёт таймер в soft-lock.

## What Changes

- Уточнить соло end: steps=0 и нет **легального** peek (свободный незавершённый на живой `*`) → немедленный `timeExpired` + снятие deadline — в том числе после ordinary center finish.
- Trapped / клетки старта / path / висящий `openPeek` **не** удерживают seat «живым» при steps=0 (openPeek clear/force-close, затем expire).
- Mocha на репортный сценарий; client modal уже по `timeExpired` (SC-PRESENCE-21) — дописать infer copy: trapped ≠ live task (SC-PRESENCE-35), иначе после server-fix модалка откроется с timer-copy.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: соло steps-exhaustion после ordinary center finish и при незавершённых только на старте / в тюрьме (легальный peek отсутствует).
- `game/presence`: модалка steps-exhausted при таком `timeExpired` (трассировка к SC-PRESENCE-21; copy/infer без учёта trapped как «живой задачи»).

## Scope

- Capability: `game/move`, `game/presence`
- Пакет: **server** (авторитетный solo end в room `tourist`); **client** — infer steps vs timer copy без учёта trapped (SC-PRESENCE-35); модалка остаётся на `watch(timeExpired)` (SC-PRESENCE-21)
- Контракт сообщений/схемы без **BREAKING** (то же поле `timeExpired` / turn deadline)

## Out of scope

- Смена правил jail / стоимости rescue / all-jail=только все 4 trapped
- Смена соло peeks∞ / конечных steps / пятиминутного бюджета как такового
- Multi auto-end / end-turn
- Катапульта / trap-pipeline turn pass (отдельный archived change)
- Новые HTTP/auth/DB поля

## Impact

- Server room `tourist`: гарантированный `timeExpired` + clear solo deadline при steps=0 без legal peek
- Client: существующая модалка по `timeExpired`; pure helper infer «live task» (**!trapped**, inject `isPresentTaskCell`) + vitest SC-PRESENCE-35/21 без mount GamePage; i18n keys as-is
- Mocha: репортный SC; Vitest client — infer/modal copy
- Meta skills: `work-with-game` / `work-with-game-board` уже отражают `!hasLegalPeek` / exclude trapped (не откатывать при apply)

## References

- Main specs: [`openspec/specs/game/move/spec.md`](../../specs/game/move/spec.md) (SC-MOVE-48), [`openspec/specs/game/presence/spec.md`](../../specs/game/presence/spec.md) (SC-PRESENCE-21)
- Sibling AGENTS: `../happy-tourist-server/AGENTS.md`, `../happy-tourist.github.io/AGENTS.md`
- Explore: соло last-step → center; remainder на старте + один trapped; таймер тикал, модалки не было
