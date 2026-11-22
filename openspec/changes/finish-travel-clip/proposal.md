## Why

При финише туриста (клик в центр или катапульта→центр) и при return-from-finish фигура часто **snap-vanish**: пропадает с клетки без slide. Повторные фиксы через реконструкцию анимации из sync (`lastKnown` / double-rAF / defer-флаги) остаются флаковыми — презентация гоняется с серверным `finished`+coords. Нужна надёжная модель: локальный travel-clip владеет пикселями, sync не обрывает анимацию.

## What Changes

- Client Game: presentation-owned **travel clip** для enter-center finish (клик и catapult→центр) и для return-from-finish — на **всех** клиентах в комнате.
- Убрать хрупкую реконструкцию finish travel из гонки sync (seed/`from`-трюк с clear после rAF как источник истины).
- Серверный контракт finish/return **без изменений** (authority как сейчас).

## Scope

- **Capability ID:** `game/finish` (по касанию формулировок презентации — `game/board`, только если требования hop→finish уже завязаны на клиентский travel)
- **Client:** Game board — надёжный slide+fade на финиш и travel на return для каждого зрителя доски; mid-join уже finished — travel MAY omit.
- **Server:** out of product behavior change (room `tourist` finish/return authority unchanged).

## Out of scope

- Обычный ход / catapult fling **не** на центр (остаётся текущий move/pin/fling path).
- Knowledge modal, `finishPlace`, transfer knowledge, trap seed, turn timer.
- Auth, lobby, packs/maps, leave, say.
- Серверная задержка `finished`/coords под UX-тайминги.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/finish`: finish travel (клик + catapult→центр) и return travel — presentation-owned clip на всех клиентах; запрет snap-vanish при раннем/позднем sync `finished`; усиление SC-FINISH-15 / SC-FINISH-19 / SC-FINISH-20 / SC-FINISH-35; **новые** SC-FINISH-36 (видимость mid-travel) и SC-FINISH-37 (ранний `finished` во время catapult overlay до старта clip) — поведенческие критерии, не только source-assert.

## Impact

- Client UX only: Game board piece presentation.
- Server **reuse-only** (без правок runtime): room `tourist` `move` / `returnFromFinish`, sync piece `finished`+`row`/`col`, paced catapult→center finish — как сейчас; delay `finished` под UX **запрещён** (Out of scope).
- Тесты client (vitest) на clip/`from` и отсутствие drop finished без travel; meta skills board/finish UX при apply.

## References

- Explore 2026-11-22: finish snap-vanish, clip vs reconstruct, scope оба входа + все клиенты + return.
- Main specs: `openspec/specs/game/finish`, `openspec/specs/game/board` (SC-BOARD-31 hop→center).
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
