## Why

В лобби уже можно создать комнату с `maxSeats = 1`, но партия не стартует: один сидящий остаётся в `waiting` («жду соперника»). Одиночная игра (создали на одно место) и соло в конце мульти (остался один eligible) — разные режимы; нужна играбельная одиночная партия со своей экономикой ходов, без таймера.

## What Changes

- Комната с `maxSeats = 1` при одном сидящем считается полным столом и входит в countdown 5…1 → `playing`.
- В одиночной (`maxSeats === 1`): без таймера хода; finite шаги/просмотры; при входе в ход +1 шаг +1 просмотр; без кнопки end-turn и без модала «безлимит / 1 минута».
- Когда нет доступных действий (ход, просмотр, rescue / return / push) — сервер сам добавляет +1 шаг и +1 просмотр тому же месту.
- Соло в конце (`maxSeats ≥ 2`, один eligible) — **без изменений** (∞ + 60 с как сейчас).

## Scope

- **Capability ID:** `game/start`, `game/move`, `game/presence`
- **Server:** room `tourist` — старт при полном столе из 1; бюджеты/дедлайн/авто-+1/+1 только для `maxSeats === 1`; соло-остаток не трогать.
- **Client:** Game — числовые счётчики в одиночной; без turn ring / end-turn / соло-unlimited модала; анимация +N при авто-+1/+1.

## Out of scope

- Изменение правил соло в конце (∞ steps/peeks, 60 с, time-expired, модал «Вы один…»).
- Multiplayer при `maxSeats ≥ 2` (60 с ход, end-turn, finite +1/+1 при смене хода) — кроме того, что одиночная не должна ошибочно попадать в соло-ветку.
- Lobby create UI / выбор `maxSeats` (уже есть 1…map.players).
- Auth, leave, say, board layout, content packs/maps.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/start`: полный стол при `maxSeats = 1` и одном seated → countdown (убрать эффективный запрет старта при `seated < 2` для этого случая).
- `game/move`: режим одиночной по `maxSeats === 1` — без turn deadline; finite budgets; +1/+1 при become-current и при отсутствии доступных действий; не включать `infinite` / соло-minute.
- `game/presence`: HUD/chrome одиночной — числа (не ∞), без end-turn, без красного соло-кольца таймера, без unlimited-модала на входе.

## Impact

- Контракт room `tourist`: различение одиночной vs соло-остатка по `maxSeats` (не только по числу eligible); private `budgets` в одиночной всегда finite; `turnUntil` / `turnBudgetSeconds` в одиночной не активны.
- Client Game UX: детект одиночной через synced `maxSeats === 1`; i18n соло-unlimited модала не используется для этого режима.
- Тесты: server mocha (start maxSeats=1; move budgets/auto-+1/+1; соло-остаток регрессия); client vitest (HUD / ring / modal gates).

## References

- Explore: одиночная ≠ соло в конце; countdown оставить; end-turn не нужен; модал на входе не нужен; +1/+1 при отсутствии действий (включая rescue/return/push).
- Main specs: `openspec/specs/game/start/spec.md`, `game/move/spec.md`, `game/presence/spec.md`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
