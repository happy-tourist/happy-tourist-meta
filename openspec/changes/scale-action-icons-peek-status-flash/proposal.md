## Why

Иллюстрированные board-action иконки после wire слишком мелкие, а say / focus / end-turn у аватара «висят» сбоку, а не на окружности маркера. Закрытие peek (верный ответ, ошибка, таймер хода) не объясняется в шапке — игрокам неочевидно, почему модалка исчезла.

## What Changes

- Увеличить ~×2 глифы и hit affordances на поле (peek / rescue / push) и у аватара (say / focus / end-turn); return на strip — тоже ~×2; finished-flag размер без изменений.
- Поставить presence-кнопки на окружность маркера (12 / 9 / 3).
- После закрытия peek все клиенты 10 с видят в центре шапки вместо обычного match-status: «Правильно!» (зелёный) / «Ошибка!» (красный) / «Не успел!» (жёлтый при таймере хода с открытым peek); затем снова обычный статус.
- Расширить `peekClose` причиной закрытия (wrong vs timeout), чтобы FE различал «Ошибка!» и «Не успел!».

## Scope

- **Capabilities:** `game/board`, `game/move`, `game/finish`, `game/presence`, `game/say`
- **Пакеты:** client (Game chrome / App header status) + server (`peekClose` reason на tourist-room)
- **Страница:** Game; match-status в shared header

## Out of scope

- Перерисовка PNG / новые ассеты
- Изменение размера finished-flag
- Новые тексты кроме трёх flash-фраз; i18n-расширение сверх этих ключей по желанию apply
- Отдельный flash при leave mid-peek / иных force-close без timeout (не «Не успел!»; wrong-path или без flash — по design default)
- Budgets / ready chrome
- Deploy / Pages pipeline

## Capabilities

### New Capabilities

(нет)

### Modified Capabilities

- `game/board`: размер peek (и родственных board) affordance; flash-триггер от `peekClose` (correct / reason)
- `game/move`: turn timeout с открытым peek → `peekClose` с reason timeout (поведение KEEP без изменения)
- `game/finish`: return affordance ~×2; flag без изменений размера
- `game/presence`: clock на окружности; say/focus/end-turn ~×2; match-status flash 10 с поверх обычного label
- `game/say`: say affordance размер + положение на 12 o’clock обода

## Impact

- Client: Game affordance CSS/DOM; App header status flash; store читает `peekClose.reason` / `correct`.
- Server: `peekClose` payload (+reason); timeout path помечает timeout; wrong submit — wrong. Контракт совместим с добавлением поля (старые клиенты игнорят).
- Skills notes: board / presence / game-page / messages при apply.

## References

- Explore: ×2 glyph+hit presence+board+return; flag unchanged; circumference 12/9/3; flash replaces status 10s all clients; Правильно! / Ошибка! / Не успел! (timer+open peek); end-turn blocked while peek open
- Sibling AGENTS: client + server; skills `work-with-game-board`, `work-with-styles/board.md`, `work-with-pages` (shell / game), `work-with-messages`
