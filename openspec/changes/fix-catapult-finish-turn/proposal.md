## Why

В multiplayer катапульта переносит последнего туриста на центр: `finishPlace` назначается mid-pipeline (`applyCenterFinish`, модалка победы уже зеркалит sync), но на idle синхронизированный текущий ход может остаться на уже финишировавшем seat, пока есть другие eligible игроки. Нужно закрыть разрыв между назначением finish place и правилом «finished seat не держит ход» — снятие хода на `onTrapPipelineIdle`.

## What Changes

- На idle trap-pipeline, если seat уже получил finish place mid-pipeline (в т.ч. катапульта → центр на четвёртом туристе), сервер обязан снять текущий ход с этого seat, если остаётся хотя бы один eligible соперник.
- Уточнение поведения в `game/move` / `game/finish`: полный финиш через катапульту не оставлять `currentTurn` на финишере после idle.
- Server mocha-покрытие нового сценария; client UI не меняется (модалка уже зеркалит `finishPlace`).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: после idle trap-pipeline seat с новым finish place не остаётся current turn при наличии другого eligible seat.
- `game/finish`: полный финиш через катапульту на центр назначает место **и** освобождает ход так же, как полный финиш обычным ходом на центр.

## Scope

- Capability: `game/move`, `game/finish`
- Пакет: **server** (авторитетная очередь хода / trap pipeline в room `tourist`)
- Client: вне scope правок UI (place-модалка уже по `finishPlace`, turn chrome по `currentTurnSessionId`; mid-pipeline модалка до idle chrome — ожидаемо, см. design D3); контракт сообщений/схемы без breaking

## Out of scope

- Изменение правил плотности катапульт, fling-геометрии, paced presentation budgets
- Авто-advance после обычного хода, который **не** назначает finish place
- Solo (один eligible) — ротация хода к «сопернику» не применима
- Client chrome / i18n / анимации доски

## Impact

- Server room `tourist`: синхронизированные `finishPlace` + `currentTurnSessionId` после catapult→center full finish
- Нет новых HTTP/auth/DB полей; нет **BREAKING** client protocol
- Тесты: mocha room scenarios для нового SC

## References

- Main specs: [`openspec/specs/game/move/spec.md`](../../specs/game/move/spec.md), [`openspec/specs/game/finish/spec.md`](../../specs/game/finish/spec.md)
- Sibling AGENTS: `../happy-tourist-server/AGENTS.md` (paced trap pipeline / turn skip finished)
- Explore: catapult→center fourth piece → place modal without turn pass
- Design D1: anti-double-advance / pending absorb на `onTrapPipelineIdle`
