## Context

See `proposal.md` — Why. Cross-package: client Game chrome + App header status; server `peekClose` reason on tourist-room.

Сейчас (после `board-action-icons`): illustrated PNG в `GamePage`; glyph ~18px; presence hit ~36px с offsets `top:-40` / `left:-8` / `right:-8` (не на ободе 96px маркера); board/strip hit ~32px; flag ~18px. `peekClose` = `{ sessionId, correct, row, col }` — client `_onPeekCloseMessage` игнорирует payload. Match-status — `App.vue` `statusLabel` / `data-test-id="game-status"`. End-turn скрыт при `openPeek` (`!isBoardBusy`).

## Goals / Non-Goals

**Goals**

- Presence say/focus/end-turn: hit-center на окружности маркера (12/9/3); glyph ~36px; hit ~56–64.
- Board peek/rescue/push + strip return: glyph ~36px; hit ~48–56; flag без scale.
- `peekClose.reason`: `wrong` | `timeout` (+ `correct`); flash в header slot 10 с всем клиентам.

**Non-Goals**

- Новые PNG / crop; flag scale; budgets/ready; flash на leave mid-peek как «Не успел!»; отдельный BE-event кроме поля в `peekClose`.

## Decisions

### D1 — Размеры (×2 family)

| Slot | Glyph | Hit |
|------|-------|-----|
| presence say/focus/end-turn | ~36px | ~56–64px |
| board peek/rescue/push | ~36px | ~48–56px |
| strip return | ~36px | ~48–56px |
| strip flag | unchanged (~18px) | n/a |

Shared class `.board-action-icon` может остаться одним размером ~36; presence hit CSS отдельно. Transparent chrome / hover opacity — без изменений смысла D1 из `board-action-icons`.

**Альтернатива:** только glyph ×2 при hit 36 → тесно; отброшена (hit тоже растёт умеренно).

### D2 — Circumference placement (presence)

Маркер 96×96, R=48. Центр кнопки на ободе:

| Clock | Placement |
|-------|-----------|
| 12 say | `left:50%; top:0; transform: translate(-50%,-50%)` |
| 9 focus | `left:0; top:50%; transform: translate(-50%,-50%)` |
| 3 end-turn | `left:100%; top:50%; transform: translate(-50%,-50%)` |

Убрать `top:-40` / `left:-8` / `right:-8`. Проверить budgets gap (SC-PRESENCE-25) и say-picker `bottom` относительно нового say. Ready TL без изменений.

**Альтернатива:** polar `translate(R·cos,R·sin)` от центра — избыточно для трёх слотов.

### D3 — `peekClose` reason (server)

Расширить broadcast:

```ts
{ sessionId, correct, row, col, reason?: 'wrong' | 'timeout' }
```

- `correct: true` → reason omit / ignore; FE → «Правильно!»
- wrong submit (`handlePeekSubmit` mismatch) → `correct: false`, `reason: 'wrong'` → «Ошибка!»
- `onTurnTimeout` / solo deadline path while `openPeek` → `resolveOpenPeek(false, 'timeout')` → «Не успел!»
- leave mid-peek / иные force (не timeout) → `reason: 'wrong'` **без** flash «Не успел!»; FE: flash «Ошибка!» **или** no flash — **default: no flash** если reason отсутствует для legacy, для leave передать `reason: 'wrong'` но FE flash только когда close был user-visible answer path…  

Уточнение FE (канон этого change):

| Payload | Flash |
|---------|-------|
| `correct: true` | Правильно! green |
| `correct: false`, `reason: 'wrong'` | Ошибка! red |
| `correct: false`, `reason: 'timeout'` | Не успел! yellow |
| leave force | `reason: 'wrong'` → **Ошибка!** (продукт: модалка закрылась из‑за ухода; редко) — **default** |

Точки server: `resolveOpenPeek(correct, reason?)`; callers timeout → `'timeout'`; submit wrong → `'wrong'`; leave/endTurn force → `'wrong'`.

Mocha: assert `peekClose.reason` на timeout vs wrong submit.

**Альтернатива:** отдельное сообщение `peekOutcome` — отвергнуто (дубль канала).

### D4 — Header flash (client)

- Store: на `peekClose` выставить `peekStatusFlash: { kind: 'correct'|'wrong'|'timeout', until: now+10000 }` (или i18n key + tone).
- `App.vue` `statusLabel`: если flash активен и `isGameRoute` — текст/класс flash; иначе существующая ветка.
- Все клиенты получают broadcast → все видят. Таймер локальный 10 с; clear по expiry (и при leave room).
- Цвета: green / red / yellow (Quasar semantic или CSS vars; без нового token set обязательно).
- Copy: «Правильно!» / «Ошибка!» / «Не успел!» — i18n keys под `game.*` (RU в en-US catalog как принято).

**Альтернатива:** второй overlay поверх без смены `statusLabel` — отвергнута (explore: перекрывать тот же слот).

### D5 — Skills

После wire: notes в `work-with-game-board` (+ focus), `work-with-styles/board.md`, `work-with-pages` (shell/game-page), `work-with-messages` / server messages — размеры, circumference, `peekClose.reason`, flash.

## Risks / Trade-offs

- [Крупные presence hit ↔ budgets / picker] → Mitigation: D2 + визуальная проверка gap; при наложении сдвинуть budgets, не ломая clock.
- [Старые клиенты без reason] → Mitigation: optional field; без reason + `correct:false` → FE default «Ошибка!» или ignore flash — выбрать **Ошибка!** только если `reason==='wrong'`, timeout требует явного reason (иначе нет «Не успел!» на старом сервере — ок).
- [Два flash подряд <10с] → Mitigation: новый peekClose перезаписывает flash + restart 10с.

## Migration Plan

Deploy server (reason) затем client (flash + scale). Rollback = revert siblings независимо; без DB migration.

## Open Questions

(нет — explore закрыт)

## Skills / apply notes

- Client: `GamePage.vue` CSS affordances; `App.vue` status; `stores/game.ts` peekClose; i18n; Vitest presence/board/App status.
- Server: `MyRoom.ts` `resolveOpenPeek` + mocha peekClose reason.
- Tasks: см. `tasks.md`.
