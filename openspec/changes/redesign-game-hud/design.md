## Context

См. `proposal.md` — Why / Scope. Сейчас на client: локальный chrome на `GamePage` (leave + `statusLabel` + `roomId.slice(0, 8)`), presence через `presenceLayout` (top opponents / bottom self вокруг `.tourist-board`), личный strip отдельным блоком под frame с caption «Мои туристы», общая шапка только с theme в `App.vue`. Server и Pinia I/O leave/say/budgets не меняются.

Пакет: **client** (`../happy-tourist.github.io`). Контракт room `tourist` без изменений.

## Goals / Non-Goals

**Goals:**

- Общая шапка: leave слева + статус по центру только на Game; theme справа как сейчас.
- Единая нижняя sticky HUD-панель: seated L→R own+budgets → strip → opponents right; spectator — occupied по центру.
- Say bubbles только вверх; убрать room id из UI и подпись strip.
- Обновить client skills под новый layout после кода.

**Non-Goals:**

- Мобильный overflow-policy (отложено).
- Вынос GamePage на компоненты ради рефакторинга (допустимо минимально, если разгрузка шапки потребует shared helper для status/leave).
- Server / schema / messages.

## Decisions

### D1 — Chrome leave + status в `App.vue` (route-aware)

**Выбор:** расширить `src/App.vue` `q-toolbar`: при `route.name === 'game'` показывать icon-only leave слева и centered status; theme справа (`q-space` / flex). Leave confirm dialog и `consentedLeaving` / `leaveGame` остаются логикой Game — либо тонкий event/callback из page, либо вынести computed status + leave handlers в shared composable / читать `useGameStore` из App и держать confirm dialog в App или Teleport из GamePage.

**Альтернативы:** (B) `Teleport` из GamePage в named slot header — чище границы page, больше wiring; (C) визуально «в шапке» внутри page — ломает sticky header Quasar.

**Rationale:** один source of truth для toolbar; skills `work-with-pages` обновить: game leave/status MAY жить в App на game-route.

**Предпочтительная врезка:** App читает `useRoute` + `useGameStore` для status strings (те же ветки, что нынешний `statusLabel`); leave click эмитит/вызывает page-local confirm — проще держать `q-dialog` leave в GamePage и пробрасывать open через store flag **или** перенести confirm dialog в App рядом с кнопкой, вызывая существующие `leaveGame` / router из store. Минимальный путь: confirm + leave orchestration переехать в App вместе с кнопкой (store уже умеет leave), GamePage только `ensureTouristRoom` / board.

### D2 — Нижняя панель: layout shell на GamePage

**Выбор:** заменить `.presence-frame` column (top row → board → bottom row) на:

```
q-page (column, min-height fill)
  board scroll region (flex 1, overflow auto)
  .game-hud (sticky/fixed bottom: own | budgets | strip | opponents)
```

Убрать `presenceLayout` top/bottom slots; markers получают один «bottom» контекст для say. Seated: flex row `justify-between` / `margin-left: auto` на opponents group. Spectator: `justify-center` только markers group, без own/strip.

Strip перенести внутрь `.game-hud` (после own budgets), удалить caption «Мои туристы».

Say CSS: только `say-bubbles--bottom` (выше аватара); удалить/не использовать top-below ветку.

### D3 — Sticky к viewport

**Выбор:** панель как `position: sticky; bottom: 0` внутри page **или** Quasar `q-footer` на game-route. Sticky внутри column fill проще не ломая `q-layout`. Зарезервировать padding-bottom у board region = высота HUD, чтобы последний ряд доски не прятался под панелью.

**Альтернатива:** `position: fixed` — нужна ручная компенсация высоты; хуже с safe-area.

### D4 — Удаление room id chrome

Убрать отображение `game.roomId?.slice(0, 8)` из GamePage; route param / reconnect без изменений.

### D5 — Skills (после кода)

Обновить `.agents/skills/client/work-with-game-board`, `work-with-pages` (и при необходимости say/styles notes): bottom HUD, header leave/status, no room id, bubbles always up.

Чеклист шагов — `tasks.md`.

## Risks / Trade-offs

- [Узкая ширина / 4 strip + 3 opp] → Mitigation: out of scope; допускается horizontal scroll панели без сжатия 72px avatar (как сейчас row-scroll).
- [App знает game status] → Mitigation: только зеркало store; Colyseus I/O остаётся в Pinia `game`.
- [Say picker / bubbles clip у sticky footer] → Mitigation: `overflow: visible` на HUD; z-index над board; не клипать picker (SC-SAY-15 сохранён).
- [Purpose main specs] → при `/opsx-sync` обновить Purpose у `game/presence` / `game/say` под bottom HUD.

## Migration Plan

Только client deploy (GitHub Pages). Rollback = revert UI commit. Server не затрагивается.

## Open Questions

Нет (мобильный overflow отложен сознательно).
